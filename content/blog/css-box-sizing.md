---
title: CSS box-sizing on mobile
date: "2024-11-16"
description: "CSS box-sizing to the rescue when your content unexpectedly scrolls horizontally"
---

When working on Tuist previews, I was checking out the website on mobile and noticed that the content was scrolling horizontally – but it took me a bit before I figured out why.

The screen ended up looking like this:

<img src="/img/css-box-sizing/preview-with-scrolling-indicator.png" width=300px alt="Screenshot of iPhone simulator with horizontal scrolling indicator at the bottom"></img>

There are two issues, both related:
- There is no padding between the rounded content with the preview metatada
- The content is scrolling horizontally.

The code that ends up stretching the content over the viewport looks roughly like this:
```html
<style>
    ...

    .preview__metadata {
        display: flex;
        flex-direction: column;
        padding: 16px;
        width: 100%;
    }
</style>

<div class="preview">
  <div class="preview__metadata">
    <h2 class="preview-metadata__title">Title</h2>
  </div>
</div>
```

Commenting out either `padding` or `width` would fix the content overflowing the viewport horizontally. But ... why? When the `preview__metadata` is set to `width: 100%`?

## CSS box-sizing

Turns out, my mental model of how the `width` in CSS is computed was not in line with the default. As described in the [docs](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing) for `box-sizing`, the default behavior is `content-box`: "If you set an element's width to 100 pixels, then the element's content box will be 100 pixels wide, and the width of any border or padding will be added to the final rendered width, making the element wider than 100px." In other words, CSS computes the width of the element based on the content and the padding and border are added as extra to the final width.

To get, in my opinion more intuitive behavior, you can set `box-sizing: border-box` which tells the browser to include the padding and border in the width of the element. Some folks actually make this property the default for all elements in their CSS resets, like [Josh Comeau](https://www.joshwcomeau.com/css/custom-css-reset/#one-box-sizing-model-2):
```css
*, *::before, *::after {
  box-sizing: border-box;
}
```

Sure enough, if you compare Josh's example to my code, it boils down to the same structure:
<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="vYoMxoy" data-pen-title="Untitled" data-user="fortmarek" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/fortmarek/pen/vYoMxoy">
  Untitled</a> by Marek Fořt (<a href="https://codepen.io/fortmarek">@fortmarek</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

If the same reset is added to my example (or directly to the `preview__metadata` class), our Tuist Preview page has padding with no horizontal overflow:

<img src="/img/css-box-sizing/preview-with-no-overflow.png" width=300px alt="Screenshot of iPhone simulator with padding between the content and the viewport"></img>
