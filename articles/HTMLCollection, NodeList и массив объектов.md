# HTMLCollection, NodeList и массив объектов

Assuming the DOM is as described in the snippet below, my requirement is to get a javascript array of all the child nodes of `container` div.

```html
<div id="container">  
 <div class="divy">...</div>  
 <div class="divy">...</div>  
 <div class="divy">...</div>  
 <div class="divy">...</div>  
</div> 
```

During the good jQuery days, you could just do `$('.divy')` or `$('#container').children()` 

I was trying to do the same thing using the native DOM selector API and was in for a surprise

```javscript
const childDivs = document.querySelectorAll('.divy')

Array.isArray(childDivs) //=> false  
childDivs.constructor.name //=> NodeList
```

Okay let’s try one of the `getElementBy%` methods and check what is returned

```javscript
const childDivsAgain = document.getElementsByClassName('divy')

Array.isArray(childDivs) //=> false  
childDivs.constructor.name //=> HTMLCollection
```

Now what are `NodeList` and `HTMLCollection` objects and why are we not getting the plain vanilla javascript array from these methods?

Let’s try to understand the difference between HTMLCollection and NodeList first.

> An HTMLCollection is a list of nodes. An individual node may be accessed by either ordinal index or the node’s name or id attributes.

> Collections in the HTML DOM are assumed to be live meaning that they are automatically updated when the underlying document is changed.

HTML Collections are _always_ “in the DOM”, whereas a NodeList is a more generic construct that may or may not be in the DOM.

> A NodeList object is a collection of nodes. The NodeList interface provides the abstraction of an ordered collection of nodes, without defining or constraining how this collection is implemented. NodeList objects in the DOM are live or static based on the interface used to retrieve them

Let’s test the specification with relevant code to understand more

```javscript
let parentDiv = document.getElementById('container')

let nodeListDivs = document.querySelectorAll('.divy')  
let htmlCollectionDivs = document.getElementsByClassName('divy')

nodeListDivs.length //=> 4  
htmlCollectionDivs.length //=> 4

//append new child to container  
var newDiv = document.createElement('div');  
newDiv.className = 'divy'  
parentDiv.appendChild(newDiv)

nodeListDivs.length //=> 4  
htmlCollectionDivs.length //=> 5
```

We can see that the HTMLCollection is literally live, in the sense, any change to DOM is updated automatically and available in the collection.

Not all NodeList objects are static. For example, `document.getElementByName` will return a live NodeList.

But remember that neither `HTMLCollection` nor `NodeList` support the array prototype methods like `push`  `pop` or `splice` methods . Iterator methods like `forEach` have been recently added though and latest browsers might support them.

I came to know about this difference, when I was trying to use the [dragula drag-n-drop](https://github.com/bevacqua/react-dragula) library to add selected components to the library

```javscript
let draggableLists = document.querySelectorAll('div.draggable')  
dragula(draggableLists); //will not work
```

This will not work as expected as the `dragula()` constructor expects a javascript array and we are passing a NodeList to it.

To convert the NodeList or HTMLCollection object to a javascript array, you can do one of the following:

#### Use Array.from method

```javscript
const nodelist = document.querySelectorAll(‘.divy’)  
const divyArray = Array.from(nodelist)
```

#### Use Array.prototype.slice

```javscript
const nodelist = document.querySelectorAll(‘.divy’)  
const divyArray = Array.prototype.slice.call(nodelist)
```

#### ES6 way

And I like this one. If you are using ES6, you can just use the spread operator

```javscript
const divyArray = […document.querySelectorAll(‘.divy’)]
```

Now, let’s apply that to the dragula api to make it work

```javscript
dragula([...document.querySelectorAll('div.draggable')])
```

So when do you really convert NodeList to an array? Well, it depends on your usecase. If you really want an iterator to the latest updated DOM at all times, you should use the `NodeList` or `HTMLCollection` as it is without converting it to an array.

**********
[javascript](/tags/javascript.md)
