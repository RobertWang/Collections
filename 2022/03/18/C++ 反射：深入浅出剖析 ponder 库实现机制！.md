> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=2655472385&idx=4&sn=0884b9074dc9dd55af5d52b2122b8dac&chksm=bd7287768a050e602f509239c68d4f07722e7fda77e8ba5d74d83eb376f57bd3bcef0c861f4e&mpshare=1&scene=1&srcid=0316BOAjBRCCdEpCHjzRQnyT&sharer_sharetime=1647401018185&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

导语 | 给静态语言添加动态特性，似乎是 C++ 社区一件大家乐见其成的事情，轮子也非常多，我们不一一列举前辈们造的各种流派的轮子了，主要还是结合我们框架用到的 C++ 反射实现，结合 C++ 的新特性，来系统的拆解目前框架中的反射实现。另外代码最早脱胎于 ponder，整体处理流程基本与原版一致，所以相关的源码可以直接参考 [ponder 的原始代码](https://github.com/billyquith/ponder) 

**一、简单的示例**

```
//-------------------------------------
//register code
//-------------------------------------
  using namespace framework;
  using namespace framework::reflection;
  using namespace framework::math;
  __register_type<Vector3>("Vector3")
      .constructor()
      .constructor<Real, Real, Real>()
      .property("x", &Vector3::x)
      .property("y", &Vector3::y)
      .property("z", &Vector3::z)
      .function("Length", &Vector3::Length)
      .function("Normalise", &Vector3::Normalise)
      .overload(
          "operator*", [](Vector3* caller, Real val) { return caller->operator*(val); },
          [](Vector3* caller, const Vector3& val) { return caller->operator*(val); });
//-------------------------------------
//use code
//-------------------------------------
auto* metaClass = __type_of<framework::math::Vector3>();
ASSERT_TRUE(metaClass != nullptr);
auto obj = runtime::CreateWithArgs(*metaClass, Args{1.0, 2.0, 3.0});
ASSERT_TRUE(obj != UserObject::nothing);
auto obj2 = runtime::CreateWithArgs(*metaClass, Args{1.0});
ASSERT_TRUE(obj2 == UserObject::nothing);
const reflection::Property* fieldX = nullptr;
metaClass->TryGetProperty("x", fieldX);
ASSERT_TRUE(fieldX != nullptr);
const reflection::Property* fieldY = nullptr;
metaClass->TryGetProperty("y", fieldY);
ASSERT_TRUE(fieldY != nullptr);
const reflection::Property* fieldZ = nullptr;
metaClass->TryGetProperty("z", fieldZ);
ASSERT_TRUE(fieldZ != nullptr);
double x = fieldX->Get(obj).to<double>();
ASSERT_DOUBLE_EQ(1.0, x);
fieldX->Set(obj, 2.0);
x = fieldX->Get(obj).to<double>();
ASSERT_DOUBLE_EQ(2.0, x);
fieldX->Set(obj, 1.0);
x = fieldX->Get(obj).to<double>();
double y = fieldY->Get(obj).to<double>();
double z = fieldZ->Get(obj).to<double>();
ASSERT_DOUBLE_EQ(1.0, x);
ASSERT_DOUBLE_EQ(2.0, y);
ASSERT_DOUBLE_EQ(3.0, z);
const reflection::Function* lenfunc = nullptr;
metaClass->TryGetFunction("Length", lenfunc);
ASSERT_TRUE(lenfunc != nullptr);
const reflection::Function* normalizeFunc = nullptr;
metaClass->TryGetFunction("Normalise", normalizeFunc);
ASSERT_TRUE(normalizeFunc != nullptr);
// Overload tests
auto& tmpVec3 = obj.Ref<Vector3>();
const reflection::Function* overFunc = nullptr;
metaClass->TryGetFunction("operator*", overFunc);
Value obj3 = runtime::CallStatic(*overFunc, Args{obj, 3.0});
auto& tmpVec4 = obj3.ConstRef<UserObject>().Ref<Vector3>();
ASSERT_DOUBLE_EQ(3.0, tmpVec4.x);
ASSERT_DOUBLE_EQ(6.0, tmpVec4.y);
ASSERT_DOUBLE_EQ(9.0, tmpVec4.z);
Value obj4 = runtime::CallStatic(*overFunc, Args{obj, Vector3(2.0, 2.0, 2.0)});
auto& tmpVec5 = obj4.ConstRef<UserObject>().Ref<Vector3>();
ASSERT_DOUBLE_EQ(2.0, tmpVec5.x);
ASSERT_DOUBLE_EQ(4.0, tmpVec5.y);
ASSERT_DOUBLE_EQ(6.0, tmpVec5.z);
```

上面的代码**演示了框架中 C++ 属性和方法的注册****，以及怎么动态的设置获取对象的属性和调用相关的方法了，最后也演示了如何调用对象的 overload 方法**。我们先不对具体的代码做太细致的拆解，先有个大概印象，注意以下几点：

*   对于反射对象的构建，我们使用的代码：
    

```
runtime::createWithArgs(*metaClass, Args{ 1.0, 2.0, 3.0 });
 ASSERT_TRUE(obj != UserObject::nothing);
```

*   对于属性的获取，我们使用的代码：
    

```
const reflection::Property* fieldX = nullptr;
  metaClass->tryProperty("x", fieldX);
  ASSERT_TRUE(fieldX != nullptr);
  Real x = fieldX->get(obj).to<Real>();
```

*   对于函数的调用，我们使用的代码：
    

```
const reflection::Function* overFunc = nullptr;
  metaClass->tryFunction("operator*", overFunc);
  Value obj3 = runtime::callStatic(*overFunc, Args{obj, 3.0 });
```

其中有几个 “神奇” 的类，Args，Value，UserObject，你注意到这点，那么恭喜你关注到了动态反射实现最核心的点了，我们要做到运行时反射，先得完全类型的“统一”。

**二、实现概述**

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe97lJkV0H922wz1kplaBCbQNtXaqwU5QWjYdWhqAe36Io0cvvr4d7vvXISR6oF3eUPgUOM1kKMXcww/640?wx_fmt=png)

上图是 relection 的工程视图，我们重点关注图中标有蓝色数字的几部分，也大概对他们的作用先做简单的描述:

*   meta: 反射信息的包装呈现的部分。  
    

*   traits: 用于做 compiler time 的类型萃取，通过相关的代码，我们可以方便的对类的成员和函数做编译期分析，如获取函数的返回值类型和参数类型。  
    

*   type_erasure: 为了让反射的使用有统一的接口，我们必须提供基本的类型擦除容器，从而可以运行时动态的调用某个对象的接口或者获取对象的属性。  
    

*   builder: meta 信息不是自动就有的，我们需要一种手段，从原始类去构建相关的 meta 信息 (非侵入式)。  
    

*   runtime: 提供一些相比 meta，更友好的接口，如上面看到的 runtime::createWithArgs() 方法。
    

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95skjJvvMZP0lx7xYDc41KY44T9tLZcHsBkfBGZqG5licrJS46sQOWj613rhegibqaO7H1zKsCPc4aA/640?wx_fmt=png)

几者的大致关系如上图所示:

*   traits，type_erasure: 提供最核心的功能支持。  
    

*   builder: 完成 Compiler time 对原始类的分析和 Meta 信息的输出，实际上是 compiler time->runtime 的桥梁，也相当于是整个反射的脚手架，我们通过 builder 的实现来完成对反射 meta 信息的构成。  
    

*   meta，runtime: 一起完成了运行时信息的表达和提供相关的使用接口。
    

下面我们来具体看一下每一部分的实现。

**三、Meta 实现—类 C# 的表达**

聊到运行时系统，我们首先想到的是弱鸡的 rtti，功能弱也就算了，很多地方还被禁用，能够运行时获取的信息和相关的设施都十分简陋。

另外一点要提的是，C++ 使用的是 compiler time 类型完备的设计方式，运行时类型都会被塌陷掉，圈子里的黑话叫 “擅长裸奔”。

所以我们想要给 C++ 增加动态反射的能力，第一点需要做的就是想办法为它补充足够的运行时信息，也就是类似 C# 的各种 meta data，然后我们就可以愉快的利用动态特性来开发一些更具通用性的序列化反序列化，跨语言支持的功能了。

那么运行时创建操作对象，需要哪些信息? 我们直接给出答案：

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95skjJvvMZP0lx7xYDc41KYqJF7QYMvNfjoBFf7dnINdSsoa3CD8ZE9qqqnvEk7mvibH4Db1p97Zzw/640?wx_fmt=png)

整个反射 meta 部分的设计其实就是为了满足前面说到的，补齐运行时需要的类型相关的各种 meta 信息。

了解了 meta 信息包含的部分，我们来结合代码看一下具体的实现。

### **（一）reflection::Class**

```
class Class : public Type
{
    size_t                          m_sizeof;                // Size of the class in bytes.
    TypeId                          m_id;                    // Unique type id of the metaclass.
    Id                              m_name;                  // Name of the metaclass
    FunctionTable                   m_functions;             // Table of meta functions indexed by name
    FunctionIdTable          m_functions_by_id;   // Table for meta functions indexed by number id
    PropertyTable                   m_properties;            // Table of meta properties indexed by ID
    //Here use function to implement read only static properties, to get a simple implement
    FunctionTable                   m_static_properties;     // Table of meta static properties indexed by ID
    BaseList                        m_bases;                 // List of base metaclasses
    ConstructorList                 m_constructors;          // List of metaconstructors
    Destructor                      m_destructor;            // Destructor (function able to delete an abstract object)
    UserObjectCreator               m_userObjectCreator;     // Convert pointer of class instance to UserObject
    ClassUserdata                   m_userdata;
    public: // reflection
    const Class& base(size_t index) const;
    size_t constructorCount() const;
    const Constructor* constructor(size_t index) const;
    void destruct(const UserObject& uobj, bool destruct) const;
    const Function& function(size_t index) const;
    const Function& function(IdRef name) const;
    const Function* function_by_id(uint64_t funcId) const;
    bool tryFunction(const IdRef name, const Function*& funcRet) const;
    bool tryStaticProperty(const IdRef name, const Function*& funcRet) const;
    size_t propertyCount() const;
    bool hasProperty(IdRef name) const;
    const Property& property(size_t index) const;
    const Property& property(IdRef name) const;
    bool tryProperty(const IdRef name, const Property*& propRet) const;
    size_t sizeOf() const;
    UserObject getUserObjectFromPointer(void* ptr) const;
    bool has_meta_attribute(const IdRef attrName) const noexcept;
    const Value* get_meta_attribute(const IdRef attrName) const noexcept;
    std::string get_meta_attribute_as_string(const std::string_view attrName) const noexcept;
};
```

具体实现跟前面的图基本是一一对应的，这里不展开细述了。

### **（二）reflection::Function**

```
class Function : public Type
{
public:
    IdReturn name() const;
    uint64_t id() const;
    FunctionKind kind() const { return m_funcType; }
    ValueKind returnType() const;
    policy::ReturnKind returnPolicy() const { return m_returnPolicy; }
    virtual size_t paramCount() const = 0;
    virtual ValueKind paramType(size_t index) const = 0;
    virtual std::string_view paramTypeName(size_t index) const = 0;
    virtual TypeId paramTypeIndex(size_t index) const = 0;
    virtual TypeId returnTypeIndex() const = 0;
    virtual bool argsMatch(const Args& arg) const = 0;
protected:
    // FunctionImpl inherits from this and constructs.
    Function(IdRef name);
    Id m_name;                          // Name of the function
    uint64_t m_id;            // Id of the function
    FunctionKind m_funcType;            // Kind of function
    ValueKind m_returnType;             // Runtime return type
    policy::ReturnKind m_returnPolicy;  // Return policy
    const void* m_usesData;
};
```

注意此处的 Function 是一个虚的实现，后面我们会看到真正的 Function 是由 builder 来构建出来的。

另外一点是 meta function 没有像 C# 那样直接给出 Invoke 方法，这个是因为目前的实现针对不同使用场合，类型擦除的函数是不同的，比如对于 lua，类型擦除的函数原型是 lua_CFunction。对于 C++，则是：

```
std::function<Value(Args)>;
```

不同场合不同统一类型的好处是不需要 Wrapper，没有额外的性能开销，但同时也会导致外围的使用变麻烦，这里可能需要根据项目实际情况做一定的调整。

### **（三）reflection::Property**

```
class Property : public Type
{
public:
    IdReturn name() const;
    ValueKind kind() const;
    virtual bool isReadable() const;
    virtual bool isWritable() const;
  Value get(const UserObject& object) const;
  void set(const UserObject& object, const Value& value) const;
protected:
    virtual Value getValue(const UserObject& object) const = 0;
  virtual void setValue(const UserObject& object, const Value& value) const = 0;
protected:
    Id m_name; // Name of the property
    ValueKind m_type; // Type of the property
    TypeId m_typeIndex;
    PropertyImplementType m_implement_type = PropertyImplementType::Unknow;
};
```

这个类跟 reflection::Function 一样，也是一个虚基类，builder 会负责具体的 Property 的实现，真正特化实现的主要是 getValue()，setValue() 这两个虚接口，外部则直接使用 get()，set() 接口来完成对对应属性的获取和设置操作。

### **（四）static property**

static property 的实现比较简单，目前其实是利用 Function 来间接实现的，只提供 read only 类型的 static 变量。这个地方不详细展开了。

**四、traits 实现—基础工具**

这部分实现也是 C++ 新特性引入后 “减法” 效应比较明显的部分, 也是 type erasure 的基础。核心部分主要是两块，TypeTraits 和 FunctionTraits，我们主要侧重于了解两者的功能和使用，忽略实现部分，随着 c++20 的到来，concept 的引入，这部分又可以得到进一步的简化。目前主要是通过 C++ 的 SIFINEA 特性来完成相关的推导实现，更多是细节的处理相关的代码，了解相关的具体实现价值感觉有限，就不做太细致的展开了。

### **（一）TypeTraits**

TypeTraits 主要用于移除类型的 Pointer 和 Reference 等修饰，获取原始类型。另外我们也可以通过其提供的 get()，getPointer() 函数来快速获取预期类型的数据。

```
template <typename T>
struct TypeTraits<T*>
{
static constexpr ReferenceKind kind = ReferenceKind::Pointer;
using Type = T*;
using ReferenceType = T*;
using PointerType = T*;
using DereferencedType = T;
using DataType = typename DataType<T>::Type;
static constexpr bool isWritable = !std::is_const<DereferencedType>::value;
static constexpr bool isRef = true;
static inline ReferenceType get(void* pointer) { return static_cast<T*>(pointer); }
static inline PointerType getPointer(T& value) { return &value; }
static inline PointerType getPointer(T* value) { return value; }
};
```

*   #### **DataType<>**
    

TypeTraits 中通过 DataType<> 间接获取了类型 T 的 DataType(移除 *，& 修饰的类型)，DataType 的实现如下：

```
template <typename T, typename E = void>
struct DataType
{
    using Type = T;
};
// const
template <typename T>
struct DataType<const T> : public DataType<T> {};
template <typename T>
struct DataType<T&> : public DataType<T> {};
template <typename T>
struct DataType<T*> : public DataType<T> {};
template <typename T, size_t N>
struct DataType<T[N]> : public DataType<T> {};
// smart pointer
template <template <typename> class T, typename U>
    struct DataType<T<U>, typename std::enable_if<IsSmartPointer<T<U>, U>::value >::type>
    {
        using Type = typename DataType<U>::Type;
    };
```

*   #### **示例**
    

```
using rstudio::reflection::detail::TypeTraits;
using rstudio::reflection::ReferenceKind;
ASSERT_TRUE(TypeTraits<int>::kind == ReferenceKind::Instance);
ASSERT_TRUE(TypeTraits<float>::kind == ReferenceKind::Instance);
ASSERT_TRUE(TypeTraits<int*>::kind == ReferenceKind::Pointer);
ASSERT_TRUE(TypeTraits<float*>::kind == ReferenceKind::Pointer);
ASSERT_TRUE(TypeTraits<float&>::kind == ReferenceKind::Reference);
ASSERT_TRUE(TypeTraits<Methods&>::kind == ReferenceKind::Reference);
ASSERT_TRUE(TypeTraits<std::shared_ptr<Methods>>::kind == ReferenceKind::SmartPointer);
```

### **（二）FunctionTraits**

FunctionTraits 主要用来完成编译期对函数类型的推导，获取它的返回值类型，参数类型等信息。常见于各种跨语言中间层的实现，如 Lua 的各种 bridge，也是函数类型擦除的起点，我先得知道这个函数本身的信息，才能进一步做 type erasure。我们先来看一个例子：

```
/*
 * Specialization for native callable types (function and function pointer types)
 *  - We cannot derive a ClassType from these as they may not have one. e.g. int get()
 */
template <typename T>
struct FunctionTraits<T,
typename std::enable_if<std::is_function<typename std::remove_pointer<T>::type>::value>::type>
{
    static constexpr FunctionKind kind = FunctionKind::Function;
    using Details = typename function::FunctionDetails<typename std::remove_pointer<T>::type>;
    using BoundType = typename Details::FuncType;
    using ExposedType = typename Details::ReturnType;
    using ExposedTraits = TypeTraits<ExposedType>;
    static constexpr bool isWritable = std::is_lvalue_reference<ExposedType>::value
        && !std::is_const<typename ExposedTraits::DereferencedType>::value;
    using AccessType = typename function::ReturnType<typename ExposedTraits::DereferencedType, isWritable>::Type;
    using DataType = typename ExposedTraits::DataType;
    using DispatchType = typename Details::DispatchType;
    template <typename C, typename A>
    class Binding
    {
        public:
        using ClassType = C;
        using AccessType = A;
        Binding(BoundType d) : data(d) {}
        AccessType access(ClassType& c) const { return (*data)(c); }
        private:
        BoundType data;
    };
};
```

这个是对 function 和 function pointer 的 FucntionTraits<> 特化，其他函数类型的特化提供的能力与这个的特化基本一致，主要包含以下信息：

*   kind: 函数的类型，其实就是 FunctionTraits 的特化实现各类，详见下文。  
    

*   Details: 函数的具体信息，如返回值类型，参数表 tuple<> 等，都存储在其中。  
    

*   BoundType: 函数类型。  
    

*   ExposedType: 返回值类型。  
    

*   ExposedTraits: ExposedType 的 TypeTraits。  
    

*   DataType: ExposedType 的数据类型。  
    

*   DispatchType: 配合 std::function<> 使用，作为 std::function 的模板参数，这样就可以构造一个与原始 Function 类型匹配的 std::function<> 对象了。
    

下文对几个重点属性做一下具体的说明：

*   #### **FunctionTraits<>::kind**
    

FunctionKind 枚举，主要有以下值：

```
/**
 * \brief Enumeration of the kinds of function recognised
 *
 * \sa Function
 */
enum class FunctionKind
{
    None,               ///< not a function
    Function,           ///< a function
    MemberFunction,     ///< function in a class or struct
    FunctionWrapper,    ///< `std::function<>`
    BindExpression,     ///< `std::bind()`
    Lambda              ///< lambda function `[](){}`
};
```

*   #### **FunctionTraits<>::Detatils**
    

Details 本身也是一个模板类型，包含两种实现，FunctionDetails<> 和 MethodDetails<>，两者主要成员基本一致，MethodDetails<> 针对的是类的成员函数，所以会额外多一个 ClassType，两者差异很小，我们就只列出 FunctionDetails<> 的代码了：

```
template <typename T>
struct FunctionDetails {};
template <typename R, typename... A>
struct FunctionDetails<R(*)(A...)>
{
    using ParamTypes = std::tuple<A...>;
    using ReturnType = R;
    using FuncType = ReturnType(*)(A...);
    using DispatchType = ReturnType(A...);
    using FunctionCallTypes = std::tuple<A...>;
};
template <typename R, typename... A>
struct FunctionDetails<R(A...)>
{
    using ParamTypes = std::tuple<A...>;
    using ReturnType = R;
    using FuncType = ReturnType(*)(A...);
    using DispatchType = ReturnType(A...);
    using FunctionCallTypes = std::tuple<A...>;
};
```

实现比较简洁，主要的成员：

*   ParamTypes: 包含函数所有参数的 tuple<>。
    

*   ReturnType: 函数的返回值类型。
    

*   FuncType: 函数指针类型。
    

*   DispatchType: 配合 std::function<> 一起使用，作为 std::function 的模板参数，这样就可以构造一个与原始 Function 类型匹配的函数对象了。
    

*   FunctionCallTypes: 同 ParamTypes，注意对于 MethodDetails<>，这两者是有区别的，ParamTypes 不包含类本身，MethodDetails 首个参数是类本身。
    

#### 使用示例：

```
using rstudio::reflection::detail::FunctionTraits;
using rstudio::reflection::PropertyKind;
using rstudio::reflection::FunctionKind;
ASSERT_TRUE(FunctionTraits<void(void)>::kind == FunctionKind::Function);
ASSERT_TRUE(FunctionTraits<void(int)>::kind == FunctionKind::Function);
ASSERT_TRUE(FunctionTraits<int(void)>::kind == FunctionKind::Function);
ASSERT_TRUE(FunctionTraits<int(char*)>::kind == FunctionKind::Function);
// non-class void(void)
ASSERT_TRUE(FunctionTraits<decltype(func)>::kind == FunctionKind::Function);
// non-class R(...)
ASSERT_TRUE(FunctionTraits<decltype(funcArgReturn)>::kind == FunctionKind::Function);
// class static R(void)
ASSERT_TRUE(FunctionTraits<decltype(&Class::staticFunc)>::kind == FunctionKind::Function);
```

FunctionTraits 的单元测试代码，各种类型的函数的 Traits。

### **（三）ArrayTraits**

ponder 原始的 array traits 主要通过 ArrayMapper 来完成，ArrayMapper 用在了 ArrayPropertyImpl 中，同其他 Traits，我们先以 std::vector<> 的特化实现来看一下 ArrayMapper:

```
template <typename T>
struct ArrayMapper<std::vector<T> >
{
    static constexpr bool isArray = true;
    using ElementType = T;
    static constexpr bool dynamic()
{
        return true;
    }
    static size_t size(const std::vector<T>& arr)
{
        return arr.size();
    }
    static const T& get(const std::vector<T>& arr, size_t index)
{
        return arr[index];
    }
    static void set(std::vector<T>& arr, size_t index, const T& value)
{
        arr[index] = value;
    }
    static void insert(std::vector<T>& arr, size_t before, const T& value)
{
        arr.insert(arr.begin() + before, value);
    }
    static void remove(std::vector<T>& arr, size_t index)
{
        arr.erase(arr.begin() + index);
    }
    static void resize(std::vector<T>& arr, size_t totalsize)
{
        arr.resize(totalsize);
    }
    static constexpr bool support_raw_pointer()
{
        return true;
    }
    static void* ptr(std::vector<T>& arr)
{
        return arr.data();
    }
};
```

ArrayMapper 通过各种版本的特化实现提供对各类型 Array 进行操作的统一接口，如:

*   size()  
    

*   get()  
    

*   set()  
    

*   insert()  
    

*   resize() 等
    

另外对于一个类型，我们也可以简单的通过 ArrayMapper<T>::isArray 来判断它是否是一个数组。

### **（四）其它 Traits**

*   #### **IsSmartPointer<>**
    

```
template <typename T, typename U>
struct IsSmartPointer
{
    static constexpr bool value = false;
};
template <typename T, typename U>
struct IsSmartPointer<std::unique_ptr<T>, U>
{
    static constexpr bool value = true;
};
template <typename T, typename U>
struct IsSmartPointer<std::shared_ptr<T>, U>
{
    static constexpr bool value = true;
};
```

判断一个类型是否为 smart pointer(可以考虑直接使用 concept 实现)。

*   #### **get_pointer()**
    

```
template<class T>
    T* get_pointer(T* p)
{
    return p;
}
template<class T>
    T* get_pointer(std::unique_ptr<T> const& p)
{
    return p.get();
}
template<class T>
    T* get_pointer(std::shared_ptr<T> const& p)
{
    return p.get();
}
```

为 smart pointer 和 raw pointer 提供获取 raw pointer 的统一接口。

**五、type_erasure 实现——统一外观**

我们来回顾一下 Meta 部分介绍的 Property 的两个接口：
----------------------------------

```
virtual Value getValue(const UserObject& object) const = 0;
virtual void setValue(const UserObject& object, const Value& value) const = 0;
```

很容易看到其中的 UserObject 和 Value, 这基本是整个反射 type_erasure 实现最核心的两个对象了。其中 UserObject 用于统一表达通过 MetaClass 创建的对象，Value 则类似 variants，用于统一表达反射支持的所有类型的值。通过这两个类型的引入，我们很好的完成了 getValue() 和 setValue() 对所有类型的统一表达。下面我们来具体介绍一下这两者的实现。

### **（一）UserObject**

区别于 reflection::Class 用于表达 Meta 信息，UserObject 其实就是实际数据部分，两者结合在一起，我们就能够获取完整的运行时的类型表达了，可以根据 ID 或者 name 动态的构造一个对象，并对它进行属性的获取设置或者接口的调用。

```
class UserObject
{
public:
    template <typename T>
    static UserObject makeRef(T& object);
    template <typename T>
    static UserObject makeRef(T* object);
    template <typename T>
    static UserObject makeCopy(const T& object);
    template <typename T>
    static UserObject makeOwned(T&& object);
    UserObject();
    UserObject(const UserObject& other);
    UserObject(UserObject&& other) noexcept;
    template <typename T>
    UserObject(const T& object);
    template <typename T>
    UserObject(T* object);
    UserObject& operator = (const UserObject& other);
    UserObject& operator = (UserObject&& other) noexcept;
    template <typename T>
    typename detail::TypeTraits<T>::ReferenceType get() const;
    void* pointer() const;
    template <typename T>
    const T& cref() const;
    template <typename T>
    T& ref() const;
    const Class& getClass() const;
    Value get(IdRef property) const;
    Value get(size_t index) const;
    void set(IdRef property, const Value& value) const;
    void set(size_t index, const Value& value) const;
    bool operator == (const UserObject& other) const;
    bool operator != (const UserObject& other) const { return !(*this == other); }
    bool operator < (const UserObject& other) const;
    static const UserObject nothing;
private:
    void set(const Property& property, const Value& value) const;
    UserObject(const Class* cls, detail::AbstractObjectHolder* h)
        : m_class(cls)
            , m_holder(h)
        {}
    // Metaclass of the stored object
    const Class* m_class;
    // Optional abstract holder storing the object
    std::shared_ptr<detail::AbstractObjectHolder> m_holder;
};
```

估计跟大部分人想的满屏的复杂的实现不一样，UserObject 的代码其实比较简单，原因前面说过了，UserObject 主要完成对象数据的持有和管理，数据的持有 ponder 原有实现做得比较简单，直接使用 std::shared_ptr<>，通过几种从 AbstractObjectHolder 继承下来的不同用途的 Holder，来完成对数据的持有和生命周期的管理。

UserObject 的接口大概有几类:

*   #### **静态构造方法**
    

```
template <typename T>
static UserObject makeRef(T& object);
template <typename T>
static UserObject makeRef(T* object);
template <typename T>
static UserObject makeCopy(const T& object);
template <typename T>
static UserObject makeOwned(T&& object);
```

静态构造一个 UserObject，几者的主要区别在于对象生命周期管理的差异：

*   makeRef() : 不创建对象，间接持有对象，所以可能会有一个对象被 UserObject 持有的时候，被外界错误的释放导致异常的问题。
    

*   makeCopy(): 区别于 makeRef，Holder 内部会直接创建并持有一个相关对象的拷贝，生命周期安全的实现。
    

*   makeOwned(): 类同 makeCopy()，差别的地方在于会转移外界对象的控制权到 UserObject，也是生命周期安全的实现。
    

*   #### **泛型构造函数**
    

```
template <typename T>
UserObject(const T& object);
template <typename T>
UserObject(T* object);
```

对类型 T 的构造实现，注意内部调用的是前面介绍的 makeCopy() 静态方法，所以通过这种方式构造的 UserObject 是生命周期安全的。

*   #### **到原始 C++ 类型的转换**
    

```
template <typename T>
typename detail::TypeTraits<T>::ReferenceType get() const;
template <typename T>
const T& cref() const;
template <typename T>
T& ref() const;
```

注意转换失败会直接抛出 C++ 异常。

*   #### **Property 相关的便利接口**
    

```
Value get(IdRef property) const;
Value get(size_t index) const;
void set(IdRef property, const Value& value) const;
void set(size_t index, const Value& value) const;
```

对 Property 进行存取操作的快捷接口。

*   #### **其他**
    

```
void* pointer() const;
const Class& getClass() const;
static const UserObject nothing;
```

比较常用的两个特殊接口，其中：

*   pointer(): 用于获取 UserObject 内部存储对象的 raw pointer。
    

*   **getClass(): 用于获取 UserObject 对应的 MetaClass。
    

*   nothing: UserObject 的空值，判断一个 UserObject 是否为空可以直接与该静态变量比较。
    

### **（二）Value**

Value 的实现也很简单，核心代码如下：

```
using Variant = std::variant<
        NoType,
        bool,
        int64_t,
        double,
        reflection::String,
        EnumObject,
        UserObject,
        ArrayObject,
        detail::BuildInValueRef
      >;
Variant m_value; // Stored value
ValueKind m_type; // Ponder type of the value
```

其实就是通过 std::variant<>，定义了一个包含：

*   NoType
    

*   bool
    

*   int64_t
    

*   double
    

*   string
    

*   EnumObject
    

*   UserObject
    

*   ArrayObject
    

*   BuildInValueRef  
    

以上这些类型的一个和类型，然后利用 std::visit() 访问内部的 std::variant 来完成各种操作的一个实现，实现思路比较简单，这样对于反射支持的类型，我们都可以统一用 Value 来进行表达了。

常用的接口如下:

```
ValueKind kind() const;
template <typename T>
T to() const;
template <typename T>
typename std::enable_if_t<detail::IsUserType<T>::value, T&> ref();
template <typename T>
typename std::enable_if_t<!detail::IsUserType<T>::value, T&> ref();
template <typename T>
typename std::enable_if_t<detail::IsUserType<T>::value, const T&> cref() const;
template <typename T>
typename std::enable_if_t<!detail::IsUserType<T>::value, const T&> cref() const;
template <typename T>
bool isCompatible() const;
static const Value nothing;
```

**六、builder 实现——合到一起**

builder 的目的主要就是利用 compiler time 特性，通过类型推导的方式完成从原始类型到 Meta 信息的生成。

**（一）function 的实现**
-------------------

function 的实现请参考 [[reflection function implement]]。

**（二）property 的实现**
-------------------

property 的实现请参考 [[refection property implement]]。

### **（三）总结**

由于 C++ 本身 compiler time 类型其实是相对完备的，给我们提供了一个通过 compiler time 特性来完成 Meta 信息生成的目的，Property 的实现与 function 类同，利用 Traits 和一些其他设施，最后整个 meta class 的信息就被填充完成了，然后我们就能够在运行时对已经注册的类型进行使用了。

**七、runtime 实现——便利的运行时接口**

### **（一）对象创建和删除的辅助方法**

```
template <typename... A>
static inline UserObject create(const Class& cls, A... args);
static inline UserObject createWithArgs(const Class& cls, Args&& args);
using UniquePtr = std::unique_ptr<UserObject>;
inline UniquePtr makeUniquePtr(UserObject* obj);
template <typename... A>
static inline UniquePtr createUnique(const Class& cls, A... args)
static inline void destroy(const UserObject& obj)
```

通过这些方法我们可以完成对反射对象的快速创建和销毁操作。

### **（二）函数调用**

```
template <typename... A>
static inline Value call(const Function& fn, const UserObject& obj, A&&... args);
static inline Value callWithArgs(const Function& fn, const UserObject& obj, Args&& args);
template <typename... A>
static inline Value callStatic(const Function& fn, A&&... args);
static inline Value callStaticWithArgs(const Function& fn, Args&& args);
```

介绍 Function 对象的时候，我们没有发现上面的 invoke 方法，ponder 原始的实现，是间接通过辅助的 call()，callStatic() 方法来完成的对应 meta function 的执行，这个地方应该是可以考虑直接在 Function 上提供对应实现，而不是目前的模式。

**八、Future——思考题**

反射之后呢? 更进一步:

```
// Define the interface of something that can be drawn
struct Drawable : decltype(dyno::requires_(
  "draw"_s = dyno::method<void (std::ostream&) const>
)) { };
// Define an object that can hold anything that can be drawn.
struct drawable {
  template <typename T>
  drawable(T x) : poly_{x} { }
  void draw(std::ostream& out) const
{ poly_.virtual_("draw"_s)(out); }
private:
  dyno::poly<Drawable> poly_;
};
void f(drawable const& d) {
  d.draw(std::cout);
}
struct Square {
  void draw(std::ostream& out) const { out << "Square"; }
};
struct Circle {
  void draw(std::ostream& out) const { out << "Circle"; }
};
int main() {
  f(Square{}); // prints Square
  f(Circle{}); // prints Circle
}
```

需要实现类 rust traits 这种编译期和运行期一致的多态，我们还需要做哪些事情？我们已经具备的特性？还需要的特性？

**九、小结**

其实系统的了解后会发现，随着 C++ 本身的迭代，像反射这种轮子，开发难度变得越来越简单，对比 C++98 年代的 luabind，cpp-framework 中的反射实现代码已经很精简了，而且我们也能发现功能更强大，适应的场合更多了，代码复杂度也大幅下降了，从这个角度看，可以看到 C++ 新特性的迭代，其实是在做减法的，可以让你有更简洁易懂的方式，去表达原来需要更复杂的实现去做的事情。从 14/17 这个节点去学习和了解 Meta Programming，是一个不错的时间点。

**参考资料：**

1.[github ponder 库] 

https://github.com/billyquith/ponder