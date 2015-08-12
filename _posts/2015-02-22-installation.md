---
layout: post
title:  "Getting Started with Argon"
date:   2015-02-22 13:35:15
short_description: "This tutorial shows you how to get your Argon environment ready."
source_directory: tutorial1
---

An *argon.js* web application is created like any other web application; a server responds to http protocol requests and delivers an HTML5 web application to the browser.  In these examples, we will use static HTML and Javascript files, but any dynamic web application serve could be used. The files will include *argon.js* (as well other javascript libraries such as *three.js*, a 3D framework). The main or launch file (e.g. index.html) has the following structure: 

{% highlight html %}
<!doctype html>
</html>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, user-scalable=no, minimum-scale=1.0, maximum-scale=1.0">

<!--three.js, a 3D Scene graph for the web-->
<script src=“../../js/three.js"></script>

<!--threestrap.js is a bootstrapping library that makes three.js easier to work with.  Argon-three.js requires it-->
<script src=“../../js/threestrap.js"></script>

<!--The argon library support for integration of three.js and argon.j-->
<script src=“../../build/argon.js"></script>
<script src=“../../build/argon-three.js"></script>

<!-- One or more style sheets for styling the elements in the body -->
<link rel="stylesheet" type="text/css" href="style.css">

<body>
    <div id="argon-immersive-context">
         <!--any html for interface elements etc. that will appear on the screen in AR mode-->
    </div>
	<div>
		<!--one or more divs that you want to appear in "page mode" described below-->
	</div>
</body>

<!--application javascript code-->
<script src=“../app.js"></script>

</html>
{% endhighlight %}

In these examples, we follow a coding style that does not include any inline Javascript, but rather puts the main application code in an external javascript file included at the bottom of the page, after the body.

# The html page (index.html) 

In addition to offering AR features (such as geolocation, video of the surrounding word, 3D graphics and image tracking), the *Argon3 browser* is a standard web browser (e.g., on iOS, it uses Apple's web renderer) and can therefore render any web content available on the platform. You can take advantage of these capabilities in three ways.  

First, argon uses (or creates) a special div with the id ```argon-immersive-context```.  Anything you put in this div (or in divs nested inside this one) will be rendered on the screen in 2D, in front of the 3D AR context (*argon-three.js* creates a CSS 3D perspective div and/or WebGL canvas behind any other content in this div, into which the 3D AR content is rendered). You can use this feature to create interface elements such as display boxes or button to provide information and  interactivity for your application. You can see examples of this special argon div in tutorials 2 ([Geolocation](geolocation.html)) and 5 ([Periodic](periodic.html)). 

Second, you can put any additional divs into the body that you wish. These will be visible when the user puts the *Argon3 browser* into "Page Mode" by touching the toggle button in the upper right hand corner of the interface. *Argon3* will enable the "Page Mode" button when it sees non-AR content in the ```<body></body>``` of the document.  Initially, this non-AR content is hidden.  When the user activates Page Mode, the 3D content in that channel disappears (i.e., the content *argon-three.js* inserted into the argon-immersive-context), and the user sees the other HTML of the page (which is normally hidden). You can use this feature to add instructions,  text, images, or any other multimedia content to your *argon.js* experience. For example, in tutorials 2-6, we add text that explains the purpose of the example.

Third, because Webkit is a full-featured HTML5 engine (used by Safari), Argon can render most web pages (e.g., including those without any AR features).  Just type the url into the text box. 

## The application code (app.js)

In these tutorials, we place the application javascript in a separate file (you can create your web applications as you see fit, placing all content wherever you prefer, as with any web application). Segregating the javascript code into a file separate from  index.html file allows for a cleaner workflow as well as the use a wide variety of Javascript development tools to transcode, minify or otherwise manipulate your code. We name the file app.js in each of the tutorials.

Inside app.js, in order to initialize Argon, scripts typically begin with: 
{% highlight js linenos=table %}
var options = THREE.Bootstrap.createArgonOptions( Argon.immersiveContext )
options.renderer = { klass: THREE.CSS3DRenderer }
var three = THREE.Bootstrap( options )
{% endhighlight %}

Line 1 requests the default *immersiveContext* (the view from the device’s camera) as the surrounding visual environment that will be shown. (*argon.js* will eventually support having other **Contexts** in addition to the default *immersiveContext*, such as having embedded AR contexts in Page Mode. )

Line 2 indicates which renderer this web application will use. The *three.js* CSS renderer is used here. Alternatives include the WebGL renderer (```options.renderer = { klass: THREE.WebGLRenderer }```) or a combined WebGL/CSS3 renderer (```options.renderer = {klass: THREE.MultiRenderer }```)

Line 3  uses [threestrap](https://github.com/unconed/threestrap) to create a *three.js* scene graph along with the chosen renderer and store it as the variable ```three```. 

Together these three lines initialize the *argon.js* framework to use the device’s camera to display the world as an immersive background and create a scene graph to which the channel can attach 3D content.  You then add the javascript code that creates and manipulates that content.  For a first example of how this works, continue to  Tutorial 2 ([Geolocation](geolocation.html)). 