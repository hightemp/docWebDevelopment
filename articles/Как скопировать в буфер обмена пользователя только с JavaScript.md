# Как скопировать в буфер обмена пользователя только с JavaScript 

### I need to be honest, this isn’t really one API.

The clipboard API is made up of multiple API’s. But more than that, we’re going to need to pull in some more Web API’s that don’t directly relate to the clipboard and its utilities in order to completely take full advantage of the browser’s clipboard abilities. By the end you should have several new tools that make it easier to work with text on your websites and apps!

* * *

####  **Starting small: ClipboardEvent** 

The clipboard event is the easiest to use, and is technically created through the [real clipboard API](https://developer.mozilla.org/en-US/docs/Web/API/ClipboardEvent/ClipboardEvent) , but here’s the dilemma: the clipboard API isn’t exposed in most web browsers. So you can only get to it through event handlers on the `copy` , `cut` , and `paste` events.

This means you can modify what is happening during those events. Your code can change both what eventually ends up on the user’s clipboard, and what ends up showing on the screen. However, you can’t create your own event, [except in Firefox](http://caniuse.com/#search=clipboardevent) .

 **So.. What can we even do under these constraints?** 

As it turns out, plenty!

I can think of a few reasons to modify these events and I’ll give examples below.

*   Add or remove something from a copied or cut text.
*   Obtain the data that was pasted and use it elsewhere in your app.

That’s not necessarily a _lot_ , but with some extra API juice we can enhance our abilities. More on that later.

 **Lets take a look at some examples.** 

This won’t really stop people from copying your text, but it shows how it overrides the initial value.

In the above example all you have to do is add an eventListener to the copy event. The event that you receive contains `clipboardData` and its three methods: `setData` , `getData` , and `clearData` .

It actually has all methods and properties of the [DataTransfer](https://html.spec.whatwg.org/multipage/interaction.html#datatransfer) object. So you can inspect the properties, `items` and `types` in case you are not sure what arguments to use for `setData` or `getData` .

Don’t let the user paste into the confirmation input, or don’t let them paste their own password.

This example shows what it would look like to not allow a user to paste text into an input with the ID of “passwordConfirmation”. It also doesn’t allow them to paste “theUserPassword” into any field (we’re pretending that’s actually their password. Please ignore for a moment that you should never have the user’s password available in plain text in your web app.).

Another example could be a profanity filter if you are creating a web app that expects young users.

You could go on to use the value retrieved from `getData` elsewhere in your app. As an example, maybe you need to log when data is pasted, like in a test-taking app. Sometime’s pasting text in that setting could be OK, but you may want to be able to go back later and ensure the pasted content wasn’t copied from elsewhere.

* * *

### Creating your own copy event.

> Copying text to the clipboard shouldn’t be hard. It shouldn’t require dozens of steps to configure or hundreds of KBs to load. But most of all, it shouldn’t depend on Flash or any bloated framework. —  [clipboardjs.com](https://clipboardjs.com/ "https://clipboardjs.com/") 

Before we get into this, I’d like to introduce you to an open source project that simplifies everything I’m about to show you. It’s called “clipboard.js” and it allows you to easily copy text to your clipboard. You may find it easier than doing this on your own. It’s quite a small library, and has a very simple API.

But we’re here to learn, so lets look at how you can copy text to the user’s clipboard using only Web APIs.

Since the `ClipboardEvent` object isn’t exposed in most browsers, we must go a different route. To accomplish this we’ll need to add to our tools, `document.execCommand()` . This will allow us to run a `copy` command that copies that current selection to the keyboard.

Here’s an example shown with CodePen:

```html
<button onclick='copyText("Copied!")'>  Click me to copy!</button>
```

```javascript
// This must be triggered by a user event.
function copyText (text) {
  // Create the textarea input to hold our text.
  const element = document.createElement('textarea');
  element.value = text;
  // Add it to the document so that it can be focused.
  document.body.appendChild(element);
  // Focus on the element so that it can be copied.
  element.focus();
  element.setSelectionRange(0, element.value.length);
  // Execute the copy command.
  document.execCommand('copy');
  // Remove the element to keep the document clear.
  document.body.removeChild(element);
}// This must be triggered by a user event.
function copyText (text) {
  // Create the textarea input to hold our text.
  const element = document.createElement('textarea');
  element.value = text;
  // Add it to the document so that it can be focused.
  document.body.appendChild(element);
  // Focus on the element so that it can be copied.
  element.focus();
  element.setSelectionRange(0, element.value.length);
  // Execute the copy command.
  document.execCommand('copy');
  // Remove the element to keep the document clear.
  document.body.removeChild(element);
}
```

This must be triggered by a user event, like a click. This is a safety feature implemented on important operations, such as opening the file upload window, copying to a clipboard, etc. It helps to keep users safe from websites interacting with their computer when they don’t them want to.

A few notes:

*   You can use either an `input` with the type attribute set to text, or a `textarea` . With the latter requiring one less line of code in order to create it.
*   You may read that `document.execCommand` requires `designmode` , but as far as I can tell this is not true. It just needs to be triggered by a user initiated event.
*   You probably want to wrap `document.execCommand` in a try/catch block. In some browsers it will throw an error on failures.
*   You can inspect the result of `execCommand` to see if it worked or not. It returns a Boolean that relates to the success of the command.

And that’s it! You’re now successfully copying text to your clipboard!
