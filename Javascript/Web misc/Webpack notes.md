[[#What is Babel?]]
[[#Webpack on a high level]]
[[#Describe old way of using js via script tag, and the two problems which led to birth of NPM and Webpack]]
[[#How NPM solves one of the problem?]]
[[#How Webpack solves the other problem?]]
[[#What is a Loader]]
[[#What is a Plugin]]


---

Typescript compiler compiles ts into js.

#### What is Babel?
Babel transpiles new generation JS which may yet not be supported by browser into older JS that's supported by the browser.

#### Webpack on a high level
Webpack is a file bundler. From a high level, webpack is used to minify and then bundle all your JS files together so your end users only need to download a single minified bundle.js file.

##### Describe old way of using js via script tag, and the two problems which led to birth of NPM and Webpack
Imagine without webpack, we would have a bunch of scripts. And then we have to insert those script tags into our html page. And they have to be imported in a carefully chosen order, because you have to make sure utility function is available before use.

For external libraries, e.g. lodash, we would have to download the `lodash.js` file and then include it in our script tag. See pic below.

That's a lot of script tags!
If the order is not correct, then code may break!
![[1676127300188.png|500]] 

So now we have two problems.
1. We have to download the deps manually, when we share we would also need to share all the external deps (solved with npm).
2. We have bunch of scripts, either downloaded from us or by npm. Their order in the html is important (solved with webpack).



##### How NPM solves one of the problem?
Npm helps us such that we don't need to download the external deps manually, we can use `npm install` command. It does two things, downloads the package into `node_modules` folder and secondly it updates `package.json` to include this dep as part of the project dependency. This is useful when sharing with others, we only need to share the `package.json` file and other devs can install the deps with `npm install`, no need to share `node_modules` which can get very large.


##### How Webpack solves the other problem?
It builds a dependency graph of all files (not just code, but other assets) and their deps, and minify/bundle all the files into one or more bundles, starting from the entry point.

Remember js in browser doesn't have access to file system. Loading modules is very tricky.

Now we have only 1 js file we need to include, all deps are bundled inside this file.
![[1676128445375.png|500]]



#### What is a Loader
Webpack at its core is only capable to bundle js/json files.

We use loaders to load static resources like stylesheets and images and output them as javascript modules before they get bundled. E.g, we have basic loaders like style-loader, css-loader, sass-loader etc. Like for js, webpack will also build the correct dependencies from your css and html files etc.

 By default, the html-loader and css-loader will inline your template/styles in the JS so you wind up with one big JS file and one request, all loaded in proper order.

#### What is a Plugin
Plugins influence the output and extend the capability of the webpack compiler. Eg. optimization, minifying , injection of env variables etc.



https://medium.com/the-node-js-collection/modern-javascript-explained-for-dinosaurs-f695e9747b70
