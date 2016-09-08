ModularTHREE [![NPM version][npm-image]][npm-url]
========

ModularTHREE simplifies the creation of [**THREE.js**](http://threejs.org/) based [**WebGL**](https://en.wikipedia.org/wiki/WebGL) scenes written in **ES2015**. In particular it handles:

* Basic scene setup with intelligent defaults
* Resizing your scene when the window is resized
* [Memoization](https://en.wikipedia.org/wiki/Memoization) of scene assets such as models and textures

**ModularTHREE** is designed to be used with build tools such as Rollup, Babel etc, however it should work fine if you use the old method of including ```<script>``` files in your ```<head>```.

To see it in action check out [**Modular THREE Boilerplate**](https://github.com/looeee/modular-three-boilerplate),
which includes a [Gulp](http://gulpjs.com/) setup for compiling ES2015 with
[Rollup](http://rollupjs.org/) and [Babel](https://babeljs.io/),
as well as compiling and [autoprefixing](https://github.com/postcss/autoprefixer)
[SCSS](http://sass-lang.com/) and piping through [Livereload](http://livereload.com/)
to be used with [Chrome](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei)
or [Firefox](https://addons.mozilla.org/en-US/firefox/addon/livereload/) livereload
plugins.


### NOTE: This is a very early build. Features are changing rapidly and this readme may be out of date. You probably shouldn't use this yet. ###

Installation
------

```sh
$ npm install --save modular-three
```
from your project root folder.

Requirements
------
####[**THREE.js**](http://threejs.org/)
```sh
$ npm install --save three
```
and include THREE in your build, or add ```<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r79/three.min.js"></script>``` to your ```<head>```.

Optional Addons
------

####[**GSAP**](http://greensock.com/gsap) for animation / tweening. ####

```sh
$ npm install --save gsap
```
and include GSAP in your build, or add ```<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/1.19.0/TweenMax.min.js">``` to your ```<head>```.

[**GSAP**](http://greensock.com/gsap) is a powerful animation library which allows
for the creation of much more complex animation than is possible using THREE alone.

See the animation section below for detailed instructions.

#### [**Stats**](https://github.com/mrdoob/stats.js/) ####
Show FPS and other stats on screen. If you haved installed THREE as an npm module you can include  ```three/examples/js/libs/stats.min``` in your build, otherwise add
```https://github.com/mrdoob/stats.js/blob/master/src/Stats.js``` to your ```<head>```.

Usage
------

The following instructions are heavily biased to using [Rollup](http://rollupjs.org/) and [Babel](https://babeljs.io/) as build tools. Rollup will require the ```rollup-plugin-node-resolve``` and ```rollup-plugin-commonjs``` plugins to allow importing of THREE, GSAP and other npm modules. See [**Modular THREE Boilerplate**](https://github.com/looeee/modular-three-boilerplate) for a ready to go build setup.

If your are using THREE as an npm module, include it in any files that use it:
```js
import THREE from 'three';
```

Then include **modularTHREE** at the start of any files that use it like so:
```js
import modularTHREE from 'modular-three';
```

Alternately if you want to just do these imports once, expose these variables globally:

```js
import THREE from 'three';
window.THREE = THREE;

import modularTHREE from 'modular-three';
window.modularTHREE = modularTHREE;
```

Important Note
------

THREE.js has recently (as of r80) switched to a modular build. This allows you to include it with an import statement as above.

#### Creating a drawing ####

The first step in using **modularTHREE** is to create a **Drawing**, which has a
```THREE.Scene```, ```THREE.Camera``` and ```THREE.Renderer``` associated with
a unique ```<canvas>``` element.

Create a class that extends ```modularTHREE.Drawing```:

```js
class ExampleDrawing extends modularTHREE.Drawing {
  constructor(rendererSpec, cameraSpec) {
    super(rendererSpec, cameraSpec);
  }

  init() {
    //Code for creating objects and adding them to the scene goes here
    //The drawing is reset (all objects removed) and init() is called again
    //on screen resize (throttled using lodash.throttle to once per 500ms),
    //or you can call ExampleDrawing.reset() to do this manually.
  }
}
```
 ```rendererSpec``` and ```cameraSpec``` are optional. If you just want to create a standard fullscreen drawing with a black background and [```THREE.PerspectiveCamera```](http://threejs.org/docs/?q=camera#Reference/Cameras/PerspectiveCamera), you can leave them out entirely.

Next instantiate your ```Drawing``` and call it's ```render()``` function.
```js
const exampleDrawing = new ExampleDrawing();
//If you are passing in a rendererSpec, cameraSpec do this instead:
//const exampleDrawing = new ExampleDrawing(rendererSpec, cameraSpec);

exampleDrawing.render();
}
```

That's it! With just a couple of lines of code you should now have a (blank)
scene.

To stop rendering, call ```exampleDrawing.cancelRender()```.

#### ```rendererSpec``` and ```cameraSpec```
For more control you can create ```rendererSpec``` and ```cameraSpec``` objects,
which have following options. If they are omitted the defaults shown will be used:

```js
const rendererSpec = {
  canvasID: '',
  antialias: true,
  alpha: true, //true required for multiple scenes
  autoClear: true, //false required for multiple scenes
  clearColor: 0x000000,
  clearAlpha: 0,
  width: () => window.innerWidth,
  height: () => window.innerHeight,
  pixelRatio: window.devicePixelRatio,
  postprocessing: false,
  useGSAP: false, //whether to use GSAP for animation
};
```

Note that ```alpha``` and ```autoClear``` **are** required if you are using multiple scenes.

```js
const cameraSpec = {
  type: 'PerspectiveCamera', //Or 'OrthographicCamera'
  near: 10,
  far: -10,
  position: new THREE.Vector3(0, 0, 100),
  //PerspectiveCamera only
  fov: 45, //PerspectiveCamera only
  aspect: () => window.innerWidth / window.innerHeight,
  // OrthographicCamera only
  width: () => window.innerWidth,
  height: () => window.innerHeight,
};
```

Note also that ```width```, ```height``` and ```aspect``` are passed in as functions.
This is because they are generally based on window dimensions, and this approach allows
them to be recalculated on window resize.

```rendererSpec``` and ```cameraSpec``` cover the minimal amount of options that **must** be set for every THREE scene. The ability to set all scene/camera/renderer options will be added in a future update.

#### Adding Objects to the Drawing ####
ModularTHREE currently provides the MeshObject class, which interfaces with the [THREE.LoadingManager](http://threejs.org/docs/?q=loading#Reference/Loaders/LoadingManager),
which can be used to provide a loading overlay, using for example  [**HeartcodeLoader**](https://github.com/heartcode/CanvasLoader),
or you own custom loader.

MeshObjects can be created like so:

```js
class Cube extends modularTHREE.MeshObject {
  constructor(spec) {
    super(spec);
  }

  init() {
    const geometry = new THREE.BoxGeometry(20, 20, 20);
    const material = new THREE.MeshBasicMaterial({ color: 0xFF0000 });
    this.mesh = new THREE.Mesh(geometry, material);
  }
}
```

Again the **spec** object is optional, but can be used to pass in variables such as material, dimensions etc.

Next, update the ```init()``` function of your ```Drawing``` class to instantiate the cube.

```js
class ExampleDrawing extends modularTHREE.Drawing {
  constructor(rendererSpec, cameraSpec) {
    super(rendererSpec, cameraSpec);
  }

  init() {
    this.cube = new Cube();
    this.scene.add(this.cube);
  }
}
```

And that's it! You should now have a small red cube in the middle of a black screen.

Let's make it a bit more interesting. We'll copy this example of a [**spinning wooden cube**]((http://threejs.org/examples/#webgl_geometry_cube)).

First, copy the texture file [**crate.jpg**](https://github.com/looeee/modular-three-boilerplate/blob/master/images/textures/crate.jpg) into your project root - say into ```<project-root>/images/textures/crate.jpg```.

Next update the ```Cube.init()``` function to use this texture:

```js
init() {
  const texture = this.loadTexture('images/textures/crate.jpg');
  const geometry = new THREE.BoxBufferGeometry(20, 20, 20);
  const material = new THREE.MeshBasicMaterial({
    map: texture,
  });
  this.mesh = new THREE.Mesh(geometry, material);
}
```

#### Adding Animation ####

Let's make the cube spin.

The standard method of adding animation with THREE is to use [```window.requestAnimationFrame()```](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame), and update the objects we want to animate each frame. See the [**source code for the spinning cube**](http://threejs.org/examples/webgl_geometry_cube.html) for an example.

To add a function which will be called every animationFrame to your drawing, call ```Drawing.addPerFrameFunction(yourFunction)```. Here is our testDrawing extended to make the cube rotate:


```js
class TestDrawing extends modularTHREE.Drawing {
  constructor() {
    super(rendererSpec, cameraSpec);
  }

  init() {
    this.initObjects();
    this.initAnimations();
  }

  initObjects() {
    this.cube = new Cube();
    this.scene.add(this.cube);
  }

  initAnimations() {
    const rotateCube = () => {
      this.cube.rotation.x += 0.005;
      this.cube.rotation.y += 0.01;
    };

    this.addPerFrameFunction(rotateCube);
  }
}
```

Your ```Drawing``` should now look pretty similar to the spinning cube example.

#### Using [**GSAP**](http://greensock.com/gsap) for Animation ####

The above is fine for simple animations, however things will get messy quickly if you are trying to do anything complex. To switch to using [**GSAP**](http://greensock.com/gsap) to handle animations, set ```rendererSpec.useGSAP = true```. If you have correctly included the GSAP script,
everything should be the same - the code will spin exactly as before, however you can now create GSAP timelines and tweens in your ```Drawing```.

A deep exploration of GSAP is beyond the scope of this Readme, however the GSAP [**documentation**](https://greensock.com/docs) is thorough and [**this is a good place to start**](https://greensock.com/get-started-js). But let's create a simple falling animation for our cube using ```Timeline``` and ```TweenMax```. Extend your ```TestDrawing``` function like so:

```js
initObjects() {
    this.cube = new Cube();

    //set the cube's initial position and rotation
    this.cube.rotation.set(-2, 2, 0);
    this.cube.position.set(0, 30, 0);

    this.scene.add(this.cube);
  }

  initAnimations() {
    const cubeTimeline = new TimelineMax();

    const cubeFallTween = TweenMax.to(this.cube.position, 3.5, {
      y: -20,
      ease: Bounce.easeOut,
    });

    const cubeRotateTween = TweenMax.to(this.cube.rotation, 3.5, {
      x: 0,
      y: 0,
      ease: Sine.easeInOut,
    });

    cubeTimeline.add(cubeFallTween);

    //add the rotation tween at time 0 so that falling and rotating happen
    //simultaneously
    cubeTimeline.add(cubeRotateTween, 0);
  }
```

#### Postprocessing ####

Forthcoming...

#### Using custom loaders with the THREE.LoadingManager ####

Forthcoming...


#### License MIT © Lewy Blue 2016 ####


[npm-image]: https://badge.fury.io/js/modular-three.svg
[npm-url]: https://npmjs.org/package/modular-three
