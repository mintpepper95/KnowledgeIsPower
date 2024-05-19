#### Setup the project
1. Type of script needs to be module
2. We can use import syntax with mjs file, 
`import { simpleFetch } from './simple-fetch/index.mjs'`

1. Create a nodejs project with `npm i`
2. Paste simple-fetch into the root
3. Go to package.json, and create a start cmd under scripts section
 ![[Pasted image 20240519183638.png]]
4. Add `"type": "module"` to package.json to be able to use import statements. This will ensure all js and mjs files are interpreted as ES modules.
5. Install additional packages such as jest `npm install --save-dev jest` and localStorage, we use the flag `save-dev` because we don't need it for prod, only for testing under dev env.


##### Start parameter
"start" is a script defined under the scripts section, used to start an application. It's not a field by itself but a key within the scripts object.

##### Main parameter
"main" specifies the entry point file of a package/module, when others want to import our package.

![[Pasted image 20240519184017.png]]
Without a main parameter, we have to import our package be specifying the actual module. `require('my-npm-module/lib/module.js')`

However if we set main as `main": "lib/module.js`
Then we would only have to import our module like `require('my-npm-module')`

