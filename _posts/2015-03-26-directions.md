---
layout: post
title:  "Directions"
date:   2015-03-26 13:35:15
short_description: "This channel demonstrates how to position content in space relative to the camera."
source_directory: tutorial4
---

Skybox Tutorial
--------------------
By default Argon will display whatever your phone's camera is capturing as a background, and for many apps this behavior is fine or even desirable. However, in some cases you might want to use a custom backdrop for an Argon app. This tutorial will demonstrate how Argon and Three.js can be used together to render a skybox panorama in the background of an Argon app.

Camera Setup
--------------------
To do almost any 3D rendering in Argon you will probably want to set up Three.js's camera to be bound to your phone's physical camera. This process was covered in the first tutorial, but in case you skipped that tutorial or need a refresher, here is code for binding the Three.js camera to Argon's camera information.

{% highlight js %}

var options = THREE.Bootstrap.createArgonOptions(ARGON.immersiveContext)
options.renderer = { klass: THREE.CSS3DRenderer }
var three = THREE.Bootstrap(options)

three.argon.bindComponent(new ARGON.Component.CameraTarget, three.camera)

var cameraLocationTarget = new ARGON.Component.CameraTarget
cameraLocationTarget.setFilter(ARGON.filters.onlyPosition)

var cameraLocation = new THREE.Object3D
three.argon.bindComponent(cameraLocationTarget, cameraLocation)
three.scene.add(cameraLocation)

{% endhighlight %}

Creating and Drawing the Panorama
--------------------
Once the camera is set up, rendering a skybox is fairly straightforward. There are multiple ways to render a skybox in Three.js, but this tutorial will only cover one simple method. Since a skybox is simply a cube with different textures on each face, an obvious way to create a Skybox is to use a [MeshFaceMaterial](http://threejs.org/docs/#Reference/Materials/MeshFaceMaterial), where each material corresponds to a different side of the skybox cube. The code to create a skybox using this method is shown below.

{% highlight js %}

//texture filenames for the panorama - depending on how the panorama was created, the ordering of each side may vary
//obviously the filenames here are placeholders 
var images = [
	"image_front.png",
	"image_back.png",
	"image_top.png",
	"image_bottom.png",
	"image_left.png",
	"image_right.png"
]

//this array will store each material
var materials = [];

for (var i = 0; i < 6; i++) {
	//load the image for a face's material
	var tex = new THREE.ImageUtils.loadTexture(images[i])
	tex.onLoad = function() {
		tex.needsUpdate = true
	}

	//notice the use of THREE.BackSide - since the camera is viewing the skybox from the inside of the cube,
	//the back of each face must be rendered rather than the front
	materials.push(new THREE.MeshBasicMaterial({color: 0xffffff, map: tex, side: THREE.BackSide}))
}

//create the skybox geometry and use the MeshFaceMaterial as its material
var skybox = new THREE.Mesh(new THREE.BoxGeometry(100, 100, 100), new THREE.MeshFaceMaterial(materials));

//add the cube to the scene relative to the cameraLocation Object3D so that it is guaranteed that the skybox will
//surround the camera even if the user moves around
cameraLocation.add(skybox)

{% endhighlight %}

One of the most intesting aspects of this code snippet is that the vast majority of the code is not specific to Argon. Loading the textures, creating the panorama material, and creating the actual skybox cube are all done with standard Three.js functionality. In fact, the only line that is Argon-specific is adding the skybox geometry to the Three.js scene, where the information about the user's physical location that Argon.js provides is used to ensure that the skybox will always surround the camera's position.

Using Argon Backgrounds
--------------------
When put together, all of the code shown so far will display a skybox panorama simply by creating a textured cube in 3D space around the user's current location. However, this simple approach has some drawbacks. The skybox created using this method has a fixed size, so any 3D objects placed outside of the cube will be clipped by the skybox and hidden, which is probably not desirable behavior. Making the skybox larger can temporarily fix this problem, but a much better solution would be to display the skybox in the background of the scene at all times, so that it can't possibly hide any other geometry. Fortunately Argon provides the functionality necessary to achieve this solution through the ARGON.Background object, which allows the user to create a customized background for any Argon app.

Creating a custom Argon background object takes a little bit more work than simply dropping a skybox cube into the Three scenegraph, but luckily most of the code shown so far can be reused. There are three components to an Argon background that need to be implemented to create a customized background: an initialization function to set up the background, an array of Javascript dependencies used by the background, and most importantly a rendering function to handle drawing and updating the background. Here is the code for the ARGON.Background object, with each of those three components being passed into its constructor (warning: it's fairly long, but hopefully some of it will look familiar).

{% highlight js %}

var panorama = new ARGON.Background({
	init: function() {
		var displayFrame = ARGON.ReferenceFrame.subscribe('display')
		displayFrame.on('pushState', function(state) {
			this.frame.pushState(state)
			this.update(state.timestamp)
		}.bind(this))
	},
	jsDeps:[
		'../../vendor/three.js',
		'../../vendor/threestrap.js'
	],
	renderScript: function(port) {
		var three = THREE.Bootstrap()

		var blankCanvas = document.createElement('canvas')
		blankCanvas.width = 256
		blankCanvas.height = 256

		var texture = new THREE.Texture(blankCanvas)
		texture.needsUpdate = true

		var materials = [];
		for (var i = 0; i < 6; i++) {
			materials.push(new THREE.MeshBasicMaterial({map: texture, side: THREE.BackSide}))
		}

		var skybox = new THREE.Mesh(new THREE.BoxGeometry(100, 100, 100),
				new THREE.MeshFaceMaterial(materials))

		three.scene.add(skybox)

		port.on('resize', function() {
			three.plugins.size.queue(null, three)
		})

		three.camera.matrixAutoUpdate = false

		port.on('update', function(e) {
			three.camera.fov = e.state.fov
			three.camera.updateProjectionMatrix()
			three.camera.matrix.fromArray(e.transform)
			three.camera.matrix.setPosition({x: 0, y: 0, z: 0})
			three.camera.matrixWorldNeedsUpdate = true
		})

		var options = null
		port.on('options', function(e) {
			options = e.options
			if (e.options.source.skybox) {
				_loadSkybox()
			}
		})

		THREE.ImageUtils.crossOrigin = 'anonymous';

		var _loadSkybox = function() {
			var images = options.source.skybox

			for (var i = 0; i < 6; i++) {
				var tex = new THREE.ImageUtils.loadTexture(images[i])
				tex.onLoad = function() {
					tex.needsUpdate = true
				}

				materials[i] = new THREE.MeshBasicMaterial({map: tex, side: THREE.BackSide})
			}
		}
	}
})

{% endhighlight %}

This code will give us an ARGON.Background object named panorama that is capable of displaying skybox panoramas. To understand how it does this, we need to examine the arguments passed into the constructor. First, the init function is given, which sets up the background to receive updates when Argon detects changes in its reference frame (such as a change in orientation or the user moving around with their phone). The jsDeps argument should be fairly self-explanatory: it is an array of locations to access resources the Background depends on. The most important argument is the renderScript function, since it is responsible for drawing and updating the background image. First the function sets up a blank texture to draw on the skybox if no images are supplied by the user and then uses that texture to setup an array of materials which is passed into the skybox mesh's constructor. This is all fairly similar to the code used to initialize the skybox earlier, except that we initially default to a blank texture since the actual panorama images are now passed into the object later. After this are three functions to update the state of the background in various situations: when the screen is resized, when Argon's state is updated, and when the options for the background are updated. The state update function is what keeps Three.js's camera aligned with Argon's information about the phone's physical camera, which is why this code does not need to use the specialized bootstrapping code we had to use to create a panorama without using an Argon Background. The function responding to updates to the options variable is very important, as it allows the actual skybox images are passed to the background by specifying an options.source.skybox variable, which will be an array of images for the skybox. The function called to load the skybox once the options variable has been updated should look very familiar, as it is creating textures for each of the six sides of the cube as shown in the earlier code snippet. Now that the code should make some sense, here is how to actually use the panorama object it creates.

{% highlight js %}

panorama.setOptions({source: {skybox: [
	ARGON.Util.resolveURL('image1.png'),
	ARGON.Util.resolveURL('image2.png'),
	ARGON.Util.resolveURL('image3.png'),
	ARGON.Util.resolveURL('image4.png'),
	ARGON.Util.resolveURL('image5.png'),
	ARGON.Util.resolveURL('image6.png')
]}})

ARGON.immersiveContext.requestBackground(panorama).catch(function(e) {
	console.log(e)
})

{% endhighlight %}

First this code sets the options for the panorama so that the Background object will load the images onto the skybox. Then, to actually display the background, simply call ARGON.immersiveContext.requestBackground with the Background object as the parameter (in this case the custom background we created earlier). The catch function added on to this function call will catch and log any errors that occur when switching backgrounds.