---
layout: post
title: "Nodejs: forEach vs for"
date: 2014-11-24 18:19:44 +0800
comments: true
categories: node.js
---
#背景
今天想要用node.js实现一个小程序，程序任务是：遍历某folder下所有文件，打印出是Directory的文件名。Node最大的特征是异步非阻塞，所以程序中用到的fs函数，全部想用异步的调用
#错误版本
<pre>
<code>
var fs = require('fs')
var homePath = './'

fs.readdir(homePath, function(err, files){
	for (var i = files.length - 1; i >= 0; i--) {
		var file = files[i]
		fs.stat(file, function(err, stats){
			if(stats.isFile()){
				console.log("2:" + file);
			}
		})
	};
});
</code>
</pre>
然后打印出的结果是
<pre>
<code>
:nodejs$ node forCallbackPractice.js
2:FileUploadServer.js
2:FileUploadServer.js
2:FileUploadServer.js
2:FileUploadServer.js
2:FileUploadServer.js
</code>
</pre>
错误：结果恒为一个文件名！
出现这种情况的原因是，在for循环中的fs.stat方法是非阻塞的，因此不会在次停留，只是注册了一个查询stat事件。等到查询stat事件执行完毕返回callback的时候，循环已经执行到了末尾，file对象恒为files中的最后一个被遍历对象。call back中拿到的file对象已经更改过，不是注册事件时的对象了。
#正确版本
JavaScript中的array支持一个函数叫做forEach
####forEach
<pre>
array1.forEach(callbackfn[, thisArg])
</pre>
####参数
<table>
<tr><td>参数</td><td>定义</td></tr>
<tr><td>array1</td><td>必选。 一个数组对象</td></tr>
<tr><td>callbackfn</td><td>必选。 最多可以接受三个参数的函数。 对于数组中的每个元素，forEach 都会调用 callbackfn 函数一次</td></tr>
<tr><td>thisArg</td><td>可选。 callbackfn 函数中的 this 关键字可引用的对象。 如果省略 thisArg，则 undefined 将用作 this 值</td></tr>
</table>
####备注
对于数组中出现的每个元素，forEach 方法都会调用 callbackfn 函数一次（采用升序索引顺序）。其实forEach是Node中实现并行loop的一种方式。找到一篇文章，有很好的讲到node的[Control flow](http://book.mixu.net/node/ch7.html)。Mark在这里，阅读完毕在深入研究。
####Code
<pre><code>
var fs = require('fs')
var homePath = './'

fs.readdir(homePath, function(err, files){
	files.forEach(function(file){
		fs.stat(file, function(err, stats){
			if(stats.isFile()){
				console.log("1: " + file);
			}
		})
	})
});
</code></pre>
####output
<pre><code>
:nodejs$ node forCallbackPractice.js
1: FileUploadServer.js
1: Leve4.mp4
1: eclipse_ClearCase.zip
1: forCallbackPractice.js
1: MonkeyTalkSDK
</code></pre>


