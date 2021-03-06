API
===

## maquette

The maquette object is the main object. It is either bound to window.maquette or it can be obtained using requirejs.

## maquette.h

```js
function h(selector, properties, children) // returns a VNode object
```

The `h` method is used to create a virtual DOM node. 
This function is largely inspired by the mercuryjs and mithril frameworks.
The h stands for (virtual) hyperscript.

### parameters

1. `selector` *string*  
Contains the tagName, id and fixed css classnames in CSS selector format. 
It is formatted as follows: `tagname.cssclass1.cssclass2#id`. 
2. `properties` *object*  
An object literal containing properties that will be place on the DOM node.
This object may contain plain attributes like `href`, event handlers like `onclick`, 
a `key` to make nodes distinguishable and a `classes` object literal containing a className properties with boolean values.  
Example: `h('div', {classes: {class1: true, class2: false} }, [])`
3. `children` *Array of VNode objects*  
An array of virtual DOM nodes to add as child nodes. 
This array may contain nested arrays and null or undefined values.
Nested arrays are flattened and null and undefined values will be skipped.

### returns

A [VNode](#vnode) object, used to render a real DOM later.
 
NOTE: Because maquette needs to be able to animate transitions and because it wants to be fast, you need to
follow these rules:

* All children should either have a unique selector, or have a unique `key` property. 
* The `properties` object must always contain the same set of properties in subsequent renderings. 
When properties are to be cleared, they must be set to either null or undefined.
* The `properties.classes` object must also always have the same properties in subsequent renderings.



## Maquette.createProjector

```js
function createProjector(element, renderFunction, options) // returns a Projector object
```

Starts a projector that executes the renderFunction and updates the DOM at the optimal moments. 
The renderFunction is always executed on requestAnimationFrame.
The renderFunction is scheduled to be executed on the first animation frame after:
* The projector was first started
* The [`projector.scheduleRender()`](#projectorchedulerender) function was called
* An event handler on a rendered virtual DOM node (onclick for example) was executed

### parameters

* `element` *HTMLElement*  
  The DOM node where the virtual DOM is rendered. See [mergeDom](#maquettemergedom) for details on how this is done.
* `renderFunction` *function*  
  The render function that takes zero arguments and returns a VNode.
* `options` *object*  
 Options that influence how the DOM is rendered and updated.

### returns

A [Projector](#projector) object containing a [`scheduleRender()`](#projectorschedulerender) method.

## Maquette.createCache

```js
function createCache() // returns a CalculationCache object
```

Creates a [CalculationCache](#calculationcache) object, useful for caching VNode trees.

### returns

A [CalculationCache](#calculationcache) object.



## Maquette.createDom

```js
function createDom(vnode, options) // returns a Projection object
```
The createDom method creates a real DOM tree given a [VNode](#vnode). The [Projection](#projection) object returned 
will contain the resulting DOM Node under the [`Projection.domNode`](#projectiondomnode) property.
This is a low-level method. Users wil typically use [`Maquette.createProjector`](#maquettecreateprojector) instead.

NOTE: VNode objects may only be rendered once.

### parameters

* `vnode` *VNode*  
  A virtual DOM tree that was created using the [`h()`](maquetteh) function.
* `options` *object*  
  Projection options

### returns

A [Projection](#rprojection) object.



## Maquette.mergeDom

```js
function mergeDom(element, vnode, options) // returns a Projection object
```

The mergeDom method creates a real DOM tree at an already existing DOM element given a [VNode](#vnode). 
The selector for the root VNode will be ignored, but its properties and children will be applied to the provided
element.
This is a low-level method. Users wil typically use Maquette.createProjector instead.

NOTE: [VNode](#vnode) objects may only be rendered once.

### parameters

* `element` *HTMLElement*  
  The element that is used as the root of the virtual DOM. It usually matches the selector of vnode, but this
  is not a hard requirement.
* `vnode` *VNode*  
  The root of the virtual DOM tree that was created using the [`h()`](#domsetterh) function.
* `options` *object*  
  Projection options

### returns

A [Projection](#projection) object.



## Projector

A projector is an object that reschedules the rendering and projection of the virtual DOM at the right moment.
It has the following properties:

## Projector.scheduleRender

```js
function scheduleRender()
```

Signals the projector that a render needs to take place at the next animation frame.

## Projector.destroy

```js
function destroy()
```

Makes sure that no more renderings take place. Note that calling `destroy()` is not mandatory. 
A projector is a passive object that will get garbage collected as usual if it is no longer in scope.


## Projection

A Projection object represents a VNode tree that has been converted to a real DOM tree. 
It provides the following properties:

## Projection.update

```js
function update(updatedVNode)
```

This function updates the rendered VNode to another VNode from a subsequent rendering.

## Projection.domNode

This property contains the root DOM Node that has been rendered.



## CalculationCache

A CalculationCache object remembers the previous outcome of a calculation along with the inputs.
On subsequent calls the previous outcome is returned if the inputs are identical.
This object can be used to bypass rendering of a virtual DOM subtree to speed up rendering and diffing of 
the virtual DOM.

## CalculationCache.result(inputs, calculation)

Returns the previous outcome of `CalculationCache.result` if the input array matches the previous input array.
Otherwise, the calculation function is executed and its result is cached and returned.
objects in the inputs array are compared using ===.

### parameters

* `inputs` *array*  
  Inputs is an array of objects that are to be compared using === with the previous invocation. They are
  assumed to be immutable primitive values.
* `calculation` *function*  
  Calculation is a function that takes zero arguments and returns an object (A VNode assumably) that can be cached.

## CalculationCache.invalidate()

Manually invalidates the cached outcome.

## VNode

A virtual representation of a DOM Node. This object is not meant to be used outside the maquette library. 
It is assumed to be immutable.
