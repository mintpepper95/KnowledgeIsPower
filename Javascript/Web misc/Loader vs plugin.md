Typescript compiler compiles ts into js.

Babel transpiles JS into highly supported JS that's supported by the browser.


#### Webpack
Webpack is a file bundler. There are alternatives like vite.

Imagine without webpack, we would have a bunch of scripts. And then we have to insert those script tags to our html page. And they have to be imported in a carefully chosen order, because you have to make sure utility function is available before use.

That's a lot of script tags!
If the order is not correct, then code may break!
![[1676127300188.png|500]] 

It would be better if each file can tell us what other file it requires (its dependencies) and we can use that mapping. This is where Webpack comes in. It builds a dependency graph of all files (not just code, but other assets) and their deps, and bundle all the files into one or more bundles.

Now we have only 1 js file we need to include, all deps are bundled inside this file.
![[1676128445375.png|500]]


#### Loader
Webpack at its core is only capable to bundle js/json files.

We use loaders to parse static resources like stylesheets and images and output them as javascript modules before they get bundled. Eg, we have basic loaders like style-loader, css-loader, sass-loader etc.

#### Plugins
Plugins influence the output and extend the capability of webpack compiler. Eg. optimization, minifying , injection of env variables etc.