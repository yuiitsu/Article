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

其中的done，即用pending方法包起了我们想回调执行的方法，当计数器为0时，就会执行它，那我们得改进一下pending方法：

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

这个问题解决了，我们要思考一下，如何让这样的方法封装重用，不然，每次都写pending不是很不科学吗？

下面看看UnJs（我的一个基于NodeJs的Web开发框架）的处理方式，应用于模板解析中的子模板操作：

```
unjs.asyncSeries = function(task, func, callback) {
	var taskLen = task.length;
	if (taskLen <= 0) {
		return;
	}

	var done = unjs.pending(callback);
	for(var i = 0; i < taskLen; i++) {
		func(task[i], done);
	}
}
```

asyncSeries有三个参数，意思是：

> task: 需要处理的对象，比如需要读取的文件，它是一个列表，如果不是列表，或列表长度为0，它将不会执行

> func: 异步方法，比如fs.readFile，就是通过它传进去的

> callback: 我们希望回调的方法

done和前面同理，它传给了func，但并没有执行，因为希望应用端能可控制参数，所以让应用端去执行。

再看看处理子模板时的操作：

```
var subTemplate = [];
var patt = /\{\% include \'(.+)\' \%\}/ig;
while(sub = patt.exec(data)) {
	var subs = sub;
	subTemplate.push([subs[0], subs[1]]);
}

unjs.asyncSeries(subTemplate, function(item, callback) {
	fs.readFile('./template/' + item[1], 'utf-8', callback(item[0]));
}, function(data) {
	for(var key in data) {
		html = html.replace(key, data[key]);
	}
});
```

subTemplate这个列表，是根据对子模板的解析生成的数据，它是一个二维的数组，每个子项的第一个值为子模板的调用文本，即：{% include 'header.html' %}这样的字符串，第二个参数为子模板文件名，即：header.html

asyncSeries的第二个参数是的callback，实际上是第三个参数，也就是我们希望执行的回调函数经过pending处理的回调方法，如前面所说，在asyncSeries内部，它并没有运行，而是到这里运行的，即：callback(item[0])，带上了参数，因为后面还要根据这个参数将父模板中调用子模板的字符串替换为对应子模板的内容。

这样子，只要需要连续异步时，就可以使用asyncSeries方法来处理了。因为异步的关系，程序的流程有点绕，可能开始不太好理解，即使熟悉了，也有可能突然想不明白，没关系，比如，第二个参数中的callback实际是第三个参数生成的，开始可能你就会想，这个callback倒底是啥。还有就是pending的两个return，也是不太好理解的，需要多想想。

好了，连续异步的回调使用Js函数的高级特性完成了。但NodeJs的异步性着实让程序的控制很成问题，诸如还有连续异步，但要传值的操作等，这些都是可以通过这样的思路，变化一下即可实现的。

希望本文对被NodeJs的异步操作搞得头晕的同学们有些许的帮助。

