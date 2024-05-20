install local stoage
`npm install localStorage`
install jest
`npm install --save-dev jest`


jest enable esm import
https://stackoverflow.com/questions/35756479/does-jest-support-es6-import-export


#### checking against empty object
Note `!obj` does not work with empty object, we have to use `Object.keys(obj).length === 0`


##### Writing a jest test for the API
```ts

import { FeatureFlagApi } from "./index";


describe("featureFlagApi", () => {

  let client;

  beforeEach(() => {

    client = new FeatureFlagApi();

  });

  
  test("Check Amazon true", async () => {

    const val = await client.checkFlagValue("isAmazonPay");

    expect(val).toBe("isAmazonPay: true");

  });

  
  test("Check Google false", async () => {

    const val = await client.checkFlagValue("isGooglePay");

    expect(val).toBe("isGooglePay: false");

  });

  
  test("Check new feature true", async () => {

    client.updateStore("hallo", true);

    const val = await client.checkFlagValue("hallo");

    expect(val).toBe("hallo: true");

  });

  
  test("Check throws error", async () => {

    await expect(() => client.checkFlagValue("boombastic")).rejects.toThrow(

      Error

    );

  });

});
```