## [轉載] Rain & Water Effect Experiments [Back](./../post.md)

> - Author: [Lucas Bebber](https://github.com/lbebber)
- Origin: https://tympanus.net/codrops/2016/05/03/animated-heat-distortion-effects-webgl/
- Time: May, 3rd, 2016

<br />

> Some experimental rain and water drop effects made with WebGL and shown in different demo scenarios.

<p aling="center">
    <img src="./RainEffects.jpg" />
</p>

Today we’d like to share some WebGL experiments with you. The idea is to create a very realistic looking rain effect and put it in different scenarios. In this article, we’ll give an overview of the general tricks and techniques used to make this effect.

```
Please note that the effect is highly experimental and might not work as expected in all browsers. Best viewed in Chrome.
```

### Getting Started

If we want to make an effect based on the real world, the first step is to dissect how it actually looks, so we can make it look convincing.
If you look up pictures of water drops on a window in detail (or, of course, observed them in real life already), you will notice that, due to refraction, the raindrops appear to turn the image behind them upside down.

<p aling="center">
    <img src="./755px-GGB_reflection_in_raindrops.jpg" />
</p>
<p aling="center">
    <i>Image credits: Wikipedia, <a href="https://en.wikipedia.org/wiki/File:GGB_reflection_in_raindrops.jpg" target="_blank">GGB reflection in raindrop</a></i>
</p>

You’ll also see that drops that are close to each other get merged – and if it gets past a certain size, it falls down, leaving a small trail.
To simulate this behavior, we’ll have to render a lot of drops, and update the refraction on them on every frame, and do all this with a decent frame rate, we’ll need a pretty good performance – so, to be able to use hardware accelerated graphics, **we’ll use WebGL**.

### WebGL

WebGL is a JavaScript API for rendering 2D and 3D graphics, allowing the use of the GPU for better performance. It is based on OpenGL ES, and the shaders aren’t written in JS at all, but rather in a language called GLSL.
All in all, that makes it look difficult to use if you’re coming from exclusively web development — it’s not only a new language, but new concepts as well — but once you grasp some key concepts it will become much easier.
In this article we will only show a basic example of how to use it; for a more in depth explanation, check out the excellent [WebGl Fundamentals](http://webglfundamentals.org/) page.
The first thing we need is a canvas element. WebGL renders on canvas, and it is a rendering context like the one we get with `canvas.getContext('2d')`.

```html
<canvas id="container" width="800" height="600"></canvas>
```

```js
var canvas = document.getElementById("container");
var gl = canvas.getContext("webgl");
```

Then we’ll need a program, which is comprised of a *vertex shader* and a *fragment shader*. Shaders are functions: a vertex shader will be run once per vertex, and the fragment shader is called once per pixel. Their jobs are to return coordinates and colors, respectively. **This is the heart of our WebGL application.**

First we’ll create our shaders. This is the vertex shader; we’ll make no changes on the vertices and will simply let the data pass through it:

```html
<script id="vert-shader" type="x-shader/x-vertex">
    // gets the current position
    attribute vec4 a_position;
    
    void main() {
        // returns the position
        gl_Position = a_position;
    }
</script>
```

And this is the fragment shader. This one sets the color of each pixel based on its coordinates.

```html
<script id="frag-shader" type="x-shader/x-fragment">
    precision mediump float;
    
    void main() {
        // current coordinates
        vec4 coord = gl_FragCoord;
        // sets the color
        gl_FragColor = vec4(coord.x/800.0,coord.y/600.0, 0.0, 1.0);
    }
</script>
```

Now we’ll link the shaders to the WebGL context:

```js
function createShader(gl,source,type){
    var shader = gl.createShader(type);
    source = document.getElementById(source).text;
    gl.shaderSource(shader, source);
    gl.compileShader(shader);
    
    return shader;
}

var vertexShader = createShader(gl, 'vert-shader', gl.VERTEX_SHADER);
var fragShader = createShader(gl, 'frag-shader', gl.FRAGMENT_SHADER);

var program = gl.createProgram();
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragShader);

gl.linkProgram(program);
gl.useProgram(program);
```

Then, we’ll have to create an object in which we will render our shader. Here we will just create a rectangle — specifically, two triangles.

```js
// create rectangle
var buffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
gl.bufferData(
    gl.ARRAY_BUFFER,
    new Float32Array([
        -1.0, -1.0,
         1.0, -1.0,
        -1.0,  1.0,
        -1.0,  1.0,
         1.0, -1.0,
         1.0,  1.0]),
    gl.STATIC_DRAW);

// vertex data
var positionLocation = gl.getAttribLocation(program, "a_position");
gl.enableVertexAttribArray(positionLocation);
gl.vertexAttribPointer(positionLocation, 2, gl.FLOAT, false, 0, 0);
```

Finally, we render the whole thing:

```js
gl.drawArrays(gl.TRIANGLES, 0, 6);
```

And this is the result:

<p align="center">
    <img src="./webgl-1.png" />
</p>

After that, you can play with the shaders to get a hang of how it works. You can see a lot of great shader examples on [ShaderToy](http://shadertoy.com/).

### Raindrops

Now let’s see how to make the raindrop effect. First, let’s see how a single raindrop looks:

<p align="center">
    <img src="./drop1.png" />
</p>

Now, there are a couple of things going on here.
The alpha channel looks like this because we’ll use a technique similar to the one in the [Creative Gooey Effects](http://tympanus.net/codrops/2015/03/10/creative-gooey-effects/) article to make the raindrops stick together.


