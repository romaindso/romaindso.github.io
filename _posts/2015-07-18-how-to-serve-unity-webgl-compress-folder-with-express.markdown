---
layout: post
title: "How to serve the \"Compressed\" folder with Express in an Unity WebGL release ?"
date:   2015-07-18 12:48:45
description: "How to serve the \"Compressed\" folder with Express in an Unity WebGL release ?"
categories:
- blog
- nodejs
- webgl
- performance
- express
- unity
---

Recently, Google killed the Unity web plugin (and many others) when they released Chrome 42 which block by default NPAPI support. For now, it's still possible to reactivate this support by enabling a special flag in Chrome (chrome://flags/#enable-npapi). 

But the final countdown for NPAPI continue :

> In September 2015 we will remove the override and NPAPI support will be permanently removed from Chrome. Installed extensions that require NPAPI plugins will no longer be able to load those plugins. > - By [Chromonium blog][blog-chromium]

Hopefully, a recent alternative exists aka **WebGL** and Unity works hard with it for continue to render 3D models in browsers without plugins. So if you work with Unity 5 or higher, you have now the possibility to build an WebGL project.

Happy end ? Not really. The main cons about WebGL's solution is the consequent weight to embed because all 3D processing is now performed by Javascript. Consequence, download times are really bad and your users have disappeared before the 3D models can be rendered.

###So how can we improve the performance ?

#####Unity side

There is some work to do on the 3D side before building the WebGL project.(optimize texture assets, set some options, reduce files of the build, ...). I'm not covering these points as you can check out the [Unity WebGL docs][unity-webgl] for more info on how to get the most out of WegGL with Unity.

#####Server side (Express)

Unity give us some help :

> If you make a release build, Unity will generate a “Compressed” folder next to your build output, which contains a gzip-compressed version of your build. If your web server is configured correctly, it will use these files instead of the much larger “Release” folder, as the http protocol natively supports gzip compression (which is faster then Unity could handle decompression by itself in JavaScript code). However, this means that you need to make sure that your http server actually supplies the compressed data. >

An important point is that it's not necessary to compress on the fly all Unity assets as they are already compressed by Unity. Moreover, to avoid troubles, don't include both **"Released"** and **"Compressed"** folders, only the latest is required.

This is the arborescence of my basic Express server :

	- app.js
	- dist
		- index.html
		- assets
	- public
		- js
		- css
		- vendors
			- unity
				- Compressed
				- TemplateData

`app.js` correspond to the Express server.

`dist` folder correspond to my client application. The `index.html` contains the Unity WebGL player with several urls pointing on the **"Released"** folder which is not included anywhere in the project.

Extract from `index.html` :

{% highlight html %}
<script src="/assets/vendors/unity/Release/UnityConfig.js"></script>
<script src="/assets/vendors/unity/Release/fileloader.js"></script>
<!-- (...) -->
<script>
script.src = "/assets/vendors/unity/Release/WebGL512.js";
// (...)
</script>
{% endhighlight %}

`public` folder contains all my assets. With a gulp task, these assets are moved into my client application in `dist` / `assets` folder when i build my project for production.   

To respect the Unity preconisations, when the server receives requests about files contents in “Release” folder, he must to serve the matching compresseed files placed into the **"Compressed"** folder.

#####How to proceed ?

1.Activate gzip on express using the compression middleware.
{% highlight javascript %}
var compression = require('compression');
//Compress all requests
app.use(compression());
{% endhighlight %}
2.Serve static files with express static. Normally you already have something similar.
{% highlight javascript %}
//Serve static files
app.use('/', express.static(path.resolve(__dirname , 'dist')));
{% endhighlight %}
3.Serve the compressed version of Unity files. **This is the most important point.**
{% highlight javascript %}
app.use('/assets/vendors/unity/Release/:filename', function(req, res){
    //Response used gzip encoding
    res.header('Content-Encoding', 'gzip');
    //Send file if there is a match into "Compressed" Unity folder
    res.sendFile(path.resolve(__dirname , '../public/vendors/unity/Compressed/'+req.params.filename+'gz'));
});
{% endhighlight %}
4.Add complentary headers for binary data
{% highlight javascript %}
app.use('/assets/vendors/unity/Release/:filename', function(req, res){
    var extensionFile = path.extname(req.params.filename);
    if(extensionFile === '.data' || extensionFile === '.mem'){
        res.header('Content-Type', 'application/octet-stream');
    }
    //(...)
});
{% endhighlight %}


Here is the complete code :
{% gist acfcdc015afe26b94d40 %}


[blog-chromium]: http://blog.chromium.org/2014/11/the-final-countdown-for-npapi.html
[unity-webgl]: http://docs.unity3d.com/Manual/webgl-building.html