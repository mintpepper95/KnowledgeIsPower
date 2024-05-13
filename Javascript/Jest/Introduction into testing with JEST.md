
#### Installation and config

To install
`npm install --save-dev jest`


To enable ESM imports
Jest support ES6 and ESM as experimental features in 2024.

https://stackoverflow.com/questions/35756479/does-jest-support-es6-import-export
1.  Add 'jest': { 'transform': {} }  into package.json, prevents transforming ESM code to CommonJS

2. To be able to parse ES modules, use the experimental flag
```json
"scripts": {
    "test": "node --experimental-vm-modules node_modules/jest/bin/jest.js"
}
```


#### Testing
You can run test with `npm run test`

##### Testing sync functions


##### Testing async functions
