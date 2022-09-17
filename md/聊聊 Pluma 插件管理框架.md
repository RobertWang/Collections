> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NzM0MjcyMQ==&mid=2650160750&idx=5&sn=d2d09366f86943b4a24271fa8af7f985&chksm=bed9e50089ae6c161c4f4a8ab9c501a7afd1df69139f78f883d82566b87b19a62521c3fc2cb3&mpshare=1&scene=1&srcid=0915eTTeJxa7LDFpSEGKqjLt&sharer_sharetime=1663188433870&sharer_shareid=8a467675e94cd5b11b6640b7770d6cc6#rd)

来源 | OSCHINA 社区

作者 | 悠然红茶 - 侯亮

原文链接：https://my.oschina.net/youranhongcha/blog/5577374

**1. 概述**
=========

 Pluma 是一个用 C++ 开发的可用于管理插件的开源架构，其官网地址为：http://pluma-framework.sourceforge.net/。该架构是个轻量级架构，非常易于理解。  
    Pluma 架构有以下基本概念：  
1）插件的外在行为体现为一个纯虚类，可以叫作插件接口；  
2）继承于同一个插件接口的若干派生类，被认为属于同一种插件，可以叫作插件类；  
3）每一个插件接口或插件类都有个一一对应的 Provider 类，其中，插件接口对应的 Provider 类里会定义一个特殊字符串常量：PLUMA_PROVIDER_TYPE，表示这一类 “插件 Provider” 共同的类型名称，而这个类型名称其实就是插件接口的类名字符串。  
4）多个插件类可以被放入一个插件动态库中，而这个动态库文件名（不包括后缀部分）可以叫作 “插件名”。  
5）插件机制使用者可以在自己的架构中包含一个 Pluma 管理类，该类支持从所指定的位置加载一个或多个插件动态库，并将每个插件类对应的 Provider，记录进内部的表中。  
6）插件机制使用者可以在合适时机，利用 Pluma 获取内含的插件 Provider，并调用某个插件 Provider 的 create () 函数，创建出对应的插件对象。  
7）使用完插件对象后，不要忘了 delete 它。

 现在我们画一张示意图：

![](https://mmbiz.qpic.cn/mmbiz_png/dkwuWwLoRKibgRbSwmoBoBQMibb2QfWZdgOXwRKJc2TwB8SIQwzCn5cMzpvaMcWaPY16lX6WIeFzGDrLVjcb1uDA/640?wx_fmt=png)

**2. Pluma 管理类**
================

我们刚刚也说了，插件机制使用者可以包含一个 Pluma 管理类。该类继承于 PluginManager 类。

【pluma-1.1/include/pluma/PluginManager.hpp】

```
class PLUMA_API PluginManager{
public:
   ~PluginManager();
   bool load(const std::string& path);
   bool load(const std::string& folder, const std::string& pluginName);
   int loadFromFolder(const std::string& folder, bool recursive = false);
   bool unload(const std::string& pluginName);
   void unloadAll();
   bool addProvider(Provider* provider);
   void getLoadedPlugins(std::vector<const std::string*>& pluginNames) const;
   bool isLoaded(const std::string& pluginName) const;
protected:
   PluginManager();
   void registerType(const std::string& type, unsigned int version, 
                      unsigned int lowestVersion);
   const std::list<Provider*>* getProviders(const std::string& type) const;
private:
   static std::string getPluginName(const std::string& path);
   static std::string resolvePathExtension(const std::string& path);
private:
   typedef bool fnRegisterPlugin(Host&);
   typedef std::map<std::string,DLibrary*> LibMap;
   LibMap libraries;  ///< Map containing the loaded libraries
   Host host;         ///< Host app proxy, holding all providers
};

```

 从上面的 load () 函数和 loadFromFolder () 函数可以看出，插件管理器既允许用户单独加载某个插件动态库，也允许批量性加载某个目录下所有的插件动态库。另外，值得注意的是，getProviders () 函数是 protected 的成员，也就是说，这套架构是不希望用户直接使用这个 PluginManager 类的，即便用了，你也拿不到 Provider。正确的做法是，使用 PluginManager 的子类：Pluma 管理类。

 另外，上面的成员变量 libraries，就是记录所有已加载的插件动态库的映射表。而成员变量 host 则负责记录每个插件类对应的 Provider 信息。之所以被称为 host（宿主），是针对插件而言的。也就是说插件本身实际上是没资格知道其真实宿主的全貌的，它只能访问和它相关的很小一部分数据而已，因此 Pluma 将这一小部分数据整理成一个 host 代理，供插件使用。    Pluma 管理类的代码截选如下：  
【pluma-1.1/include/pluma/Pluma.hpp】

```
class Pluma: public PluginManager{
public:
   Pluma();
   template<typename ProviderType>
   void acceptProviderType();
   template<typename ProviderType>
   void getProviders(std::vector<ProviderType*>& providers);
};
#include <Pluma/Pluma.inl>

```

请大家注意上面代码中最后一行，这个 Pluma.hpp 还真是有点手黑，偷偷摸摸 #include 了个 Pluma.inl 文件，其实展开来就是 acceptProviderType () 和 getProviders () 这两个模板函数的实现。Pluma.inl 文件的内容如下：  
【pluma-1.1/include/pluma/Pluma.inl】

```
inline Pluma::Pluma(){
   // Nothing to do
}
template<typename ProviderType>
void Pluma::acceptProviderType(){
   PluginManager::registerType(
       ProviderType::PLUMA_PROVIDER_TYPE,
       ProviderType::PLUMA_INTERFACE_VERSION,
       ProviderType::PLUMA_INTERFACE_LOWEST_VERSION
   );
}
template<typename ProviderType>
void Pluma::getProviders(std::vector<ProviderType*>& providers){
   const std::list<Provider*>* lst 
              = PluginManager::getProviders(ProviderType::PLUMA_PROVIDER_TYPE);
   if (!lst) return;
   providers.reserve(providers.size() + lst->size());
   std::list<Provider*>::const_iterator it;
   for (it = lst->begin() ; it != lst->end() ; ++it)
       providers.push_back(static_cast<ProviderType*>(*it));
}

```

 看到了吧，重新定义了个 getProviders ()，还搞成一个模板函数，在函数体内会反过来通过模板参数，进一步得到所涉及的插件 Provider 的 PLUMA_PROVIDER_TYPE 信息，这个技巧挺重要。也就是说，外界传来的是 vector<ProviderType>，而函数内部可以推断出 ProviderType::PLUMA_PROVIDER_TYPE。将 PLUMA_PROVIDER_TYPE 传入父类的 PluginManager::getProviders () 函数，就可以拿到符合所指类型的所有 Provider。

 我们画一张 Pluma 简图，后面再细说相关细节：

![](https://mmbiz.qpic.cn/mmbiz_png/dkwuWwLoRKibgRbSwmoBoBQMibb2QfWZdgMdicda18iaGInhuhur3k1HibW2nnGGPAYMiacEwViaA7Gkq4XQNCNCrCDUQ/640?wx_fmt=png)

同一类插件类，会对应一个 ProviderInfo 节点，该节点内部的 providers 列表，记录着同属一类的若干 Provider。

**2.1 Host 代理**
---------------

【pluma-1.1/include/pluma/Host.hpp】

```
class PLUMA_API Host{
friend class PluginManager;
friend class Provider;
public:
   bool add(Provider* provider);
private:
   Host();
   ~Host();
   bool knows(const std::string& type) const;
   unsigned int getVersion(const std::string& type) const;
   unsigned int getLowestVersion(const std::string& type) const;
   void registerType(const std::string& type, unsigned int version, unsigned int lowestVersion);
   const std::list<Provider*>* getProviders(const std::string& type) const;
   void clearProviders();
   bool validateProvider(Provider* provider) const;
   bool registerProvider(Provider* provider);
   void cancelAddictions();
   bool confirmAddictions();
private:
   struct ProviderInfo{
       unsigned int version;
       unsigned int lowestVersion;
       std::list<Provider*> providers;
   };
   typedef std::map<std::string, ProviderInfo > ProvidersMap;
   typedef std::map<std::string, std::list<Provider*> > TempProvidersMap;
   ProvidersMap knownTypes;       ///< Map of registered types.
   TempProvidersMap addRequests;  ///< Temporarily added providers
};

```

正如前文所说，Host 代理是针对插件而言的。而 Host 只有一个 public 成员函数 add ()，说明其主要对外行为就是让插件将对应的 provider 注册进 Host。

**3. 插件类和其对应的 Provider 类**
==========================

 在说了一大堆插件管理类代码后，现在终于要开始说插件部分了。前文已经说过，插件的外在行为体现为一个纯虚类，可以叫作插件接口。我们现在就以 Pluma 源码中给出的例子为准，来说明一些细节。

**3.1 Warrior 接口和 WarriorProvider 类**
-------------------------------------

 Pluma 中的插件接口例子是 Warrior，其源码截选如下：  
【pluma-1.1/example/src/interface/Warrior.hpp】

```
#include <Pluma/Pluma.hpp>
class Warrior{
public:
   virtual std::string getDescription() = 0;
   // (...)
};
PLUMA_PROVIDER_HEADER(Warrior);

```

这个接口里只象征性的写了一个成员函数 getDescription ()，大家明白意思即可。

 需要注意的是类定义之后的那句 PLUMA_PROVIDER_HEADER，这个宏负责定义和插件接口对应的 Provider 类。相关的宏定义如下：  
【pluma-1.1/include/pluma/Pluma.hpp】

```
#define PLUMA_PROVIDER_HEADER(TYPE)\
PLUMA_PROVIDER_HEADER_BEGIN(TYPE)\
virtual TYPE* create() const = 0;\
PLUMA_PROVIDER_HEADER_END
#define PLUMA_PROVIDER_HEADER_BEGIN(TYPE)\
class TYPE##Provider: public pluma::Provider{\
private:\
   friend class pluma::Pluma;\
   static const unsigned int PLUMA_INTERFACE_VERSION;\
   static const unsigned int PLUMA_INTERFACE_LOWEST_VERSION;\
   static const std::string PLUMA_PROVIDER_TYPE;\
   std::string plumaGetType() const{ return PLUMA_PROVIDER_TYPE; }\
public:\
   unsigned int getVersion() const{ return PLUMA_INTERFACE_VERSION; }
#define PLUMA_PROVIDER_HEADER_END };

```

基于这些宏定义，我们可以将 PLUMA_PROVIDER_HEADER (Warrior) 展开为：

```
class WarriorProvider: public pluma::Provider{
private:
   friend class pluma::Pluma;
   static const unsigned int PLUMA_INTERFACE_VERSION;
   static const unsigned int PLUMA_INTERFACE_LOWEST_VERSION;
   static const std::string PLUMA_PROVIDER_TYPE;
   std::string plumaGetType() const{ return PLUMA_PROVIDER_TYPE; }
public:
   unsigned int getVersion() const{ return PLUMA_INTERFACE_VERSION; }
   virtual Warrior* create() const = 0;
};

```

代码很清晰，为 Warrior 接口声明一个配套的 WarriorProvider 类。这个类里包含着重要的 PLUMA_PROVIDER_TYPE 常量，以及最关键的 create () 函数。

 Warrior 的实现文件更加简单：  
【pluma-1.1/example/src/interface/Warrior.cpp】

```
#include "Warrior.hpp"
PLUMA_PROVIDER_SOURCE(Warrior, 1, 1);

```

也在使用宏，展开宏后可见：

```
const std::string WarriorProvider::PLUMA_PROVIDER_TYPE = "Warrior";
const unsigned int WarriorProvider::PLUMA_INTERFACE_VERSION = 1;
const unsigned int WarriorProvider::PLUMA_INTERFACE_LOWEST_VERSION = 1;

```

因为 Warrior 本身是个纯虚类，所以 WarriorProvider 里也不用实现 create () 函数。

**3.2 Warrior 派生类和派生 Provider**
-------------------------------

 在 pluma 源码的例子中，提供了三个 Warrior 派生类，SimpleWarrior、Eagle 和 Jaguar。默认的是 SimpleWarrior，它被集成进 example/src/host 目录。也就是说，即便我们一个额外的插件库都不提供，示例至少还可以使用 SimpleWarrior。而 Eagle 和 Jaguar 则位于 example/src/plugin 目录，可以打包进一个插件动态库。

【pluma-1.1/example/src/host/SimpleWarrior.hpp】

```
#include "Warrior.hpp"
class SimpleWarrior: public Warrior{
public:
   std::string getDescription(){
       return "Commoner: leaded by calpoleque";
   }
};
PLUMA_INHERIT_PROVIDER(SimpleWarrior, Warrior);

```

前文我们已经看到，对于插件接口（Warrior）来说，用到的宏是 PLUMA_PROVIDER_HEADER(Warrior)，现在针对实际插件类（SimpleWarrior），会用到另一个宏 PLUMA_INHERIT_PROVIDER(SimpleWarrior, Warrior)。这个宏的定义如下：  
【pluma-1.1/include/pluma/Pluma.hpp】

```
#define PLUMA_INHERIT_PROVIDER(SPECIALIZED_TYPE, BASE_TYPE)\
class SPECIALIZED_TYPE##Provider: public BASE_TYPE##Provider{\
public:\
   BASE_TYPE * create() const{ return new SPECIALIZED_TYPE (); }\
};

```

展开后可见：

```
class SimpleWarriorProvider: public WarriorProvider{
public:
   Warrior * create() const { return new SimpleWarrior (); }
};

```

很简单，就是在完成 Provider 的核心使命，提供一个创建插件类对象的 create () 函数。与 SimpleWarriorProvider 类似，另外两个 Warrior 派生类 Eagle 和 Jaguar 大体也是这么写的。示意图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/dkwuWwLoRKibgRbSwmoBoBQMibb2QfWZdgqib6L3YwIAh4VchQbDichoVMUzZZrD6fFcs3zibHfgBTpKxycNicXJlKKw/640?wx_fmt=png)

在研究 Pluma 所给示例时，我已事先将 Pluma 封装成静态库了，现在要把 Eagle 和 Jaguar 编译并封装成一个动态库，就需要链接 Pluma 静态库，除此之外，还需要编译其他一些辅助文件，列举如下：  
1）Connector.cpp  
2）dllmain.cpp  
3）Eagle.hpp  
4）Jaguar.hpp  
5）Warrior.cpp  
其中 Connector.cpp 文件，是插件动态库向外界 Host 注册自己所有 Provider 的地方。它必须实现一个 connect () 函数，代码截选如下：

```
#include <Pluma/Connector.hpp>
#include "Eagle.hpp"
#include "Jaguar.hpp"
PLUMA_CONNECTOR
bool connect(pluma::Host& host){
   host.add( new EagleProvider() );
   host.add( new JaguarProvider() );
   return true;
}

```

 我们先不要着急分析上面的 connect () 动作，可以先跟着我看看插件的加载流程，后文我们就会知道，connect () 只是加载流程的一环而已。

**4. 插件加载流程**
=============

 我们看一下 Pluma 架构所给例子的 main () 函数，就可以了解插件的加载流程了：

```
int main() 
{
   pluma::Pluma pluma;
   pluma.acceptProviderType<WarriorProvider>();   // 表明用户感兴趣东西，添加ProviderInfo
   pluma.load("plugins", "PlumaDemoWarriorPlugin");  // 加载动作，向ProviderInfo里加料
   std::vector<WarriorProvider*> providers;
   pluma.getProviders(providers);
   std::vector<WarriorProvider*>::iterator it;
   for (it = providers.begin(); it != providers.end(); ++it) {
       Warrior* warrior = (*it)->create();
       std::cout << warrior->getDescription() << std::endl;
       delete warrior;
   }
   pluma.unloadAll();
   std::cout << "Press any key to exit";
   std::cin.ignore(10000, '\n');
   return 0;
}

```

其中和加载插件相关的句子主要就是 pluma.acceptProviderType 和 pluma.load 两句了。前者主要负责在 Host 的 knownTypes 映射表中添加一个 ProviderInfo 节点，后者负责加载插件动态库，并将动态库里匹配的 Provider 指针记入 ProviderInfo 节点。

**4.1 pluma.acceptProviderType<>()**
------------------------------------

 我们先说 pluma.acceptProviderType 一句。在前文介绍 Pluma.inl 文件的内容时，我们已经看到一个叫作 acceptProviderType 的模板函数了，当时没有细说，现在我把它的代码再贴一下：  
【pluma-1.1/include/pluma/Pluma.inl】

```
template<typename ProviderType>
void Pluma::acceptProviderType(){
   PluginManager::registerType(
       ProviderType::PLUMA_PROVIDER_TYPE,
       ProviderType::PLUMA_INTERFACE_VERSION,
       ProviderType::PLUMA_INTERFACE_LOWEST_VERSION
   );
}

```

里面调用的是 PluginManager 基类的 registerType () 函数。我们前文主要关心的是 PLUMA_PROVIDER_TYPE，现在再说一下后两个参数。PLUMA_INTERFACE_VERSION 表示管理器当前应该使用的插件接口的版本，因为我们不能确定更高版本的插件接口会不会增加或删除成员函数，所以这个值其实是个限定值，如果后续用户尝试加载更高版本的插件，那么是无法通过校验的。第三个参数 PLUMA_INTERFACE_LOWEST_VERSION 则是限定最低值，如果尝试加载比这个值更低版本的插件，肯定也是不会通过的。

 在刚刚看到的 main () 函数里，是这样写的：

```
pluma.acceptProviderType<WarriorProvider>();

```

也就是说，Pluma 插件管理器对 Warrior 接口对应的 WarriorProvider 类感兴趣。而当初定义 Warrior 时，在 Warrior.cpp 文件里的确指明了 WarriorProvider 能限定的当前版本号和最低版本号：

```
PLUMA_PROVIDER_SOURCE(Warrior, 1, 1);

```

 这些类型信息、版本号限定信息都会被注册在 Host 的 knownTypes 映射表中，每种接口类型对应一个 ProviderInfo 节点。注册动作的代码如下：  
【pluma-1.1/src/pluma/PluginManager.cpp】

```
void PluginManager::registerType(const std::string& type, unsigned int version,
                                 unsigned int lowestVersion){
    host.registerType(type, version, lowestVersion);
}

```

【pluma-1.1/src/pluma/Host.cpp】

```
void Host::registerType(const std::string& type, unsigned int version, 
                        unsigned int lowestVersion){
    if (!knows(type)){
        ProviderInfo pi;
        pi.version = version;
        pi.lowestVersion = lowestVersion;
        knownTypes[type] = pi;
    }
}

```

当然，新加的 ProviderInfo 节点的 providers 列表是个空列表，待后续再添加 Provider * 内容。

**4.2 pluma.load()**
--------------------

 接着，我们继续看 main () 函数里调用的 pluma.load ()，其实调用的是其父类 PluginManager 的 load ()。相关代码截选如下：  
【pluma-1.1/src/pluma/PluginManager.cpp】

```
bool PluginManager::load(const std::string& path){
   std::string plugName = getPluginName(path);
   std::string realPath = resolvePathExtension(path);
   DLibrary* lib = DLibrary::load(realPath);
   ......
   fnRegisterPlugin* registerFunction;
   registerFunction = reinterpret_cast<fnRegisterPlugin*>
                                        (lib->getSymbol("connect"));
   ......
   if (!registerFunction(host)){
       ......
       return false;
   }
   if (host.confirmAddictions())
       libraries[plugName] = lib;
   else{
       ......
       return false;
   }
   return true;
}

```

可以看到，一开始就在着手加载动态库，并调用动态库里的 connect () 函数。前文我们实际上已经列举过示例代码里的 connect () 函数了，现在再贴一次：

```
PLUMA_CONNECTOR
bool connect(pluma::Host& host){
   host.add( new EagleProvider() );
   host.add( new JaguarProvider() );
   return true;
}

```

 前文在阐述到 connect () 时，暂时没有细说 add () 动作，现在我们来看看它的代码：  
【pluma-1.1/src/pluma/Host.cpp】

```
bool Host::add(Provider* provider){
   if (provider == NULL){
       fprintf(stderr, "Trying to add a null provider.\n");
       return false;
   }
   if (!validateProvider(provider)){
       delete provider;
       return false;
   }
   // 临时放进 addRequests表
   addRequests[ provider->plumaGetType() ].push_back(provider);
   return true;
}

```

 上面代码中那个 plumaGetType () 函数其实是 Provider 的私有成员，一般人访问不了，但 Host 是它的友元类，所以可以访问。代码中会先校验待添加的 Provider 是否合格，如果合格则以 plumaGetType () 返回值为 key 值，并向临时映射表 addRequests 中添加该 Provider 指针。所谓合格是指，这个 Provider 的类型是 Host 感兴趣的，并且其版本号也是合适的。

 值得注意的是，待添加的 Provider*，只是临时先放进一个 addRequests 映射表中。addRequests 映射表的定义如下：  
【pluma-1.1/include/pluma/Host.hpp】

```
typedef std::map<std::string, std::list<Provider*> > TempProvidersMap;
......
TempProvidersMap addRequests;

```

那么这个临时性的 addRequests 映射表的内容会怎样处理呢？说起来也简单，会被 “搬移” 进 Host 的 knownTypes 映射表中某个 ProviderInfo 的内部列表去。main () 在调用完 connect () 函数后，调用的 confirmAddictions () 就是做这个事情的：  
【pluma-1.1/src/pluma/Host.cpp】

```
bool Host::confirmAddictions(){
   if (addRequests.empty()) return false;
   TempProvidersMap::iterator it;
   for( it = addRequests.begin() ; it != addRequests.end() ; ++it){
       std::list<Provider*> lst = it->second;
       std::list<Provider*>::iterator providerIt;
       for (providerIt = lst.begin() ; providerIt != lst.end() ; ++providerIt){
           knownTypes[it->first].providers.push_back(*providerIt);
       }
   }
   TempProvidersMap().swap(addRequests);  // 清空addRequests表
   return true;
}

```

 我们画一张调用关系图看看：

![](https://mmbiz.qpic.cn/mmbiz_png/dkwuWwLoRKibgRbSwmoBoBQMibb2QfWZdgRpdTFz297ogt62HibMKcMWcsr7SuicmUotzN0KbZ9PDDuAYDoDJJyVXA/640?wx_fmt=png)

 我们可以通过这张调用关系图回顾一下，主要流程就是在加载插件动态库，并执行动态库里的 connect () 函数。该函数会将动态库里可用的所有 Provider * 记入 Host 的 knownTypes 映射表中。同时，动态库对应的 DLibrary 对象，也会插入 Pluma 管理类内部的 libraries 映射表中。

 为了巩固知识，我们把前文的两张图再整合一下。

![](https://mmbiz.qpic.cn/mmbiz_png/dkwuWwLoRKibgRbSwmoBoBQMibb2QfWZdgnTUWtiaHkuYvGUUZ6fDRmcGzUPIovCpiaqgqQeNiaejlkfANElMGbhBmg/640?wx_fmt=png)

**5. 使用插件 Provider**
====================

**5.1 pluma.getProviders()**
----------------------------

 在 Providers 都添加进 Pluma 管理类后，我们就可以在需要时获取 provider 了，为此 Pluma 类提供了 getProviders () 函数：  
【pluma-1.1/src/pluma/PluginManager.cpp】

```
const std::list<Provider*>* PluginManager::getProviders(const std::string& type) const{
   return host.getProviders(type);
}

```

【pluma-1.1/src/pluma/Host.cpp】

```
const std::list<Provider*>* Host::getProviders(const std::string& type) const{
   ProvidersMap::const_iterator it = knownTypes.find(type);
   if (it != knownTypes.end())
       return &it->second.providers;
   return NULL;
}

```

代码很简单，就是帮使用者把感兴趣的某类插件 Provider 全部找出来。如果当初我们已经通过 acceptProviderType () 注册了对应的类型（PLUMA_PROVIDER_TYPE），那么至少可以拿到一个 list，否则就只能拿到 NULL 了。如果我们可以拿到若干 Provider，就可以调用其 create () 函数创建对应的插件对象了。

 当工作做完后，用户应该及时 delete 掉之前创建出的插件对象。在程序退出之前，用户应该调用 pluma.unloadAll () 删除所有插件 Provider 及 DLibrary 对象。DLibrary 对象析构时，会自动关闭已经打开的动态链接库。  
【pluma-1.1/src/pluma/PluginManager.cpp】

```
void PluginManager::unloadAll(){
   host.clearProviders();
   LibMap::iterator it;
   for (it = libraries.begin() ; it != libraries.end() ; ++it){
       delete it->second;   // delete掉DLibrary对象
   }
   libraries.clear();
}

```

```
void Host::clearProviders(){
   ProvidersMap::iterator it;
   for (it = knownTypes.begin() ; it != knownTypes.end() ; ++it){
       std::list<Provider*>& providers = it->second.providers;
       std::list<Provider*>::iterator provIt;
       for (provIt = providers.begin() ; provIt != providers.end() ; ++provIt){
           delete *provIt;   // delete掉Provider对象
       }
       std::list<Provider*>().swap(providers);
   }
}

```

**6. 结束**
=========

 至此，Pluma 架构的主体代码就分析完毕了，希望对大家有所帮助。