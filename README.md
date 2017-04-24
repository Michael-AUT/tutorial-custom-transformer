# Implementing a custom Transformer

In this document we'll guide you through the steps necessary to implement a custom Transformer.
The transformer we'll implement will use [Babel](http://babeljs.io) to transpile ES2015/ES6 JavaScript to plain ES5 JavaScript that works in all Browsers.

## Overview

Custom transformers provide an easy way to extend the build functionality of *atscm*. Basically, a transformer implements two behaviours: How atvise server nodes are mapped to files (when running `atscm pull`) and vice versa (when running `atscm push`).

**Where to store transformers**

Basically, transformers can be stored anywhere inside your *atscm* project. When using a non-ES5 configuration language (such as ES2015 or TypeScript, chosen when running `atscm init`) transformers should also be written in this language. *atscm* will handle the transpilation of your transformer code automatically. If you plan to write multiple custom transformers for your project, it is recommended to create your transformers in an own directory, e.g `./atscm`.

## Step 0: Project setup

In order to have the same starting point, create a new *atscm* project to follow this tutorial. Run `atscm init` and **pick ES2015 as configuration language**.

Pull the empty project by running `atscm pull`. We'll use the default project files for testing later.

As suggested above, we'll store our custom transformer inside a new directory, `./atscm`. Create the directory, enter it and create an empty file called *BabelTransformer.js*:

```bash
mkdir atscm
cd atscm
echo "" > BabelTransformer.js
```

By now you should have a project containing an `./Atviseproject.babel.js` and an empty `./atscm/BabelTransformer.js` file.

## Step 1: Import *PartialTransformer* class

As we don't want to implement things twice we'll subclass *atscm*'s [Transformer class](https://doc.esdoc.org/github.com/atSCM/atscm/class/src/lib/transform/Transformer.js~Transformer.html). As our transformer shall only be used for JavaScript source files we can even use the [PartialTransformer class](https://doc.esdoc.org/github.com/atSCM/atscm/class/src/lib/transform/PartialTransformer.js~PartialTransformer.html) which supports filtering source files out of the box. As both of these classes are exported from *atscm*'s main file, importing them is pretty straightforward. Inside the *BabelTransformer.js* file add:
 
```javascript
// BabelTransformer.js

import { PartialTransformer } from 'atscm';
```

## Step 2: Create the *BabelTransformer* class

The next step is to create and export out Transformer class:

```javascript
// BabelTransformer.js

import { PartialTransformer } from 'atscm';

export default class BabelTransformer extends PartialTransformer {
  
}
```

We just created a *PartialTransformer* subclass that is exported as the file's default export. For more detailed information on ES2015's module system [take a look at the docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export). 

## Step 3: Use *BabelTransformer*

By default, *atscm* uses just some standard transformers. Any additional transformers must be configured to use inside the project's *Atviseproject* file.

First of all, we have to import our newly created *BabelTransformer* class:

```javascript
// Atviseproject.babel.js

import { Atviseproject } from 'atscm'
import BabelTransformer from './atscm/BabelTransformer';

export default class MyProject extends Atviseproject { ... }
```

Now we override the *Atviseproject.useTransformers* [getter](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Functions/get) to use our transformers:

```javascript
// Atviseproject.babel.js

...

export default class MyProject extends Atviseproject {
  ...
  
  static get useTransformers() {
    return super.useTransformers
      .concat(new BabelTransformer());
  }
  
}
```

This statement tells *atscm* to use a new *BabelTransformer* instance **in addition to the default transformers** (`super.useTransformers`).

To verify everything worked so far run `atscm config`. Our new Transformer should show up in the *useTransformers* section:

```
lukashechenberger:custom-transformer lukas$ atscm config
[10:44:55] Configuration at ~/Documents/Bachmann/atscm/tutorials/custom-transformer/Atviseproject.babel.js 
{ host: 'localhost',
  port: 
   { opc: 4840,
     http: 80 },
  useTransformers: 
   [ DisplayTransformer<>,
     ScriptTransformer<>,
     BabelTransformer<> ],
  nodes: 
...
```

## Step 4: Implement *PartialTransformer#shouldBeTransformed*

[PartialTransformer#shouldBeTransformed](https://doc.esdoc.org/github.com/atSCM/atscm/class/src/lib/transform/PartialTransformer.js~PartialTransformer.html#instance-method-shouldBeTransformed) is responsible for filtering the files we want to transform. Returning `true` means the piped file will be transformed, `false` bypasses the file.
 
In out case we want to edit all JavaScript source files. Therefore we return true for all files with the extension `.js`. Edit *BabelTransformer.js* accordingly:

```javascript
// BabelTransformer

...

export default class BabelTransformer extends PartialTransformer {

  shouldBeTransformed(file) {
    return file.extname === '.js';
  }

}
```
