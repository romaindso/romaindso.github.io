---
layout: post
title:  "Change the CSS of the input type=\"color\""
date:   2015-07-22 19:13:54
description: Change the CSS of the input type=\"color\"
categories:
- blog
permalink: change-the-css-of-the-input-type-color
---

If you don't know the `<input type="color"/>` let Mozilla introduce him to you : 
> The input type="color" represents a color well control, for setting the element's value to a string representing a simple color. A control for specifying a color. A color picker's UI has no required features other than accepting simple colors as text (more info). > - By [Mozilla Developper Network][MDN]

Sounds great, right ? 
<br/>=> 2 things :

1. [IE don't like colors][canisue], IE11 included and probably Edge (Ex Spartan)
2. The default render is not terrible

Example :

This code...
{% highlight html %}
<input type="color" value="#4285F4" />
{% endhighlight %}

...looks like this <input type="color" value="#4285F4" /> (You can try in different browser / OS)

Ok the look is not so bad but most important is that it's very hard to override.

##How to proceed ?

The idea is to hide the input color and display a `div` instead. This `div` must have a background-color identical with the value set in the input color. And obviously, interactions with the input color must remain possible in order to use the color picker.

Luckily, `opacity` is here. All we have to do is to set the input color opacity to 0 and with a bit of JS, we bind the background color of the div wrapper on the input color. When the value is changed, the `div` is updated.

{% highlight html %}
<!doctype html>
<html>
<head>
	<style>
		#wrapper {
		    height:25px;
		    width:25px;
		    display: inline-block;
		    border-radius:25px;
		}
		input[type="color"] {
		    opacity: 0;
		}
	</style>
</head>
<body>
    <div id="wrapper">
        <input id="colorPicker" type="color" value="#4285F4" />
    </div>
    <script>
        var colorPicker = document.getElementById("colorPicker");
        var wrapper = document.getElementById("wrapper");
        wrapper.style.backgroundColor = colorPicker.value;
        
        wrapper.addEventListener("change", function () {
            wrapper.style.backgroundColor = colorPicker.value;
        });
    </script>
</body>
</html>
{% endhighlight %}

<style>
	#wrapper {
	    height:25px;
	    width:25px;
	    display: inline-block;
	    border-radius:25px;
	}
	#colorPicker {
	    opacity: 0;
	}
</style>
<div>
    <span style="font-size: 1.125rem;">And this how it's looks like with a little border-radius :</span> 
    <div id="wrapper">
        <input id="colorPicker" type="color" value="#4285F4" />
    </div>
    <script>
        var colorPicker = document.getElementById("colorPicker");
        var wrapper = document.getElementById("wrapper");
        wrapper.style.backgroundColor = colorPicker.value;
        
        wrapper.addEventListener("change", function () {
            wrapper.style.backgroundColor = colorPicker.value;
        });
    </script>
</div>


Enjoy !

tags: [HTML5, input type="color", CSS]

[MDN]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/color
[canisue]: http://caniuse.com/#feat=input-color