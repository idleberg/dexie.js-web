---
layout: docs
title: 'Consuming Dexie as a module'
---
Dexie is written in ES6 and distributed in the UMD format. It can be consumed either as a plain script tag, required as a CJS, AMD or ES6 module.

Vanilla scripts are nice when testing out something. But a module-based approach is better in the long term and package manager helps you keep track of your dependencies. There are lots of combinations of package- and module systems to choose from. For web apps, `npm + webpack` works perfectly well so let's start with that alternative.

### NPM and webpack

With NPM you keep track of versions and dependencies for your app. It's also a perfect platform to use when using commonjs modules and webpack.

Assuming you've already installed nodejs that bundles `npm` with it. Start by initing your new npm package. CD to a brand new dir and do:

```
mkdir hello-dexie
cd hello-dexie
npm init
# Just press enter on all questions to generate your package.json
npm install dexie --save
npm install webpack -g
```

Write your javascript file (index.js or whatever) that uses dexie:

index.js
```javascript
var Dexie = require('dexie');
var db = new Dexie('hellodb');
db.version(1).stores({
    tasks: '++id,date,description,done'
});

// Don't be confused over Dexie.spawn() and yield here. It's not required for using Dexie,
// but it really simplifies the code. If you're a Promise Ninja, use vanilla promise
// style instead.
Dexie.spawn(function*() {
    var id = yield db.tasks.put({date: Date.now(), description: 'Test Dexie', done: 0});
    console.log("Got id " + id);
    // Now lets add a bunch of tasks
    yield db.tasks.bulkPut([
        {date: Date.now(), description: 'Test Dexie bulkPut()', done: 1},
        {date: Date.now(), description: 'Finish testing Dexie bulkPut()', done: 1}
    ]);
    // Ok, so let's query it
    
    var tasks = yield db.tasks.where('done').above(0).toArray();
    console.log("Completed tasks: " + JSON.stringify(tasks, 0, 2));

    // Ok, so let's complete the 'Test Dexie' task.
    yield db.tasks
        .where('description')
        .startsWithIgnoreCase('test dexi')
        .modify({done: 1});

    console.log ("All tasks should be completed now.");
    console.log ("Now let's delete all old tasks:");

    // And let's remove all old tasks:
    yield db.tasks
        .where('date')
        .below(Date.now())
        .delete();

    console.log ("Done.");

}).catch (err => {
    console.error ("Uh oh! " + err.stack);
});
```
*This script uses Dexie.spawn() and yield. You need a modern browser to open it. Note that Dexie does not require using the yield keyword, but it simplifies your code a lot if you do. Read more about this on [Simplify with yield](https://github.com/dfahlander/Dexie.js/wiki/Simplify-with-yield).*

Now, create a HTML page:

index.html
```html
<html>
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        <script type="text/javascript" src="bundle.js" charset="utf-8"></script>
    </body>
</html>
```

As you can see, the page just includes a file called bundle.js. That is the file that webpack will generate when doing the following command:

```
webpack ./index.js bundle.js
```

Now your done to open your web page in a browser. If you're on the Edge browser, you cant just open the page from your file system because it would block indexedDB. You could use the nice module `http-server` to start a local web server and access it from there.

```
npm install -g http-server
http-server .
```
Now start a browser towards [http://localhost:8080/](http://localhost:8080/) and press F12 to view the console log output.

Done.

### NPM and rollup

main.js:
```javascript
import Dexie from 'dexie';

var db = new Dexie('mydb');
db.version(1).stores({foo: 'id'});

db.foo.put({id: 1, bar: 'hello rollup'}).then(id => {
    return db.foo.get(id);
}).then (item => {
    alert ("Found: " + item.bar);
}).catch (err => {
    alert ("Error: " + (err.stack || err));
});
```

index.html
```html
<html>
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        <script type="text/javascript" src="bundle.js" charset="utf-8"></script>
    </body>
</html>
```

Shell:
```
npm install dexie --save
npm install rollup -g
rollup main.js -o bundle.js
```

The es6 version is located on https://npmcdn.com/dexie@latest/src/Dexie.js but rollup will read the jsnext:main attribute in package.json, so it's enough to just import 'dexie'.


### Bower

Dexie can also be installed via bower.

```
bower install dexie --save
```

It will show up in bower_components. You'll have to configure requirejs accordningly, see requirejs below.

### requirejs

requirejs doesn't find modules with tha magic that goes with npm and webpack. So you have to update your require config accordningly:

```javascript
require.config({
    paths: {
        "dexie": "node_modules/dexie/dist/dexie" // or the bower location, or simply https://npmcdn.com/dexie/dist/dexie
    }
});

// And to consume it:
requirejs(['dexie'], function (Dexie) {
    var db = new Dexie('dbname');
    ...
});
```

### systemjs

System.js is also not that magic as npm and webpack. You need to configure both its location and its module type. Here's how to do that:

```
npm install dexie --save
```

systemjs.config.js
```javascript

System.config({
    map: {
        'dexie': 'node_modules/dexie/dist/dexie.js'
    },
    packages: {
        'dexie': { format: 'amd' } // or 'cjs'
    }
});
```

### Typescript

With typescript and npm, it's super-easy. Just make sure to:

* Use npm to install dexie `npm install dexie --save`
* Make sure tsconfig has `{ moduleResolution: 'node' }`

In your code, import dexie and subclass it:

```typescript
// Import Dexie
import Dexie from 'dexie';

// Subclass it
class MyDatabase extends Dexie {
    contacts: Dexie.Table<IContact, number>;

    constructor () {
        super("myDb");
        this.version(1).stores({
            contacts: '++id,first,last'
        });
    }
}

interface IContact {
    id?: number,
    first: string,
    last: string
}

// Instantiate it
var db = new MyDatabase('myDb');

// Open it
db.open().catch(err => {
    console.error(`Open failed: ${err.stack}`);
});

```
That's it! Typings are delivered with the package. **DON'T**:use tsd or typings to add dexie's type definitions. They are bundled with the lib and pointed out via package.json's `typings` property.

See also [Dexie Typescript Tutorial](Typescript)

### Next steps

#### [Architectural Overview](https://github.com/dfahlander/Dexie.js/wiki/Design)

or

#### [Look at some samples](https://github.com/dfahlander/Dexie.js/wiki/Samples)

or

#### [Migrating existing DB to Dexie](Migrating-existing-DB-to-Dexie)

or

#### [Back to Tutorial](https://github.com/dfahlander/Dexie.js/wiki/Tutorial)