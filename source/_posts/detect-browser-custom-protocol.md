---
title: JavaScript检查浏览器自定义协议
date: 2017-01-17 13:12:07
tags:
- 检查浏览器自定义协议
description: JavaScript检测浏览器是否可以通过一个自定义协议打开一个本地程序。如果本地程序安装了则由浏览器通知打开这个程序，否则提示尚未安装程序，指引去下载。
---

## 需求
JavaScript检测浏览器是否可以通过一个自定义协议打开一个本地程序。
如果本地程序安装了则由浏览器通知打开这个程序，否则提示尚未安装程序，指引去下载。

## 解决方案
你肯定知道常用的http、https、ftp协议。
不知道你是否注意过迅雷的`thunder://`、电驴的`ed2k://`下载链接，浏览器访问`thunder://`开头的地址它会先打开迅雷，然后进行下载。
	
检测本地是否安装了应用程序是难点，因为没有一套通用的代码或方法可以兼容泛滥的浏览器厂商、浏览器版本。

网上很多的解决办法是开发浏览器插件，然后通过浏览器插件访问注册表进行判断！擦，小坑还没填上，又给自己挖一个大坑！可以想象，浏览器插件也很难兼容泛滥的浏览器厂商、浏览器版本，而且还要判断浏览器是否已经安装了插件，没安装还要提示安装，浏览器安全级别高了，可能还安装不上。
	
那有没有简单一点的办法？有！百度`detect browser custom proctocl`，可以搜到一些资料，其中有一个GitHub项目办法就很好，[custom-protocol-detection](https://github.com/ismailhabib/custom-protocol-detection)。
	
它通过各浏览器隐性或显性打开一个地址后的行为进行判断。这个办法你应该不陌生，在ajax出现以前，为了不刷新页面，应该使用过iframe提交表单。

如果打开协议地址成功，则说明本地已经安装协议对应的应用程序，否则尚未安装。
	
各浏览器隐性或显性打开一个地址后：
1.如果打开成功后，当前window会失去焦点，否则打开失败，不失去焦点，比如IE9、IE11、Chrome；
2.如果打开失败会抛出异常，否则不会抛出异常，比如Firefox；
	
IE8不让隐性打开，那就弹出窗口，然后再关闭。
基于Chrome内核的360安全浏览器的极速模式和360极速浏览器不管成功或失败都不会失去焦点...，暂时还没有好的解决办法！

## 代码
protocolcheck.js
```
(function(window) {

    function _registerEvent(target, eventType, cb) {
        if (target.addEventListener) {
            target.addEventListener(eventType, cb);
            return {
                remove: function () {
                    target.removeEventListener(eventType, cb);
                }
            };
        } else {
            target.attachEvent(eventType, cb);
            return {
                remove: function () {
                    target.detachEvent(eventType, cb);
                }
            };
        }
    }

    function _createHiddenIframe(target, uri) {
        var iframe = document.createElement("iframe");
        iframe.src = uri;
        iframe.id = "hiddenIframe";
        iframe.style.display = "none";
        target.appendChild(iframe);

        return iframe;
    }
	
    function openUriWithHiddenFrame(uri, failCb, successCb) {

        var timeout = setTimeout(function () {
			//需要先删除handler，否则如果failCb()执行alert会失去焦点
			handler.remove();
            failCb();
        }, 1000);

        var iframe = document.querySelector("#hiddenIframe");
        if (!iframe) {
            iframe = _createHiddenIframe(document.body, "about:blank");
        }

        var handler = _registerEvent(window, "blur", onBlur);

        function onBlur() {
            clearTimeout(timeout);
            handler.remove();
            successCb();
        }

        iframe.contentWindow.location.href = uri;
    }
	
    function openUriWithTimeoutHack(uri, failCb, successCb) {
        var timeout = setTimeout(function () {
			handler.remove();
            failCb();
        }, 1000);

        //handle page running in an iframe (blur must be registered with top level window)
        var target = window;
        while (target != target.parent) {
            target = target.parent;
        }

        var handler = _registerEvent(target, "blur", onBlur);

        function onBlur() {
            clearTimeout(timeout);
            handler.remove();
            successCb();
        }
		
		window.location = uri;
		
    }
	
    function openUriWithTimeoutHackFor360JS(uri, failCb, successCb) {
        var timeout = setTimeout(function () {
			handler.remove();
            failCb();
        }, 1000);

        //handle page running in an iframe (blur must be registered with top level window)
        var target = window;
        while (target != target.parent) {
            target = target.parent;
        }

        var handler = _registerEvent(target, "blur", onBlur);

        function onBlur() {
            clearTimeout(timeout);
            handler.remove();
            successCb();
        }
		
		window.location = uri;
		onBlur();//360手动失去焦点
		
    }	
	
    function openUriUsingFirefox(uri, failCb, successCb) {
        var iframe = document.querySelector("#hiddenIframe");

        if (!iframe) {
            iframe = _createHiddenIframe(document.body, "about:blank");
        }

        try {
            iframe.contentWindow.location.href = uri;
            successCb();
        } catch (e) {
            if (e.name == "NS_ERROR_UNKNOWN_PROTOCOL") {
                failCb();
            }
        }
    }	

    function openUriUsingIEInOlderWindows(uri, failCb, successCb) {
		var iev = getInternetExplorerVersion();
        if (iev === 10) {
            //openUriUsingIE10InWindows7(uri, failCb, successCb); Win7下IE10不好使
			openUriInNewWindowHack(uri, failCb, successCb);
        } else if (iev === 9 || iev === 11) {
            openUriWithHiddenFrame(uri, failCb, successCb);
        } else{
			//console.log("ie others");
			openUriInNewWindowHack(uri, failCb, successCb);
		}
    }
	
    function openUriUsingChrome(uri, failCb, successCb) {
		//假设MimeTypes长度大于30时为360极速浏览器或360安全浏览器极速模式，暂时没找到更好的判断方法
		//基于chrome的360，window.location后不触发失去焦点事件，所以只能做到安装协议时打开，但不安装没有提示
		if(window.navigator.mimeTypes.length > 30){
			//openUriInNewWindowHack(uri, failCb, successCb);
			//openUriWithTimeoutHack(uri, failCb, successCb);
			openUriWithTimeoutHackFor360JS(uri, failCb, successCb);
		}else{
			//chrome
			openUriWithTimeoutHack(uri, failCb, successCb);
		}
    }	

    function openUriUsingIE10InWindows7(uri, failCb, successCb) {
        var timeout = setTimeout(failCb, 1000);
        window.addEventListener("blur", function () {
            clearTimeout(timeout);
            successCb();
        });

        var iframe = document.querySelector("#hiddenIframe");
        if (!iframe) {
            iframe = _createHiddenIframe(document.body, "about:blank");
        }
        try {
            iframe.contentWindow.location.href = uri;
        } catch (e) {
            failCb();
            clearTimeout(timeout);
        }
    }

    function openUriInNewWindowHack(uri, failCb, successCb) {
        var myWindow = window.open('', '', 'width=0,height=0');

        myWindow.document.write("<iframe src='" + uri + "'></iframe>");

        setTimeout(function () {
            try {
                myWindow.location.href;
                myWindow.setTimeout("window.close()", 100);
                successCb();
            } catch (e) {
				myWindow.close();
				failCb();
            }
        }, 1000);
    }
    
    function openUriWithMsLaunchUri(uri, failCb, successCb) {
        navigator.msLaunchUri(uri,
            successCb,
            failCb
        );
    }
    
    function checkBrowser() {
		//console.log(navigator);
        var isOpera = !!window.opera || navigator.userAgent.indexOf(' OPR/') >= 0;
        return {
            isOpera   : isOpera,
            isFirefox : typeof InstallTrigger !== 'undefined',
            isSafari  : Object.prototype.toString.call(window.HTMLElement).indexOf('Constructor') > 0,
            isChrome  : !!window.chrome && !isOpera,
            isIE      : /*@cc_on!@*/false || !!document.documentMode // At least IE6
        }
    }

	//下面浏览器判断360时有问题，可以改成jquery判断
    function getInternetExplorerVersion() {
        var rv = -1;
        if (navigator.appName === "Microsoft Internet Explorer") {
            var ua = navigator.userAgent;
            var re = new RegExp("MSIE ([0-9]{1,}[\.0-9]{0,})");
            if (re.exec(ua) != null)
                rv = parseFloat(RegExp.$1);
        }
        else if (navigator.appName === "Netscape") {
            var ua = navigator.userAgent;
            var re = new RegExp("Trident/.*rv:([0-9]{1,}[\.0-9]{0,})");
            if (re.exec(ua) != null) {
                rv = parseFloat(RegExp.$1);
            }
        }
        return rv;
    }

    window.protocolCheck = function(uri, failCb, successCb) {
        function failCallback() {
            failCb && failCb();
        }

        function successCallback() {
            successCb && successCb();
        }

        if (navigator.msLaunchUri) { //for IE and Edge in Win 8 and Win 10
            openUriWithMsLaunchUri(uri, failCb, successCb);
        } else {
            var browser = checkBrowser();

            if (browser.isFirefox) {
                openUriUsingFirefox(uri, failCallback, successCallback);
            } else if (browser.isChrome) {
                openUriUsingChrome(uri, failCallback, successCallback);
            } else if (browser.isIE) {
                openUriUsingIEInOlderWindows(uri, failCallback, successCallback);
            } else {
                //not supported, implement please
            }
        }
    }
} (window));
```

## 使用方法
example.html
```
<!DOCTYPE html>
<html>

<head lang="en">
    <meta charset="UTF-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Custom Protocol Detection</title>
</head>

<body>
    <h1>Click one of these labels:</h1>
	<input id = "fb" type = "text" />
    <div href="blahblah:randomstuff">Non-exist protocol
    </div>
    <div href="mailto:johndoe@somewhere.com">Send email
    </div>
    <script src="jquery-1.11.2.min.js"></script>
    <script src="protocolcheck.js"></script>
    <script>
        $(function () {
            $("div[href]").click(function (event) {
                window.protocolCheck($(this).attr("href"),
                    function () {
                        alert("protocol not recognized");
                    });
                event.preventDefault ? event.preventDefault() : event.returnValue = false;
            });
        });
    </script>
</body>

</html>
```