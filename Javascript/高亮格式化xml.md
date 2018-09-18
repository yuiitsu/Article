# 高亮格式化xml

## 两个关键点
1. 使用DOMParser解析xml
2. 递归遍历xml树，按格式输出每一个节点

## 关于使用DOMParser
此方法目前在IE9以上和其它浏览器里都是支持的，所以这里不在写关于IE9以下不支持的情况, 具体的使用请跳转
[https://developer.mozilla.org/en-US/docs/Web/API/DOMParser](https://developer.mozilla.org/en-US/docs/Web/API/DOMParser)

## Javascript代码

```
/**
 * 格式化xml
 * @param content
 * @returns {*}
 */
this.parse_xml = function(content) {
    let xml_doc = null;
    try {
        xml_doc = (new DOMParser()).parseFromString(content.replace(/[\n\r]/g, ""), 'text/xml');
    } catch (e) {
        return false;
    }

    function build_xml(index, list, element) {
        let t = [];
        for (let i = 0; i < index; i++) {
            t.push('&nbsp;&nbsp;&nbsp;&nbsp;');
        }
        t = t.join("");
        list.push(t + '&lt;<span class="code-key">'+ element.nodeName +'</span>&gt;\n');
        for (let i = 0; i < element.childNodes.length; i++) {
            let nodeName = element.childNodes[i].nodeName;
            if (element.childNodes[i].childNodes.length === 1) {
                let value = element.childNodes[i].childNodes[0].nodeValue;
                let value_color = !isNaN(Number(value)) ? 'code-number' : 'code-string';
                let value_txt = '<span class="'+ value_color +'">' + value + '</span>';
                let item = t + '&nbsp;&nbsp;&nbsp;&nbsp;&lt;<span class="code-key">' + nodeName +
                    '</span>&gt;' + value_txt + '&lt;/<span class="code-key">' + nodeName + '</span>&gt;\n';
                list.push(item);
            } else {
                build_xml(++index, list, element.childNodes[i]);
            }
        }
        list.push(t + '&lt;/<span class="code-key">'+ element.nodeName +'</span>&gt;\n');
    }

    let list = [];
    build_xml(0, list, xml_doc.documentElement);

    return list.join("");
};
```

## css

```
.code-string{color:#993300;}
.code-number{color:#cc00cc;}
.code-boolean{color:#000033;}
.code-null{color:magenta;}
.code-key{color:#003377;font-weight:bold;}
```

## 效果
![image](https://github.com/onlyfu/Blog/blob/master/static/images/javascript/20180812235710.png)

## 注意
DOMParser在解析xml时，如果xml字符串里有些特殊的字符，解出来的树节点有些是不需要的，会倒置遍历节点失败。

## 最后
些方法已用于YuiAPI
