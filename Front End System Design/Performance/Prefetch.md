`<link rel='prefetch'>` is a browser optimisation that allows us to fetch resources that maybe needed for subsequent routes/pages before they are needed.

It can be done through declaring it in HTML, via HTTP header, service workers and webpack.

When we import modules based on visibility or interactions, there maybe some delay between clicking on a button to goggle the component, and showing the actual component. This is because module has to be requested and loaded.

In many cases, we know users will request certain resources soon after initial render. Although they may not be visible instantly, thus should not be in the initial bundle, it would be great to reduce the loading time regardless.

Components or resources we know that are likely to be used at some point in the app can be prefetched. We can let Webpack know certain bundles need prefetching. With a magic comment like below.

Modules that are prefetched are requested and loaded by the browser even before user requests the resource. When browser is idle and calulcates it has enough bandwith, it will make a request to get the resource and cache it.

Having resource cached will reduce loading time significantly.

Only prefetch necessary resources.

```ts
const EmojiPicker = import(/* webpackPrefetch: true */ "./EmojiPicker");
```

