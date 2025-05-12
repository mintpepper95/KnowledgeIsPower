`<link rel='preload'>` is a browser optimisation that allows critical resources ( that maybe discovered late ) to be requested earlier.

When optimising metrics like TTI, preload can be useful to load JS bundles that are necessary for interactivity. You want to use preload carefully to avoid the cost of delaying resource necessary for FCP ( first contentful paint) or LCP ( largest contentful paint ).

If you are trying to optimise loading of first party JS, you can also consider using `<script defer>` in the document `<head>` and `<body>` to help.


While prefetching is a great way to cache resources that maybe requested soon, we can preload resources that needs to be used instantly, like a certain font or image on the initial render.

Say `EmojiPicker` should be visible instantly on initial render. Although it should not be included in main bundle, it should get loaded in parallel. 

We can add a magic comment to let Webpack ( needs preload-webpack-plugin ) know if some module needs to be preloaded.

Unlike prefetch, where browser has a say whether it's good enough to load the resource or not, a preloaded resource will always get preloaded.

![[Pasted image 20240606125453.png]]


