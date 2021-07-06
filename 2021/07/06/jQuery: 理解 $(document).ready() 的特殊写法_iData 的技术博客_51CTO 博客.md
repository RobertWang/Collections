> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/idata/1119589)

> jQuery: 理解 $(document).ready() 的特殊写法，再看到 $()，要感谢编写 jQuery 的大神们（以及其他类似框架的大神们），是他们的努力，让世界变得完美。

© 著作权归作者所有：来自 51CTO 博客作者 ztfriend 的原创作品，如需转载，请注明出处，否则将追究法律责任  
https://blog.51cto.com/idata/1119589

看书时注意到下面两条语句的功效是相同的，

```
$(function(){alert("hello!");});  
$(document).ready(function(){alert("hello!");}); 

```

这个特殊写法就是用 $() 代替 $(document).ready()，类似于 _(有差异)_window.onload 弹出个窗口：

[![](https://s4.51cto.com/attachment/201301/152320726.jpg)](https://s4.51cto.com/attachment/201301/152320726.jpg) 

查看 jQuery1.8.3 源代码，是这样封装的：

```
(function( window, undefined ) {  
     
})( window );  

```

下列语句把封装在内部的 jQuery 先赋给 window.$，紧接着再赋给 window.jQuery。这意味着在实际使用时 window.$ 和 window.jQuery 是一回事。因为 $ 这个符号只有 1 个字母，比 jQuery 短，所以更常用一些，但**要注意到 $ 非 jQuery 所独有，节约字母的代价是增加了命名冲突的风险。**

```
 
window.jQuery = window.$ = jQuery; 



```

下面是 jQuery 的初始化语句（注意到此时函数并未执行）：

```
 
jQuery = function( selector, context ) {  
     
    return new jQuery.fn.init( selector, context, rootjQuery );  
} 

```

找到 jQuery.fn 的定义，这是一个对象，其中有一个叫 init 的函数元素： 

```
jQuery.fn = jQuery.prototype = {  
    constructor: jQuery,  
    init: function( selector, context, rootjQuery ) {  
        var match, elem, ret, doc;  
 
         
        if ( !selector ) {  
            return this;  
        }  
 
         
        if ( selector.nodeType ) {  
            this.context = this[0] = selector;  
            this.length = 1;  
            return this;  
        }  
 

```

继续下去，init 中有一段逻辑：

```
 
 
} else if ( jQuery.isFunction( selector ) ) {  
    return rootjQuery.ready( selector );  
} 

```

晕了晕了，rootjQuery 的定义又回到了 jQuery：

```
 
rootjQuery = jQuery(document); 

```

有点递归的意思了，嗯，就是递归。jQuery 不仅仅是一个函数，而且还是一个递归函数。

如果调用 jQuery 时输入的是一个函数，例如文章开头提到的：

```
$(function(){alert("hello!");}); 

```

那么这个函数就会走到 rootjQuery 那里，再回到 jQuery，执行 jQuery(document).ready。而 $ 与 jQuery 是一回事，这样就解释了 $(inputFunction) 可以代替 $(document).ready(inputFunction)。

现在还不想结束此文，我的问题是 $(document) 做了什么？嗯，还是要进入到 jQuery.fn.init，确认存在 nodeType 属性，达到 “Handle $(DOMElement)” 的目的。怎么 Handle 呢？具体就是把输入参数_（此时为 document）_赋值给 this 的 context 属性，然后再返回 this。也就是说，$(document) 执行完了返回的还是 jQuery，但是情况发生了变化，具体就是 context 属性指向了输入参数_（此时为 document）_。暂时还不明白绕这么大个圈子为 context_（上下文）_属性赋值有何意义？

接下去的问题可能会是 $(document).ready 和 window.onload 的区别？提取 ready 函数的定义如下：

```
ready: function( fn ) {  
     
    jQuery.ready.promise().done( fn );  
 
    return this;  
}, 

```

阅读代码探究 promise 是有点晕啊，想到自己的 iJs 工具包了，打印 jQuery.ready.promise() 如下：

    [Object] jQuery.ready.promise()  
        |--[function] always  
        |--[function] done  
        |--[function] fail  
        |--[function] pipe  
        |--[function] progress  
        |--[function] promise  
        |--[function] state  
        |--[function] then

进一步打印整理 done 函数代码如下（这下彻底晕了~~）：

```
function() {   
    if ( list ) {   
         
        var start = list.length;   
        (function add( args ) {   
            jQuery.each( args, function( _, arg ) {   
                var type = jQuery.type( arg );   
                if ( type === "function" ) {   
                    if ( !options.unique || !self.has( arg ) ) { list.push( arg ); }   
                } else if ( arg && arg.length && type !== "string" ) {   
                     
                }   
            });   
        })( arguments );   
         
         
        if ( firing ) {   
            firingLength = list.length;   
             
             
        } else if ( memory ) {   
            firingStart = start; 
　　　　　　　fire( memory );   
        }   
    }   
    return this;   
} 

```

好在代码不长，看起来关键就在于 fire 函数了。嗯，找回一丝清醒了。在上面的 done 函数里面可以注意到使用了默认的 arguments 变量，将注入的函数 push 到了 list 数组。下面是 fire 函数：

```
fire = function( data ) {  
    memory = options.memory && data;  
    fired = true;  
    firingIndex = firingStart || 0;  
    firingStart = 0;  
    firingLength = list.length;  
    firing = true;  
    for ( ; list && firingIndex < firingLength; firingIndex++ ) {  
        if ( list[ firingIndex ].apply( data[ 0 ], data[ 1 ] ) === false && options.stopOnFalse ) {  
            memory = false;  
            break;  
        }  
    }  
    firing = false;  
    if ( list ) {  
        if ( stack ) {  
            if ( stack.length ) {  
                fire( stack.shift() );  
            }  
        } else if ( memory ) {  
            list = [];  
        } else {  
            self.disable();  
        }  
    }  
} 

```

可以看到代码中对 list 数组里面使用了 apply。用 iJs 包调试可发现 data[0] 就是 document 对象，也就是说，调用 $(**myFunction**) 的结果是在 document 对象上执行了 **myFunction**。因为 list 是个数组，所以也就不难理解 $() 其实是多次输入，一次执行。

最后，回过头来阅读 promise 源代码，关于 $() 输入函数的执行时机的秘密就在这里了：

```
jQuery.ready.promise = function( obj ) {  
    if ( !readyList ) {  
 
        readyList = jQuery.Deferred();  
 
         
         
         
        if ( document.readyState === "complete" ) {  
             
            setTimeout( jQuery.ready, 1 );  
 
         
        } else if ( document.addEventListener ) {  
             
            document.addEventListener( "DOMContentLoaded", DOMContentLoaded, false );  
 
             
            window.addEventListener( "load", jQuery.ready, false );  
 
         
        } else {  
             
            document.attachEvent( "onreadystatechange", DOMContentLoaded );  
 
             
            window.attachEvent( "onload", jQuery.ready );  
 
             
             
            var top = false;  
 
            try {  
                top = window.frameElement == null && document.documentElement;  
            } catch(e) {}  
 
            if ( top && top.doScroll ) {  
                (function doScrollCheck() {  
                    if ( !jQuery.isReady ) {  
 
                        try {  
                             
                             
                            top.doScroll("left");  
                        } catch(e) {  
                            return setTimeout( doScrollCheck, 50 );  
                        }  
 
                         
                        jQuery.ready();  
                    }  
                })();  
            }  
        }  
    }  
    return readyList.promise( obj );  
}; 

```

从代码的注释中可以看到这段代码在消除 bug 的过程中还是颇费了些心思的。查看其中一个网址 [http://bugs.jquery.com/ticket/12282#comment:15](http://bugs.jquery.com/ticket/12282#comment:15)，是关于 IE9/10 的一个 bug（document ready is fired too early on IE 9/10），好在已经解决。

绕了这么多弯子，整个事情看起来就是这样，如果每一个浏览器都能有 document.readyState === "complete"，就简单了。再看到 $()，要感谢编写 jQuery 的大神们_（以及其他类似框架的大神们）_，是他们的努力，让世界变得完美。