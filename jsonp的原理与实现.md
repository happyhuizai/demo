## jsonp的原理与实现 
1. jsonp是一种跨域通信的手段，它的原理其实很简单：

2. 首先是利用script标签的src属性来实现跨域。

4. 通过将前端方法作为参数传递到服务器端，然后由服务器端注入参数之后再返回，实现服务器端向客户端通信。

> 由于使用script标签的src属性，因此只支持get方法

``````javascript
(function (global) {
    var id = 0,
        container = document.getElementsByTagName('head')[0]
    function jsonp(options) {
        if (!options || !options.url) {
            return
        }
        var scriptNode = document.createElement("script"),
            data = options.data || {},
            url = options.url,
            callback = options.callback,
            fnName = "jsonp" + id++;
        data["callback"] = fnName
        var params = []
        for (const key in data) {
            if (data.hasOwnProperty(key)) {
                params.push(encodeURIComponent(key) + "=" + encodeURIComponent(data[key]));
            }
        }
        url = url.indexOf("?") > 0 ? (url + "&") : (url + "?");
        url += params.join("&");
        scriptNode.src = url;

        global[fnName] = function (ret) {
            callback && callback(ret);
            container.removeChild(scriptNode);
            delete global[fnName];
        }
        scriptNode.onerror = function () {
            callback && callback({
                error: "error"
            });
            container.removeChild(scriptNode);
            global[fnName] && delete global[fnName];
        }
        scriptNode.type = "text/javascript";
        container.appendChild(scriptNode)
    }
    global.jsonp = jsonp;
})(this)

jsonp({
    url: "http://127.0.0.1:8888",
    data: {
        id: 1
    },
    callback: function (ret) {
        console.log(ret);
    }
});
jsonp({
    url: "http://127.0.0.1:8888",
    data: {
        id: 1
    },
    callback: function (ret) {
        console.log(ret);
    }
});

//服务器端代码
var http = require('http');
var url = require('url');
var data = {
    'data': 'world'
};
http.createServer(function (req, res) {
    var params = url.parse(req.url, true);
    if (params.query.callback) {
        console.log(params.query.callback);
        var str = params.query.callback + '(' + JSON.stringify(data) + ')';
        res.end(str);
    } else {
        res.end('Hello World\n');
    }
}).listen(8888);
console.log('Server running at http://127.0.0.1:8888/');

``````
