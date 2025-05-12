[[#Explain the role of IntersectionObserver]]


Beside user interactions, we often have components that are not visible on the initial page. Eg. Images that gets loaded when user scrolls down our app.

![[import_on_visibility.webm]]

As we are not requesting all images instantly, we can reduce the initial loading time!


#### Explain the role of IntersectionObserver

We can use `IntersectionObserver` API (allows us to know whether components are currently in our viewport) to quickly add import on visibility to our app.


```ts

// This uses react-loadable-visibility to import EmojiPicker on visibility, so react-loadable-visibility detects when EmojiPicker should be rendered and starts importing the module while the user sees a loading component.
const EmojiPicker = LoadableVisibility({
	loader: () => import("./EmojiPicker"),
	loading: <p id="loading">Loading</p>
});


const ChatInput = () => {
  const [pickerOpen, togglePicker] = React.useReducer(state => !state, false);

  return (
    <div className="chat-input-container">
      <input type="text" placeholder="Type a message..." />
      
      <Emoji onClick={togglePicker} />
      {pickerOpen && <EmojiPicker />}
    </div>
  );
};

export default ChatInput;
```

In this case, only when Emoji Picker is supposed to render on screen, we start importing the module.





