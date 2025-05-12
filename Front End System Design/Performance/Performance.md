Based on web.dev performance

[[#Explain what is the critical path and what steps are involved]]


---
When you enter a URL in browser's address bar, browser sends a GET request to the server for data. The first request for a web page is for an HTML resource.

#### Compression
Text based responses like HTML, js, css, SVG images should be compressed to reduce their transfer size over the network.

Mostly widely used compression algorithms are Brotli and gzip, with Brotli being 15% better than gzip.

Small resources, less than 1 KiB, don't compress well. Effectiveness of data compression depends on having a large amount of data.

However, we should still avoid large JS and CSS, they take significantly more time to parse and evaluate after browser has decompressed them.

#### CDN

It's often best to let a CDN to handle compression for you.
A CDN is a distributed network of servers that cache resources from your origin server, and in turn serves them from edge servers that are physically closer to your users.

#### Explain what is the critical path and what steps are involved

The critical path involves the following steps
* Browser begins parsing HTML document and converting it into a DOM.
* Simultaneously browser also parses CSS files refenced in the HTML to build a CSS Object Model.
* When browser encounters a script tag, it must halt DOM construction until script is fetched, parsed and executed (unless we use defer or async)
* Once Dom and CSSom are ready, they are combined to create the render tree.
* The browser calculates the position and size of each element in the render tree.
* The browser paints the content onto the screen.

![[Pasted image 20240610161356.png]]
Only after these steps have been completed, will the user see the initial content on screen.

Optimising critical rendering path means prioritise display of content to the first render.

![[Pasted image 20240610161553.png]]


#### What resources are on the critical render path?
From above we have learnt that browser needs to wait for some critical resources (html, render blocking css, render blocking js as they are all needed to construct the render tree) to download before it can complete the initial render.

Browser process HTML in a streaming fashion, as soon as browser gets any portion of a page's HTML, it will start to process it. It can then decide to render it/not before receiving rest of a page's HTML.

For the initial render, browser will not typically wait for all of HTML, fonts, images and non-render blocking js (eg. script placed at end of HTML).

Fonts and images are often regarded by browser as content that need to be filled during subsequent page rerenderss.

This can mean that areas of blank space left in the initial render while text is hidden waiting on fonts, or image is loading.

Page layout can shift as a result.

`<head>` element is key to processing the critical rendering path. Not all resources reffed in `<head>` are necessary for initial render, so browser only waits for those that are.

#### Render blocking resources
Some resources are deemed critical that browser pauses page rendering until it has dealt with them. CSS falls in here by default.

When a browser sees CSS, `inline` or `style` or `<link rel=stylesheet href='''>`. The browser avoids rendering anymore content until has it completed downloading and processing of CSS.

Just because a resource blocks rendering doesn't mean it stops browser from doing other things.

Browser can be efficient, it may be downloading a CSS resource and pause rendering, but will carry on processing and rest of HTML and also look for other work.

Render blocking resources like CSS, blocks rendering when they were discovered. So whether CSS is render blocking will depend on whether browser has found it.

This means for critical path, we are typically interested in render blocking resources in the head, as they block rendering.


#### Parser blocking resources
Resources that prevent browser from looking for other work by continuing parse the HTML. JS by default is parser-blocking (unless async || defer ), as JS can change DOM and CSsom upon its execution.

Parser-blocking resources are render blocking as well, since parser can't continue past a parsing resource until it's fully processed.

Any parser blocking resources in the head means all page content is blocked from being rendered.

Blocking the parser can have a huge performance cost - much more than just block rendering.

#### Optimising resource load
The browser block rendering until CSSom is constructed, to prevent flash of unstyled content.

A parser blocking resource interrupts HTML parser, such as script elements without async or defer. The browser needs to evaluate and execute script before proceeding to parse the rest of HTML. As the script may modify or access DOM during a time while it's still being constructed. Same thing when using inline js.

#### CSS Optimisation techniques

##### Minification
Bundlers like Webpack automatically does this for you in prod builds.

##### Remove unused CSS
The browser needs to download and parse all style sheets, that includes styles that are unused on the current page.

##### Avoid CSS @import declarations
```css
/* Don't do this: */@import url('style.css');


<!-- Do this instead: --><link rel="stylesheet" href="style.css">
```

CSS import declarations allows you to import an external CSS resource from within a stylesheet.
HTML `<link>` is part of HTML response, and is discovered much sooner than a CSS file downloaded by `@import` declaration.

##### In







