Webpack is a file bundler.

1. find the entry file and load its contents into memory
2. match certain text within the content and evaluate those (for e.g. @import)
3. find the dependencies based on previous evaluation and do the same with them
4. stitch them all into a bundle in memory
5. write the results to file system

When you examine the above steps closely, this resonates with what a Java compiler(or any compiler) does. There are differences of course but those don't matter to understand loaders and plugins.

---

Loaders:

are here because webpack promises to bundle together any file type.

Since webpack at its core is only capable enough to bundle js files, this promise meant that the webpack core team had to incorporate build flows which allowed external code to transform a particular file type in a way that webpack could consume.

These external code are called loaders and they typically run during step 1 and 3 above. Thus, since the stage at which these loaders need to run is obvious, they don't require hooks and neither do they influence the build process(since the build or bundle only happens at step 4).

So Loaders prepare the stage for compilation and they sort of extend the flexibility of the webpack compiler.

---

Plugins:

are here because even though webpack doesn't directly promise variable output, the world wants it and webpack does allow it.

Since webpack at its core is just a bundler and yet goes through several steps and sub-steps in doing so, these steps can be utilised to build in additional functionality.

The production build process(minifying and writing to file system), which is a native capability of webpack compiler, for e.g., can be treated as an extension of its core capability(which is just bundling) and can be treated like a native plugin. Had they not provided it, someone else would have done it.

Looking at the native plugin above, it appears as if the webpack bundling or compilation can be broken down into core bundling process, plus a lot of native plugin processes which we can turn off or customise or extend. This meant allowing external code to join in the bundling process at specific points that they can choose from (called hooks).

Plugins therefore influence the output and sort of extend the capability of webpack compiler.