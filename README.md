## Web Vitals guide
https://web.dev/vitals/

## First Contentful Paint (FCP) improving

FCP is the time after which the user will see at least some content of your site on his screen.
In most cases this content is text and images.

Therefore, to reduce the time to First Contentful Paint, we need some text or image to render faster.
It seems it is more logical to start with accelerating the rendering of texts, since they are already in the HTML-code: there is no need to make separate requests.

Most often browsers do not display text that is written in a custom font at all until the font file is loaded.

![](https://pbs.twimg.com/media/EwsFGSHWQAUAmFo?format=jpg&name=4096x4096)

The download chronology for most sites looks like this: the browser downloads and parses HTML, finds a link to a CSS file in it and loads it, finds a font-face block in the CSS code and downloads the font file.

So you can speed up the appearance of the text by shortening the chain of critical requests, "telling" the browser about the font at the stage of parsing the page itself. The font download will start earlier and the text will appear on the screen faster.

![](https://pbs.twimg.com/media/EwsJGSxXIAMni2c?format=jpg&name=large)

Fortunately, there is a font-display CSS property that can be used to tell the browser to display text in a different font (from the font-family chain) while the font file is downloading.

![](https://pbs.twimg.com/media/EwsJci2WUAMN3YT?format=jpg&name=large)

With font-display: swap enabled, texts will be rendered on the page immediately after loading the CSS code.

![](https://pbs.twimg.com/media/EwsJnU4XAAQ8aqb?format=jpg&name=4096x4096)

## Time to Interactive и Blocking Time improving

Open the "Performance" tab in DevTools, enable emulation of a slow CPU (you can also slow down the connection) and record the first seconds of page load.

![](https://pbs.twimg.com/media/EwTrXZSWEAE3HRM?format=jpg&name=large)

Expand the "Main" section and find the tasks marked with shading - they block the main thread the most and greatly interfere with the rendering of the page.

![](https://pbs.twimg.com/media/EwTr_p-WQAkMTIV?format=png&name=large)

In most cases, the call chains that you will see inside these tasks refer to blocks that do not appear in the first viewport at all.

Just make their code run only when the user scrolls to related elements. For example using the IntersectionObserver API

![](https://pbs.twimg.com/media/EwTwsrAXIAIYPem?format=jpg&name=large)

## Page loading and rendering

[How browser renders page](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction)

Connect scripts with async / defer, and check the DOMContentLoaded event in them

The user, entering the page, sees only some part of it. It is important to make sure that this visible part is displayed as quickly as possible. To do this, you can select the styles required for rendering the first screen and inline them. Webpack plugin:

[https://github.com/GoogleChromeLabs/critters](https://github.com/GoogleChromeLabs/critters)

Applying critical css will have a positive effect on metrics, for example, Largest contentful paint

Content metrics (Largest contentful paint / First meaningful paint) are also influenced by how fonts are connected and used.

A little more about fonts:
- the woff2 format is sufficient in most cases (https://caniuse.com/?search=woff2);
- connect the font with preload;
- use font-display: swap;

![](https://pbs.twimg.com/media/Ew7pPkaXMAMZqUf?format=jpg&name=large)

About images:
- specify width / height, so the browser will reserve space for pictures in advance
- for content images - loading = "lazy" attribute ([https://caniuse.com/loading-lazy-attr](https://caniuse.com/loading-lazy-attr)) for native scrolling loading (polyfill: [https://github.com/mfranzke/loading-attribute-polyfill](https://github.com/mfranzke/loading-attribute-polyfill))

## UX

Preloading:

For the visible part, create the skeleton (the thing as in the picture) and inline its styles in <style>.

![](https://pbs.twimg.com/media/Ew_QCCPWQAE0hon?format=jpg&name=small)

If you use a skeleton rather than a spinner, then it seems to users that the page loads faster:
[https://uxdesign.cc/what-you-should-know-about-skeleton-screens-a820c45a571a](https://uxdesign.cc/what-you-should-know-about-skeleton-screens-a820c45a571a)

![](https://pbs.twimg.com/media/Ew_YYDrW8AAC3si?format=jpg&name=900x900)

It is useful to study how users navigate the pages of your site and use this knowledge to preload resources. For example, from the main 80% go to registration, then you can download the resources in advance and render this page in the background.
[https://caniuse.com/link-rel-prerender](https://caniuse.com/link-rel-prerender)

![](https://pbs.twimg.com/media/ExAFwc4XAAASH_w?format=jpg&name=medium)

You need to work carefully with rel = "prerender", as it speeds up the next page for some users by loading additional resources on the current one for everyone. It is not recommended to make more than one link with rel = "prerender" per page.

Using rel = "preload" you can download and cache resources that will be needed soon. Unlike rel = "prerender", they will not be executed (only downloaded).
[https://caniuse.com/link-rel-preload](https://caniuse.com/link-rel-preload)

![](https://pbs.twimg.com/media/ExAIEbBWEAI_CEJ?format=jpg&name=medium)

Guess.js is an experimental library that, based on Google analytics data, predicts which page the user will go to next, and dynamically makes a prerender / preload.
[https://github.com/guess-js/guess](https://github.com/guess-js/guess)

### img

Content images
- alt="My photo"

Decoration images:
- background-image
- alt=""
- role="presentation"
- aria-hidden="true"

Svg images:
- role="img"

### Image compression 
[https://imageoptim.com/online](https://imageoptim.com/online)
[https://github.com/MeFoDy/image-processor](https://github.com/MeFoDy/image-processor)

## More

[https://habr.com/ru/company/yandex/blog/434130/](https://habr.com/ru/company/yandex/blog/434130/)
[https://twitter.com/harlan_zw/status/1397928487739027462](https://twitter.com/harlan_zw/status/1397928487739027462)
[https://web.dev/](https://web.dev/)

## DEFERRING STYLESHEETS 

<!-- Deferred stylesheet -->
<link rel="preload" as="style" href="path/to/stylesheet.css" onload="this.onload=null;this.rel='stylesheet'">

<!-- Fallback -->
<noscript>
  <link rel="stylesheet" href="path/to/stylesheet.css">
</noscript>

With link rel="preload" as="style" makes sure that the stylesheet file is requested asynchronously, while onload JavaScript handler makes sure that the file is loaded and processed by the browser after the HTML document has finished loading. Some cleanup is needed, so we need to set the onload to null to avoid this function running multiple times and causing unnecessary re-renders.

This is exactly how Smashing Magazine handles its stylesheets. Each template (homepage, article categories, article pages, etc.) has a template-specific critical CSS inlined inside HTML style tag in the head element, and a deferred main.css stylesheet which contains all non-critical styles.

## Web Vitals patterns
https://web.dev/patterns/web-vitals-patterns/
  
## Async decoding
  
Consider this example:

<p> some introductory text </p>
<img src = "very-big.jpg" />
<p> very important text for the user </p>
  
In this case, the user will have to wait until a large image is loaded to see the important information that follows it.

To avoid this situation, you can use the decoding co attribute with the value async. This will allow the browser to decode the image outside of the main thread, avoiding CPU overhead and not blocking further rendering of DOM elements. That is, the process of decoding the image will be postponed for the future, and the browser, in turn, will be able to render all the content without waiting for the image to load.
  
<p> some introductory text </p>
<img src = "very-big.jpg" decoding = "async" />
<p> very important text for the user </p>
