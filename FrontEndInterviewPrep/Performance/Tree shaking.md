It can happen we add code to our bundle but it's not used anywhere in the app, this piece of dead code can be eliminated to reduce size of the bundle. The process of eliminating dead code before it's added to the bundle is called tree shaking.

Here sum is not used, so should not be added to the bundle.

![[tree-shaking.webm]]

Modern apps that uses a bundler like Webpack comes with tree shaking automatically.

Consider your app and its deps as an abstract syntax tree - we want to shake the tree to optimise it.

In tree shaking, files are treated as a graph. Each node is a dep. Tree shaking starts from entry point and marks any traversed paths for inclusion.

Only modules defined with ES2015 (import/export) can be tree-shaken.

Tree shaking starts by visiting all parts of the entry point file with side effects, and proceeds to traverse the edges of the graph until new sections are reached. Once traversal is completed, JS bundle includes only the parts there were reached during the traversal.
