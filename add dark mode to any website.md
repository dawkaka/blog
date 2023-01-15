We'll be going through how to easily add dark mode to any website regardless of the framework. The key here is using custom properties (css variables).

This is the website we'll be adding dark mode to

{% codepen https://codepen.io/dawkaka/pen/qByXaoz %}

You can see we have a white background and content with different shades of gray etc.

To add dark mode we simply use custom properties to hold each color instead of hard coding them in the css this way we just change the value of these custom properties based on the theme.
If you don't know what custom properties are you can read more about them [here](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)

This is what our css currently looks like

```css
body {
  background-color: white;
}
.contents {
  display: flex;
  justify-content: space-between;
  padding: 30px;
  margin-top: 30px;
  border-top: 3px solid #333;
}

.content-container {
  border-radius: 10px;
  width: 300px;
  padding: 10px 30px;
  flex-wrap: wrap;
  background-color: #eaeaea;
}

.content-container p {
  color: #333;
}
```

Now let's write some custom properties and assign the colors to them. We'll be applying those custom properties to the body tag so we can reuse it for any child node.

```css
body {
  /*light theme by default*/
  --bg-color: white;
  --color: black;
  --accent-1: #eaeaea;
  --accent-2: #333;
  background-color: var(--bg-color);
  color: var(--color);
}
.contents {
  display: flex;
  justify-content: space-between;
  padding: 30px;
  margin-top: 30px;
  border-top: 3px solid var(--accent-2);
}

.content-container {
  border-radius: 10px;
  width: 300px;
  padding: 10px 30px;
  flex-wrap: wrap;
  background-color: var(--accent-1);
}

.content-container p {
  color: var(--accent-2);
}
```

Now we've used custom properties to store the colors and it's exactly as before. Now to add dark mode we simply create a class `.dark` and reassign the value of those custom properties to reflect our dark theme colors. Due to the cascading nature of CSS, where styles are applied based on a priority system and classes are more specific (i.e higher priority) than HTML tags, if the body tag has the class `.dark` the custom properties of the class will be used and so to toggle dark mode all we need to do is set the class of the body tag to `.dark`

```css
.dark {
  /* dark theme custom variables*/
  --bg-color: black;
  --color: white;
  --accent-1: #111;
  --accent-2: #999;
}
```

Now when we add the `.dark` class to the `body` tag in the HTML, it'll switch to dark mode (yeah, just like that). But that's not how users change dark mode, so we need to add a button that when the user clicks we either add the `.dark` class or remove it for dark theme and light theme respectively.
Let's add a button and script to our HTML to handle this.
Add inside the body tag

```html
<button id="button" onclick="toggleTheme()">toggle theme</button>
<script>
  let body = document.getElementById('root');
  function toggleTheme() {
    const cls = body.className;
    if (cls === '') {
      body.className = 'dark';
      document.documentElement.style.colorScheme = 'dark';
    } else {
      body.className = '';
      document.documentElement.style.colorScheme = '';
    }
  }
</script>
```

setting the `document.documentElement.style.colorScheme` is necessary to make the scroll bar, <input />, <select>, etc also reflect the appropriate theme.

Here is the final results
{% codepen https://codepen.io/dawkaka/pen/JjByRLq %}

You should also save the preferred theme to local storage and set the appropriate theme whenever a page loads

##Conlusion
Everything is better with dark mode, add it to your website now. If you have any questions let me know in the comments.
