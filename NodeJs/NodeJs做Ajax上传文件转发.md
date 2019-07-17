# NodeJs做Ajax上传文件转发

![](https://www.colorgamer.com/usr/uploads/2019/07/3362909557.jpg)

有时候为了解决跨域或是在开发模式，前台做上传时，需要用到NodeJs来做一次上传的中转。

流程很简单：

**前台发起Ajax上传文件 => 提交到NodeJs的服务上 => NodeJs将请求和数据转发到后端服务上**

这里稍微有一些注意事项，主要分

+ 前台上传请求
+ NodeJs的处理



## 前台上传求（Ajax）

这里只说Ajax请求的情况。如果使用过jQuery就知道，需要在参数上设置`contentType: false`，注意，这并不是说要将请求的Content-Type设置为false，这只是jQuery的一个参数，它的目的是告诉jQuery，你不要设置Content-Type了，让系统自己处理。所以，如果你使用原生的JS Ajax请求，切记不要去设置Content-Type。

Ajax上传文件，如下（这里使用jQuery来示例）：

```javascript 
let formData = new FormData();
formData.append($('#file')[0].files[0]);

$.ajax({
  url: 'http://127.0.0.1/upload',
  data: formData,
  processType: false,
  contentType: false
});
```

这里就3个点：

1. 使用FormData来构建数据
2. processType: false。告诉jQuery不要处理数据，如果是原生JS，可以不需要。
3. contentType: false。告诉jQuery不要设置Content-Type，如果是原生JS，千万不要设置Content-Type。



## NodeJs的处理

NodeJs的处理又分两步：

1. 接收上传数据
2. 发送上传数据



### 接收上传数据

如果使用express，它有现成的包去处理，可以搜索相关内容，这里说的是不使用框架的情况。

同样，需要使用一个包来接收数据：**multiparty**

示例：

```javascript
let multiparty = require('multiparty');

let form = new multiparty.Form();
form.parse(req, function(err, fields, files) {  // req为http server的request对象
  
});
```

回调方法的参数说明：

+ err: 发生错误时会有信息，没有错误值为null
+ fields: 表单数据，是一个map
+ files: 文件数据，是一个list，它包含的主要信息有：
  + files[].fieldName: 上传表单的名称，就是在input设置的name属性
  + Files[].originalFilename: 文件名
  + files[].path: 上传文件的临时文件所在路径

所以，通过fields和files，可以拿到在Ajax的FormData里设置的数据。



## 发送上传数据

要将这些数据发送到后台，又需要将其它设置为multipart/form-data数据，所以这里又需要另一个包来处理：**form-data**

示例(结合接收示例)：

```javascript 
let FormData = require('form-data');
let multiparty = require('multiparty');

let form = new multiparty.Form();
form.parse(req, function(err, fields, files) {  // req为http server的request对象
		let form = new FormData();
  	if (fields) {
      for (var i in fields) {
				if (fields.hasOwnProperty(i)) {
          form.append(i, fields[i][0]);
        }
      }
    }
  	if (files) {
      for (var i in files.file) {
        var item = files.file[i];
        form.append(item['fieldName'], Fs.createReadStream(item['path']));
      }
    }
  
  	form.submit({
      host: 'eg.com',
      port: 80,
      path: '/upload',
      headers: req.headers
    }, function(err, res) {
      var body = '';
      res.on('data', function(d) {
        body += d;
      }).on('end', function() {
        console.log('end'):
      });
    })
});



```

+ 首先，遍历fields将表单数据set到form里
+ 再遍历files，通过path，读取文件，再将文件数据set到form里，其中fieldName参数就是前台页面设置的input name。
+ 最后，通过form.submit方法，提交FormData到后端服务器



## 最后

其实用NodeJs转发上传文件内容不多，但仍然有点绕，主要问题前面已经都有说。那么关于NodeJs转发上传文件的介绍就到这里。