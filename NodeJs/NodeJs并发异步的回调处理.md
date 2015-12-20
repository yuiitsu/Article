# NodeJs并发异步的回调处理

这里说并发异步，并不准确，应该说连续异步。NodeJs单线程异步的特性，直接导致多个异步同时进行时，无法确定最后的执行结果来回调。举个简单的例子：

```
for(var i = 0; i < 5; i++) {
	fs.readFile('file', 'utf-8', function(error, data){});
}
```

连续发起了5次读文件的异步操作，很简单，那么问题来了，我怎么确定所有异步都执行完了呢？因为要在它们都执行完后，才能进行之后的操作。相信有点经验的同学都会想到使用记数的方式来进行，但如何保证记数正确又是一个问题。仔细想想：

回调是一个函数，每个异步操作时将计数器+1，当每个异步结束时将计数器-1，通过判断计数器是否为0来确定是否执行回调。这个逻辑很简单，需要一个相对于执行时和回调时的全局变量作为计数器，而且要在传给异步方法是执行+1的操作，而且之后将返回一个用来回调的函数，有点绕，不过看看Js函数的高级用法：

```
var pending = (function() {
	var count = 0;
	return function() {
		count++;
		return function() {
			count--;
			if (count === 0) {
				// 全部执行完毕
			}
		}
	}
});
```

当pending调用时，即pending()，比如：

```
var done = pending();
```

这时计数变量count即被初始化为0，返回的函数附给了done，这时如果执行done()，会是什么？是不是直接执行pending返回的第一个函数，即：pending()()，这个执行又是什么，首先将计数变量count+1，又返回了一个函数，这个函数直接当做callback传给异步的方法，当执行这个callback的时候，首先是将计数变量count-1，再判断count是否为0，如果为0即表示所有的异步执行完成了，从而达到连续的异步，同一回调的操作。

关键就在两个return上，简单的说：

> 第一个return的函数是将count+1，接着返回需要回调的函数

> 第二个return的函数就是需要回调的函数，如果它执行，就是将count-1，然后判断异步是否全部执行完成，完成了，就回调

看个实际点的例子，读取多个文件的异步回调：

```
var fileName = ['1.html', '2.html', '3.html'];

var done = pending(function(fileData) {
	console.log('done');
	console.log(fielData);
});

for(var i = 0; i < fileName.lenght; i++) {
	fs.readFile(fileName[i], 'utf-8', done(fileName[i]));
}
```

其中的done，即用pending方法包起了，我们想回调执行的方法，当计数器为0时，就会执行它，那我们得改进一下pending方法：

```
var pending = (function(callback) {
	var count = 0;
	var returns = {};

	console.log(count);
	return function(key) {
		count++;
		console.log(count);
		return function(error, data) {
			count--;
			console.log(count);
			returns[key] = data;
			if (count === 0) {
				callback(returns);
			}
		}
	}
});
```

callback即为我们的回调函数，当var done = pending(callback)时，done其实已为第一个return的函数，它有一个参数，可以当做返回的值的下标，所以在循环体中done(fileName[i])，把文件名传了进去。这个done()是直接执行的，它将count+1后，返回了要传给异步方法的回调函数，如前面所说，这个回调函数里会根据计数变量来判断是否执行我们希望执行的回调函数，而且把文件的内容传给了它，即returns。好了，运行一下，相信能够准确的看到运行结果。

```
0
1
2
3
2
1
0
done
{"1.html": "xxx", "2.html": "xxx", "3.html": "xxx"}
```

从计数上明显能看出，从0-3再到0，之后就是我们的回调函数输出了done和文件的内容。
