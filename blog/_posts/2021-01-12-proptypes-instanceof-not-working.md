---
layout: post
title:  "PropTypes instanceOf not working"
date:   2021-01-12 22:22:47 +0100
categories: Javascript
---
**tldr;**
If you have one package validating a class instance and the class is created in another package, you can potentially find that the prop is not an instance of the class you are importing.
The Problem (example)

Class called Rectangle created in shapes package.
```js
export default class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}
```

create-shapes package installs shapes
```js
// package.json
“shapes”: “^1.0.1”  //installs 1.0.2
// CreateRectangle.jsx
import Rectangle from 'shapes';
const rectInstance = new Rectangle(12,14);
const CreateRectangle = ({ children }) => (<div>{React.cloneElement(children, { rectangle: rectInstance })}
</div>)
```

draw package installs shapes.
It consumes the shapes package to validate the prop rectangle is instanceof Rectangle class.

```js
// package.json
“shapes”: “^1.0.2”  //installs 1.0.2
// draw.jsx
import Rectangle from 'shapes';
...
rectangle: PropTypes.instanceOf(Rectangle), // this fails
```

Consumer App installs create-shapes and draw explicitly.
```js
//myapp.jsx
import CreateRectangle from 'create-shapes';
import Draw from 'draw';
...
() => (<CreateRectangle><Draw /></CreateRectangle>)
```

Importing only these two packages will result in installing same shapes package twice. The slight discrepancy in minimum version puts duplicate install of shapes in the following folders.
```js
1. myapp->node_modules->create-shapes->node_modules->shapes
2. myapp->node_modules->draw->node_modules->shapes
```

**The Solution**
To resolve this you must import “shapes”: “^1.0.1” explicitly in myapp package so there is only one shapes package being used for creating and validating the class instance.
myapp->node_modules->shapes

{% include like.html %}

