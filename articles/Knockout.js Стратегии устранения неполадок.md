# Knockout.js Стратегии устранения неполадок
This post contains a number of tips, tricks, and strategies that I have used over the last couple of years to debug [Knockout](http://knockoutjs.com/) applications. Feel free to share some of your own tips in the comments or suggest other areas that you would like to see discussed.

## Binding issues

The most common errors encountered in Knockout typically center around incorrect or invalid bindings. Here are several different ways that you can diagnose and understand the context of your issue.

### The classic “pre” tag

One of the main debugging tasks is to determine what data is being used to bind against a certain element. The quick-and-dirty way that I have traditionally accompished this is by creating a “pre” tag that outputs the particular data context that is in question.

Knockout includes a helper function called `ko.toJSON` , which first creates a clean JavaScript object that has all observables/computeds turned into plain values and then uses [JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) to turn it into a JSON string. You would apply this to a “pre” tag like:

```html
<pre data-bind="text: ko.toJSON($data, null, 2)"></pre>
```

As of KO 2.1, the second and third arguments are passed through to `JSON.stringify` with the third argument controlling the indentation to produce nicely formatted output. Prior to KO 2.1, you could still do this by using `JSON.stringify(ko.toJS($data), null, 2)` . You can use this technique to look at specific properties or use the special [context variables](http://knockoutjs.com/documentation/binding-context.html) like `$data` , `$parent` , or `$root` . This will give you output like:

```json
{
  "title": "Some items",
  "items": [
    {
      "description": "one",
      "price": 0
    },
    {
      "description": "two",
      "price": 0
    },
    {
      "description": "three",
      "price": 0
    }
  ],
  "titleUpper": "SOME ITEMS"
}
```

### Lazy? Then just console.log it

Does that “pre” tag look like too much work? An even easier and more direct solution is to simply put a `console.log` in your binding string. The binding string is parsed into a JavaScript object, so code that you place in it will be executed. In addition, you can also put bindings that don’t exist into your binding string. This is because bindings can be used as options in other bindings (think `optionsText` or `optionsValue` ), so KO does not throw an exception when it finds an undefined binding handler. This means that you can simply do:

```html
<pre data-bind="text: ko.toJSON($data, null, 2)"></pre>
```

The block above will output your data to the console just before encountering a typo in the `description` property. You may even want to wrap your data in a call to `ko.toJS` or `ko.toJSON` depending on how you want the output to appear in the console.

 _Update: in KO 3.0, a binding with no actual handler, will no longer get executed. You would need to use an actual binding like the custom one described below or even something harmless like the built-in `uniqueName` binding._ 

### Extensions / bookmarklets

There are several great community-created solutions for doing this type of investigation as well:

*    [Knockout Context Debugger](https://chrome.google.com/webstore/detail/knockoutjs-context-debugg/oddcpmchholgcjgjdnfjmildmlielhof?hl=en) Chrome extension - adds a pane to the Chrome dev tools that allows you to see the context associated with a specific element. _Update: the latest version adds some nice tracing functionality as well._ Created by [Tim Stuyckens](https://twitter.com/timstuyckens) .
*    [Knockout Glimpse plugin](https://github.com/aaronpowell/glimpse-knockout) \- [Glimpse](http://getglimpse.com/) is a powerful debugging tool for ASP.NET. [Aaron Powell](http://www.aaron-powell.com/) created an excellent Knockout plugin for it.
*    [Bowtie](http://bowtie-ko.com/) \- a bookmarklet by Max Pollack that lets you inspect the context associated with bindings and add watch values. Note - it does assume that the page includes jQuery.
*    [Knockout-debug](https://github.com/jmeas/knockout-debug) \- a bookmarklet by James Smith that gives a nice display of your view model.

### Performance testing - how often is a binding firing?

Sometimes an interesting thing to explore/investigate is how often a certain binding is being triggered and with what values. I like to use a custom binding for this purpose.

```javascript
    ko.bindingHandlers.logger = {
        update: function(element, valueAccessor, allBindings) {
            //store a counter with this element
            var count = ko.utils.domData.get(element, "_ko_logger") || 0,
                data = ko.toJS(valueAccessor() || allBindings());

            ko.utils.domData.set(element, "_ko_logger", ++count);

            if (window.console && console.log) {
                console.log(count, element, data);
            }
        }
    };
```

This will show you a count of how many times the bindings on this element have been fired. You can either pass in a value to log or it will log the values provided to all bindings on that element. You may want to include a timestamp as well, depending on your needs. There are many times where this will open your eyes to a lot of unnecessary work happening in your UI. It may be from pushing [again and again](http://www.knockmeout.net/2012/04/knockoutjs-performance-gotcha.html) to an observableArray or maybe a case where [throttling](http://knockoutjs.com/documentation/throttle-extender.html) would be appropriate. For example, you might place this on an element like:

```html
<input data-bind="logger: description, value: description, enable: isEditable" />
```

and expect output like:

```
 1 <input…/> “Robert”
 2 <input…/> “Bob”
 3 <input…/> “Bobby”
 4 <input…/> “Rob”
```

Just a note that in the KO 3.0 (almost in beta), each binding on an element will be fired independently, as opposed to now where [all bindings fire together](http://www.knockmeout.net/2012/06/knockoutjs-performance-gotcha-3-all-bindings.html) . When KO 3.0 is released, you would want to use a binding like this with that in mind and likely only pass the specific dependencies that you are interested in logging.

### Undefined properties

Another common source of binding issues is undefined properties. Here is a quick tip for handling scenarios where a property is missing and it will not be added later. This is not specifically a debugging tip, but it can help you avoid potential binding issues without much fuss.

For example, this would cause an error (if `myMissingProperty` is missing):

```html
<span data-bind="text: myMissingProperty"></span>
```

However, this will bind properly against `undefined` :

```html
<span data-bind="text: $data.myMissingProperty"></span>
```

In JavaScript it is an error to attempt to retrieve the value of a variable that is not defined, but it is fine to access an undefined property off of another object. So, while `myMissingProperty` and `$data.myMissingProperty` are equivalent, you can avoid the “variable is not defined” errors by referencing the property off of its parent object ( `$data` ).

### Catching exceptions using a custom binding provider

Errors in bindings are inevitable in a Knockout application. Many of the techniques described in the first section can help you understand the context around your binding. There is another option that can also help you track and log problems in your bindings. A [custom binding provider](http://www.knockmeout.net/2011/09/ko-13-preview-part-2-custom-binding.html) provides a nice extensibility point for trapping these exceptions.

For example, a simple “wrapper” binding provider might look like:

```javascript
    (function() {
      var existing = ko.bindingProvider.instance;

        ko.bindingProvider.instance = {
            nodeHasBindings: existing.nodeHasBindings,
            getBindings: function(node, bindingContext) {
                var bindings;
                try {
                   bindings = existing.getBindings(node, bindingContext);
                }
                catch (ex) {
                   if (window.console && console.log) {
                       console.log("binding error", ex.message, node, bindingContext);
                   }
                }

                return bindings;
            }
        };

    })();
```

Now, binding exceptions will be caught and logged with the error message, the element, and the binding context (contains `$data` , `$parent` , etc.). A binding provider can also be a useful tool to log and diagnose the amount of times that bindings are being hit.

## View Model issues

Binding issues seem to be the most common source of errors in Knockout, but there are still errors/bugs on the view model side as well. Here are a few ways to instrument and control your view model code.

### Manul subscriptions for logging

A basic technique to help understand how things are being triggered in your view model is to create manual subscriptions to your observables or computeds to log the values. These subscriptions do not create dependencies of their own, so you can freely log individual values or the entire object. You can do this for observables, computeds, and observableArrays.

```javascript
    this.firstName = ko.observable();
    this.firstName.triggeredCount = 0;
    this.firstName.subscribe(function(newValue) {
        if (window.console && console.log) {
            console.log(++this.triggeredCount, "firstName triggered with new value", newValue);
        }
    });
```

You could format the message to include whatever information is relevant to you. It would also be easy enough to create an extension to add this type of tracking to any observable/computed:

```javascript
    ko.subscribable.fn.logIt = function(name) {
        this.triggeredCount = 0;
        this.subscribe(function(newValue) {
            if (window.console && console.log) {
                console.log(++this.triggeredCount, name + " triggered with new value", newValue);
            }
        }, this);

        return this;
    };
```

Since the extension returns `this` , you can chain it onto an observable/computed during creation like:

```javascript
this.firstName = ko.observable(first).logIt(this.username + " firstName");
```

In this case, I chose to include the `username` to ensure that it is outputting a unique value in the case that I have a collection of objects that each have a `firstName` observable. The output would look something like:

```
1 "bob1234 firstName triggered with new value" "Robert"
2 "bob1234 firstName triggered with new value" "Bob"
3 "bob1234 firstName triggered with new value" "Bobby"
4 "bob1234 firstName triggered with new value" "Rob"
```

### The value of “this”

Issues with the value of `this` is one of the most common challenges encountered when starting out with Knockout. Whenever you are binding to an event ( `click` / `event` binding) and are using a function off of another context (like `$parent` or `$root` ), you will generally need to worry about the context, as Knockout executes these handlers using the current data as the value of `this` .

There are several ways to ensure that you have the appropriate context:

1- you can use `.bind` to create a new function that is always bound to a specific context from within your view model. If the browser does not support [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) natively, then Knockout adds a shim for it.

```javascript
  this.removeItem = function(item) {
      this.items.remove(item);
  }.bind(this);
```

or if your function lives on the prototype, you would have to create a bound version on each instance (not ideal) like:

```javascript
this.removeItem = this.removeItem.bind(this);
```

2- you can create a variable in your view model that corresponds to the current instance ( `var self = this;` ) and always use `self` rather than `this` in your handlers. This technique is convenient, but does not lend itself to placing functions on a prototype object or adding functions through mix-ins, as these functions need to access the instance through the value of `this` (unless bound like in #1).

```javascript
 var self = this;

  this.removeItem = function(item) {
     self.items.remove(item);
  };
```

3- you can even use `.bind` in the binding string, depending on how comfortable you are with seeing it in your markup. This would look like:

```html
<button data-bind="click: $parent.removeItem.bind($parent)">Remove Item</button>
```

Similarly, you could create a simple custom binding that wraps the `event` binding and takes in a handler and a context like `data-bind="clickWithContext: { action: $parent.removeItem, context: $parent }"` .

4- the way that I personally handle this now is to use my [delegated events plugin](https://github.com/rniemeyer/knockout-delegatedEvents) , which will do event delegation and as a by-product is able to automatically call the method off of the object that owns it. This also makes it convenient/practical to place your functions on the prototype of your object. With this plugin, the above sample would simply look like:

```html
<button data-click="removeItem">Remove Item</button>
```

I have found that this removes the vast majority of the context binding concerns that I have in my viewmodel code.

## Using the console

One of my favorite ways to debug/investigate a Knockout app is to use the browser’s debugging console. If your view model is well written, you can generally control your entire application from the console. Recently, I ran a data conversion involving hundreds of records from the debugging console by loading the appropriate data, looping through each record to manipulate any observables that needed updating, and saving each record back to the database. This may not always be the best idea, but it was the most practical and efficient way for me to accomplish the conversion for my scenario.

### Accessing your data

The first step is getting a reference to your data from the console. If your app is exposed globally, perhaps under a specific namespace, then you are already good to go. However, even if you call `ko.applyBindings` without your view model being available globally, you can still easily get at your data. Knockout includes `ko.dataFor` and `ko.contextFor`  [helper methods](http://knockoutjs.com/documentation/unobtrusive-event-handling.html) that given an element tell you the data/context that was available to the element during binding. For example, to get your overall view model, you may start with something like this in the console:

```javascript
 var data = ko.dataFor(document.body);
```

Now you have access to your overall view model (if you simply called `ko.applyBindings` without specifying a specific root element). With access to that view model, you can call methods, set observable values, and log/inspect specific objects.

### Making logging a bit smarter

Anyone that has done debugging in Knockout has likely done a `console.log` on an `observable` or `computed` directly at some point and found some less than useful output like:

```javascript
function d(){if(0<arguments.length){if(!d.equalityComparer||!d.equalityComparer(c,arguments[0]))d.H(),c=arguments[0],d.G();return this}b.r.Wa(d);return c}
```

One way to improve this output is to add a `toString` function to the various types. I like to show the name of the type (like `observable` or `computed` ) and the current value. This helps you quickly understand that it is indeed a KO object, which type it is, and the latest value. This might look like:

```javascript
    ko.observable.fn.toString = function() {
        return "observable: " + ko.toJSON(this(), null, 2);
    };

    ko.computed.fn.toString = function() {
        return "computed: " + ko.toJSON(this(), null, 2);
    };
```

The output in the Chrome console would now look like:

```
observable: "Bob"
computed: "Bob Smith"
observable: "Jones"
computed: "Bob Jones"
```

Note that not all browsers respect the `toString` function, but it certainly helps with Chrome. Also, observableArrays are a bit harder to handle. Chrome sees that the observableArray (function) has both a `length` and a `splice` method and assumes that it is an array. Adding a `toString` function is not effective in this case, unless you delete the `splice` function from observableArrays.

## Final notes

*    **Debugging Knockout core** \- When you encounter exceptions in your Knockout application, I would not recommend trying to debug the Knockout core library code as your first choice. It is almost always more effective and efficient to isolate the origin of the issue in your application code first and try to simplify the reproduction of that issue. In my experience, the main situation where stepping into Knockout core is potentially a good choice is if you are truly seeing inconsistent behavior between browsers. Knockout core can be challenging to step through and follow, so if you do find yourself debugging into the core library, a good place to start is within the binding handlers themselves.
    
*    **Be careful with logging** \- if you add logging or instrumentation to your code, be mindful of how this logging may contribute to the dependencies of your computeds/bindings. For example, if you logged `ko.toJSON($root)` in a `computed` you would gain a dependency on all observables in the structure. Additionally, if you have a large view model, be aware of the potential performance impact of frequently converting large object graphs to JSON and logging or displaying the result. Throttling may help in that scenario.
    
*    **console.log format** \- the `console.log` calls in the snippets are just samples. I understand that some browers don’t support passing multiple arguments to `console.log` and some do not let you navigate object hierarchies. If you need to do this logging in a variety of browers, then make sure that you build suitable output. The normal case where I am doing logging is temporarily in Chrome during development.
    

 Posted by Ryan Niemeyer


**********
[KnockoutJS](/tags/KnockoutJS.md)
[javascript](/tags/javascript.md)
