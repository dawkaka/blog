The HTML Canvas is basically a powerful drawing board for the web. The `<canvas>` element coupled with javascript allows developers to do stuff from basic drawing to advanced concepts like transformations and pixel manipulation.

In this article, we are going to take a deep dive into the HTML canvas element and I'll also share my experiences, tips and tricks I used while building a drawing app [draaaw](draaaw.vercel.app)

By the end of this crash course, you'll have a solid foundation in HTML5 Canvas, enabling you to create captivating web projects.

## The Canvas Element

The canvas element serves as a dynamic container that empowers you to utilize JavaScript for creating basic and intricate drawings within its space.

The markup is represented as follows:

```html
<canvas id="canvas" height="200px" width="200px"></canvas>
```

The `id` attribute is used to uniquely identify the canvas element within the document, enabling JavaScript interaction through `document.querySelector('#canvas')`.

Both `height` and `width` parameters define the dimensions of the rectangular canvas space, wherein shapes, text, images, and more can be drawn. For instance, if you attempt to draw a rectangle with dimensions surpassing the canvas' height or width, the protruding parts will be truncated.

## Creating Drawings on the Canvas Using JavaScript

Once you've inserted the canvas element, the next step is to actually start making drawings on it, and you can achieve this using JavaScript.

Here's a simple example of how this works:

```html
<canvas
  id="canvas"
  height="200px"
  width="200px"
  style="border: 2px solid black"
>
</canvas>
<script>
  const canvas = document.querySelector('#canvas');
  const ctx = canvas.getContext('2d');
  ctx.moveTo(0, 0);
  ctx.lineTo(200, 200);
  ctx.stroke();
</script>
```

Result:
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/c2toy8g6mtlo7xiuv5ou.png)

In aboveexample, the process begins by targeting the `canvas` using its assigned `id`, and then acquiring the `context` (referred to as `ctx`) using the `getContext` function.

#### Understanding getContext

`getContext` is used to determine the rendering context. It can either be `2d` for two-dimensional drawing or `3d` for three-dimensional drawing using [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API). For this tutorial, our focus is solely on `2d` drawing. The context specified determines the set of methods available on the `ctx` variable.
