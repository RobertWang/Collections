> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ShTs8rGRWJIlYdgI8TMuyw)

> 什么是 Go 语言的切片在 Go 语言中，切片（Slice）是一种基于数组的动态数据结构。切片提供了一种方便且灵活的

**什么是 Go 语言的切片**

在 Go 语言中，切片（Slice）是一种基于数组的动态数据结构。切片提供了一种方便且灵活的方式来操作变长序列的元素。切片扩展了数组的概念，它可以动态分配和扩容，而不需要像数组一样提前声明固定的长度。

切片由三个部分组成：

1.  指针（Pointer）：指向底层数组的起始位置。
    
2.  长度（Length）：表示切片当前包含的元素个数。
    
3.  容量（Capacity）：表示底层数组从切片起始位置到结束位置的总元素个数。
    

切片的特点：

1.  切片是引用类型，可以像数组一样传递和赋值给其他变量。
    
2.  切片可以使用索引访问和修改元素。
    
3.  切片的长度可以动态改变，它会自动扩容以容纳更多的元素。
    
4.  切片的容量会根据扩容机制自动调整，但可以通过预先指定切片的容量来优化性能。
    
5.  切片可以使用内置的 append() 函数在末尾添加元素。
    

**切片的增删改查**  

1.  增加元素：可以使用内置的 append() 函数来在切尾添加一个或多个元素。示例：
    
    ```
    slice := []int{1, 2, 3}
    slice = append(slice, 4) // 添加一个元素
    slice = append(slice, 5, 6) // 添加多个元素
    
    
    ```
    
2.  删除元素：可以使用切片的切片操作或者 copy() 函数来删除指定位置的元素。示例：
    

*   切片的切片操作：
    
    ```
    slice := []int{1, 2, 3, 4, 5}
    slice = append(slice[:2], slice[3:]...) // 删除索引为2的元素
    
    
    ```
    
*   copy() 函数删除元素：
    
    ```
    slice := []int{1, 2, 3, 4, 5}
    index := 2
    slice = append(slice[:index], slice[index+1:]...) // 删除索引为2的元素
    
    
    ```
    

4.  修改元素：可以通过索引直接修改切片中的元素。示例：
    
    ```
    slice := []int{1, 2, 3, 4, 5}
    slice[2] = 6 // 将索引为2的元素修改为6
    
    
    ```
    
5.  查找元素：可以通过遍历切片或使用内置的 range 关键字来查找元素。示例：
    

*   遍历切片：
    
    ```
    slice := []int{1, 2, 3, 4, 5}
    for i := 0; i < len(slice); i++ {
        if slice[i] == 3 {
            // 找到了索引为2的元素
            break
        }
    }
    
    
    ```
    
*   使用 range 关键字：
    
    ```
    slice := []int{1, 2, 3, 4, 5}
    for index, value := range slice {
        if value == 3 {
            // 找到了索引为2的元素
            break
        }
    }
    
    ```
    

**切片如何进行扩容**

Go 语言切片的扩容机制是通过 append() 函数来实现的。

当切片容量不够时，Go 语言会自动扩容切片。具体的扩容机制如下：

1.  判断切片是否需要扩容：
    

*   如果切片的长度小于其容量，则无需扩容。
    
*   如果切片的长度等于其容量，则需要扩容。
    

3.  计算新的切片容量：
    

*   如果原切片的容量小于 1024，则新的切片容量将扩大为原切片容量的 2 倍。
    
*   如果原切片的容量大于等于 1024，则新的切片容量将扩大为原切片容量的 1.25 倍。
    

5.  创建新的底层数组：
    

*   根据新的切片容量，创建一个新的底层数组，长度为新的切片容量。
    

7.  将原切片中的元素复制到新的底层数组中。
    
8.  将新的底层数组作为切片的底层数组，更新切片的指针、长度和容量。
    
9.  返回新的切片。
    

这种机制可以有效地减少内存的浪费，提高了切片的使用效率。同时，这种扩容机制也会导致切片在添加元素时可能会触发内存重新分配和复制，因此在需要频繁添加元素的场景下，建议使用预先指定切片容量的方式初始化切片，以避免频繁的扩容操作。

**源码介绍**

```
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
切片扩容大概源码
// growslice 函数 handle slice growth in append.
func growslice(et *_type, old slice, cap int) slice {
    // 根据现有的切片长度和容量，计算新的切片容量
    newcap := old.cap
    doublecap := newcap + newcap
    if cap > doublecap {
        newcap = cap
    } else {
        if old.len < 1024 {
            newcap = doublecap
        } else {
            for newcap < cap {
                newcap += newcap / 4
            }
        }
    }
    // 分配新的数组空间
    var overflow bool
    capmem, overflow = math.MulUintptr(et.size, uintptr(newcap))
    newcap, capmem, overflow = newmmap(et.size, newcap, capmem, overflow)
    if overflow || newcap < cap {
       panic(errorString("growslice: cap out of range"))
    }
    // 创建新的切片，并复制旧的切片中的数据
    var p unsafe.Pointer
    p = mallocgc(capmem, et, true)
    var newcap int
    newcap = int(capmem / et.size)
    return slice{p, old.len, newcap}
}

```