# Cordova简解

## Cordova简介
Cordova是一个开发移动端的开源框架，它允许开发人员利用web开发技术来进行跨平台移动端开发的工作。Cordova是一个基于HTML、CSS和JavaScript的，用于创建跨平台移动应用程序的快速开发平台。它使开发者能够通过JavaScript来使用用iPhone、Android等主流智能手机的核心功能——包括地理定位、加速器、联系人等。

## 基于Cordova的Hybrid App架构
Cordova可以看做是web app的包裹器。简单来讲，Cordova可以让开发人员利用丰富的前端经验开发移动应用。前端的代码运行在Webview中，其中Android是android.webkit.WebView，IOS是UIWebView。Cordova作为包裹webview的包裹器，起到了桥梁作用，使得运行在webview中的js代码和native进行交互。同时在编写前端Web App代码的时候，可以结合一些前端框架，例如jquery、bootstrap、angularjs等。从下面的官方结构图可以看出基于Cordova的app的组成情况：

![Cordova APP Architecture](https://github.com/ToBeNumerOne/cordova/blob/master/cordova.png)

一个Cordova的Hybrid App主要包括以下两部分：

1. Cordova Application
	Cordova Application是Cordova框架独立于不同手机操作系统的一个封装层。具	体包括：
	
	* Web app（包括具体的app的HTML/JS/CSS等源代码）和配置文件，该配置文件配置	app的相关信息，如所使用的插件、app的名字等。
	* Cordova框架已经封装好的核心插件（如相机、存储等系统调用）模块，这块是	Cordova作为web app和native交互的核心部分。当然，开发人员也可以基于它的插件	体系、插件规范，扩展适合自己业务需求的插件。除去Cordova框架提供的核心插件和开	发人员自己开发的插件，还有一些比较好的第三方插件。
	
2. Mobile OS

	该部分就是具体的手机操作系统层了，也就是所谓的native端。Cordova目前支持大部	分的智能手机OS：ios、android、wp、firefoxos、ubuntu等等。
	
基于上述结构图可以让我们一目了然的了解Cordova app总体的技术架构。

## Cordova跨平台概念的理解
Cordova预先帮我们封装了各种插件，这些插件提供了对于mobile os上最常用的api调用，然后以统一的JavaScript api形式提供给web app的开发者调用。对于web app的开发者来说，无需关注系统底层调用实现细节，也就实现了所谓的“跨平台”。实际上，各平台涉及到本地能力的调用，以插件形式被封装了。
## Cordova.js源码解读
Cordova.js主要是通过github上面的项目通过运行grunt命令打包生成。具体地址是[这里](https://github.com/apache/cordova-js)。源代码通过grunt生成适用于各个平台的cordova.js文件，生成的文件处于/pkg目录下。下图是cordova-js的源码结构图：

![Cordova JS Architecture](https://github.com/ToBeNumerOne/cordova/blob/master/cordovajs.jpg)

1. 模块化框架的源码
2. define各个模块
3. 最后通过首次require(‘cordova’)模块，初始化各个模块，因为cordova模块又require其他模块，通过模块间的依赖关系完成所有模块的初始化工作。同时把cordova模块这个对象赋值给window的cordova全局变量。
4. 通过require得到各个模块的实例对象，但是js和native之间的通信关系还没有确立。他们之间的通讯关系是通过bootstrap来确定。首先， bootstrap 通过 channel 模块注册监听了 onNativeReady 事件。这个事件由 Native 层触发，用来表示 Native 层准备完毕 ( 可以接受 plugin 调用 ) 。在 android 平台上面 ,onNativeReady 是在 WebView 的 onPageFinished 回调中触发的。

在Cordova中，native到webview的桥是在init模块的bootstrap方法中初始化的，对应的源代码文件是src/scripts/bootstrap.js文件。该文件中将boot方法附加到onNativeReady事件上。bootstrap方法做了几乎所有的事情。首先，它将common文件夹中定义的所有的对象注入到window全局变量中。最后bootstrap方法调用平台特有的初始化方法。至此，Cordova已经全部初始化完毕。接下来就是等待DOMContentLoaded事件来确保页面被正常载入。之后，Cordova释放deviceready事件，从而开发人员可以安全的使用Cordova API。Cordova-Js源代码中，可以看到所有平台通用的JavaScript模块文件夹src/common/，各个平台特有的js模块，例如js和native交互模块的文件夹src/legacy-exec/android/。同时还有适用于几乎所有平台的模块化模块以及cordova模块。鉴于Cordova.js中主要是由事件模块、js与native交互模块、web端通用的模块以及初始化模块等组成，所以本文侧重结合源码讲解Cordova的事件模块、js-native交互模块、以及Cordova自身定义的模块化机制。

## Cordova模块化机制
Cordova的模块化机制类似于AMD的requirejs的模块化，在其源代码中定义了一个简单的模块化框架，位于src/scripts/require.js，该模块主要定义了require和define两个函数。其中define表示定义模块，第一个参数是所定义模块的唯一标识id(模块名)，第二个参数是构建该模块的工厂函数。Require函数通过id，初始化模块、获取已经定义过的模块。
例如，在源码中，可以看到定义的各个模块，如cordova/builder、cordova/init等模块，首先在开始的时候定义依赖的模块，再次通过module.exports向外暴露api。这样在别的模块require该模块的时候可以使用这些api。

## Cordova的事件处理机制
源码中的cordova/channel模块定义cordovajs的事件处理机制。该机制简单来讲，就是会把每个事件类型，包装成一个Channel对象。Channel对象的原型prototype中定义了事件的监听器，同时给予了监听器一个标识guid。Unsubscribe和fire函数分别用来注销监听器和触发事件。触发事件会引起监听器的广播操作，fireArgs用来保证在监听器在只能触发一次的情况下获取正确的广播参数。同时事件的Channel对象本身有一个监听是否注册、注销事件监听器的操作。分别是onSubscribe和onUnsubscribe，对应为为该事件注册监听器和注销监听器。channel对象是该事件模块对外暴露的公共api。该模块是通过channel对象向外暴露api的。其中join 这个工具方法的第二个参数是个 Channel 数组。当且仅当所有的 Channel 事件都被 fire 后 ,join 的监听才会被回调。create 是个构造事件的工厂方法。新构造的 Channel 事件会被放置在 channel 对象中。在channel.create('onCordovaReady')后,便可以便捷的通过 channel[‘onCordovaReady’] 来方便的访问对应类型的 Channel 对象了。deviceReadyMap,deviceReadyArray,waitForInitialization,initializeComplete它们决定了onDeviceReady事件在何时被触发。于是bootstrap.js 中我们看到决定onDeviceReady事件的一段代码：

```
channel.join(function() {    	channel.onDeviceReady.fire();    }, channel.deviceReadyChannelsArray);
```

## Cordova中js与native交互
因为Cordova适配于几乎所有主流的智能机系统，所有会存在针对各个主流系统的cordova.js。本文主要讲解android和ios。首先明确cordova中插件的概念。插件起到js调用native功能的中间件的作用。可以理解为运行在webview中js的延伸，通过插件可以使用诸如打电话、照相机、通讯录等native端的功能。在我们的web app中，如果想要使用打电话等功能，则必须使用js调用插件，从而通过插件使用native的这些功能。在cordova.js中，我们可以找到一个命名为cordova/exec的模块。该模块是调用native端，执行native端代码的入口。在各个平台下，exec的实现方法不同。

* Android
	* 常规的交互方式
	
	  在android平台上，js和native交互的常规方式是如下：
	  	* js到native的交互
	  	  
	  	  在java类中，声明类的方法的时候，加上@JavascriptInterface注解，然后		  调用mWebView.addJavascriptInterface(new 定义的类， name)来将注解		  过的方法所在的类对象注入到js中，从而通过name.注解过的方法来调用。
	  	* native到js的交互
	  	
	  	  通过mWebView.loadUrl(‘javascript:xxxx’)来执行一段js。Android 		  4.4之后，可以使用mWebView.evaluateJavascript(script, callback)      		  方式实现native到js的交互

	* Cordova中定义的交互方式
		* js到native的交互

			在实例化CordovaWebView的时候, CordovaWebView对象会去创建一个属于			当前CordovaWebView对象的插件管理器PluginManager对象，一个消息队列			NativeToJsMessageQueue对象，一个JavascriptInterface对象			ExposedJsApi，并将ExposedJsApi对象添加到CordovaWebView中，			JavascriptInterface名字为：_cordovaNative。			
			在创建ExposedJsApi时需要CordovaWebView的PluginManager对象和			NativeToJsMessageQueue对象。因为所有的JS端与Android native代码			交互都是通过ExposedJsApi对象的exec方法。在exec方法中执行			PluginManager的exec方法，PluginManager去查找具体的Plugin并实例			化然后再执行Plugin的execute方法。由NativeToJsMessageQueue统一管			理返回给JS的消息。			Cordova在启动每个Activity的时候都会将配置文件中的所有plugin加载到			PluginManager。			当JS端通过JavascriptInterface接口的ExposedJsApi对象请求Android			时，PluginManager会从hashmap中查找到plugin，如果该plugin还未实例			化，利用java反射机制实例化该plugin，并执行plugin的execute方法。同			时Cordova中通过exec()函数请求android插件。Cordova在android端使用			了一个队列(NativeToJsMessageQueue)来专门管理返回给JS的数据。			在android平台的cordova.js中存在exec模块。该模块可以理解为js和			native交互的桥。该模块下存在androidExec(success, fail, 			service, action, args)方法，同时在该模块中存在jsToNativeModes。			默认采用addJacascriptInterface的方法，也就是jsToNativeModes下面			的JS_OBJECT。如果不存在window._cordovaNative对象，则采用prompt			方式替代。即js端执行prompt方法，prompt方法中携带native端方法执行所			需要的参数，native端拦截js端的prompt方法，主要是通过onJsPrompt方法			完成拦截并且执行native端的代码。
		* native到js的交互

			Native 调用 JS 执行方式有三种实现 loadUrl、 onLineEvent、			polling方式。			采用 webView.loadUrl(“javascript:”) 来执行js代码或者通过设置			webview的联网与掉线，从而触发js的window.ononline和			window.onoffline事件，然后再主动通过retrieveJsMessages到原生的消			息队列获取消息。
			
* iOS
Cordova作为可以让js和native通信的桥梁，提供了一系列的插件。Js可以利用这些插件调用native端的代码。
	* js到native的交互
		
	  js与native通信的关键源代码存在于cordova.ios.js下的iOSExec函数中。通过	  阅读iOSExec函数的源代码，可以看到js与objective-c的通信方式有两种。一种是	  通过原生的ajax请求，即XMLHttpRequest发起请求的方式。另一种是设置隐藏的   	  iframe的src属性。
	  
	  1. XMLHttpRequest发起请求的方式

	  	  在cordova的exec模块中，可以看到pokeNativeViaXhr函数和		  pokeNativeViaIframe函数。其中在pokeNativeViaXhr函数中可以看到如下		  代码段：
	  	  		  ```
		  execXhr.open('HEAD', "/!gap_exec?" + (+new Date()), true);
		  ```
		  		  该代码段表明请求的地址是/!gap_exec，同时根据：
		 
		  ```
  		execXhr.setRequestHeader('cmds',iOSExec.nativeFetchMessages());
		  ```

		 看到请求头设置请求数据。		 在native端则通过NSURLProtocol的子类拦截请求地址是/!gap_exec的请求。		 拦截之后分析ajax请求的数据，根据数据内容将请求分发到相应的cordova插件		 中，在通过cordova plugin这个桥梁，调用native的代码。这样就可以使用	    native的照相机、麦克风等原生功能。
		
	  2. 设置隐藏的iframe的src方式

	  	  cordova.exec往当前的html中插入一个隐藏的iframe元素，从而向		  UIWebView请求加载一个特殊的URL，这个URL里当然就包含了要调用的native 		  plugin的类名，方法名，参数，回调函数等信息。     	  
     	  接下来，由于被请求加载URL，于是UIWebViewDelegate的这个方法被调用：
	
		  ```
		  (BOOL)webView:(UIWebView*)theWebViewshouldStartLoadWithRequest
		  ```
		  
		  这里就进入了native侧，从request里就拿到了js端传过来的信息，然后调用到		  native plugin
		  
	* native和js的通信
	
	  UIWebView 有一个这样的方法					  stringByEvaluatingJavaScriptFromString:，这个方法可以让一个 	  UIWebView 对象执行一段 JS 代码，这样就可以达到 native 和js通信的效果，	  在 Cordova 的代码中多处用到了这个方法，其中最重要的两处如下：
	  
	  	* 获取js请求的数据
	  	* 把js请求的结果返回到js端
	  	
	  总的来讲，js和ios的native端通信可以理解为如下流程：
		  
	  ![Cordova ios interact with native](https://github.com/ToBeNumerOne/cordova/blob/master/ios.jpg)
	  
	  1. js端发起请求，执行以下方法：

	  	  ```
		  cordova.exec(successCallback, failCallback, service, action, actionArgs);
		  ```
		  
		  cordova为每个请求生成callbackId，这是每个请求的唯一标识符。这个标识符		  会随着请求传递到native端，最后native端在处理完请求之后，将此id和请求 		  结果一起发送到js端。
		  
	  2. js端在一个以callbackId为key的对象中，保存该请求的成功或者失败的回调函		  数。从而根据id找到回到函数，对返回的结果进行处理。
	  3. 每次js请求，发送到native端的数据有callbackId，service，action，		  actionArgs。
	  4. native端拿到上述参数，根据service找到对应的plugin。
	  5. 根据action找到该插件类中对应的处理方法，并且把actionArgs当做参数传递		  给相关处理方法。
	  6. 处理完成后，把处理结果以及相应请求的callbackId传递给js端。js端根据		  callbackId找到相应的回调方法，从而处理返回的结果。

## Cordova.js的运行流程
通过上文Cordova的源码解读部分以及针对源码各个模块的讲解，可以知道Cordova的基于事件的运行流程大致如下图所示:

![Cordova ](https://github.com/ToBeNumerOne/cordova/blob/master/cordovaevent.jpg)

为了安全的启动 ,bootstrap等待onNativeReady和onDOMContentLoaded完毕后才执行。Bootstrap方法首先是实例化和发布模块来给 Cordova用户使用。其次是广播 onCordovaReady事件来通知 Cordova层bootstrap完毕。发布是指根据id 实例化模块 , 然后把它作为某个 object 的属性。发布模块后会执行platform.js中的init方法。这个方法用于做 platform 初始化工作，将平台特性紧耦合，初始化js和native的交互。在 android 平台版本中 ,init方法包含了polling/xhr的初始化等工作。当这些工作完毕后，将会广播onCordovaReady事件。此后便可以安全的使用Cordova所提供的所有功能了。onCordovaReady后会去准备onDeviceReady事件触发的相关工作。

## Cordova插件
默认生成的Cordova项目没有使用任何插件，即默认生成的项目没有使用设备层面功能的功能。如果想要使用安卓系统等提供的功能，例如照相机、通讯录等功能，必须手动安装插件。一个Cordova插件暴露一个javascript的API，从而允许js通过该API调用native SDK的功能。Cordova提供了一些核心功能的插件。但是根据项目需求，可以下载第三方的插件或者自己开发插件，从而满足项目对于原生功能使用的需求。

## Hybrid的开发方式与其他开发方式的比较
时下流行的移动应用可分为三种：原生应用、Web app和混合型应用。

* 原生应用：通过各种应用市场安装，采用平台特定语言开发。
* Web app：通过浏览器访问，采用Web技术开发。
* 混合型应用：通过各种应用市场安装，但采用Web技术开发。它虽然看上去是一个原生应          用，但里面访问的实际上是一个Web应用。

![Cordova Compare](https://github.com/ToBeNumerOne/cordova/blob/master/compare.jpg)

混合型应用可以说是为了弥补上面两种应用开发模式的缺陷而生，它是两者混合的产物，并且尽可能继承了双方的优势：

* 首先，它可以让众多Web开发人员几乎零成本地转型成移动应用开发者。
* 其次，相同的代码只需针对不同平台进行编译就能实现在多平台的分发，大大提高了多平台开发的效率。而相较于Web应用，开发者可以通过包装好的接口调用大部分常用的系统API。

作为本文所讲的Cordova，Cordova正是Hybrid框架中的佼佼者，它基于标准的Web技术——HTML、JavaScript和CSS，用JavaScript包装平台的API供开发者调用，具备强大的编译工具来为不同平台生成应用。同时作为一个开源的项目，拥有丰富的第三方资源和产业链。虽然Cordova与原生相比性能以及用户体验方面存在些许不足，但是基于Cordova如下的有点，依旧有很多开发人员选择使用它。

* 一次开发，多出运行
* 多平台一致的用户体验
* 学习和开发成本低
* 更新方便
* 适合内容展示类的应用