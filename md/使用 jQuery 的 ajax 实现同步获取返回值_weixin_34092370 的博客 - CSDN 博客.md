> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_34092370/article/details/92377998)

直接上具体代码，改下 url, 直接调用 getValue() 函数就可以获取返回值。

```
function getValue(){
	var value;
	$.ajax({
		 async : false,
		 type : "POST",
		 url : "",
		 data : {
			id : id
		 }
	}).done(function(data){
		 value = data;
	});
	return value;
}
```

转载于: https://my.oschina.net/u/3183495/blog/826816