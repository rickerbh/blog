---
layout: post
title: "Meteor, Hroku, and bcrypt"
date: 2016-06-29 11:07:11 +1000
comments: true
categories: 
---
Just had a very frustrating day. Was getting a deployment error on Heroku with a Meteor app I'm making.

```
Starting process with command `node build/bundle/main.js`
/app/build/bundle/programs/server/boot.js:324
}).run();
   ^
Error: Module did not self-register.
    at Module.require (module.js:365:17)
    at Module._compile (module.js:460:26)
    at Module.load (module.js:355:32)
    at Object.<anonymous> (/app/build/bundle/programs/server/npm/node_modules/meteor/npm-bcrypt/node_modules/bcrypt/bcrypt.js:3:35)
    at Error (native)
    at require (module.js:384:17)
    at Function.Module._load (module.js:310:12)
    at Object.Module._extensions..js (module.js:478:10)
    at bindings (/app/build/bundle/programs/server/npm/node_modules/meteor/npm-bcrypt/node_modules/bcrypt/node_modules/bindings/bindings.js:74:15)
    at Module.load (module.js:355:32)
Process exited with status 1
State changed from starting to crashed
```

I was using the build pack at [https://github.com/srbartlett/heroku-buildpack-meteor](https://github.com/srbartlett/heroku-buildpack-meteor) due to its support for Meteor 1.3.

The issue seems to be that the `npm-bcrypt` atmosphere package doesn't force a recompliation of the npm `bcrypt` package via `node-gyp` (I could be wrong here, but that's what it seems like). It seems that the atmosphere package includes the wrong (or a fixed?) architecture, where as what we actually need is to recompile for the current target. To fix this, I've altered the buildpack to remove the bundled `bcrypt` package, reinstall from source, and copy back to the bundled location. The reinstall from source seems to force `node-gyp` to compile for the correct architecture.

My altered buildpack is available at [https://github.com/rickerbh/heroku-buildpack-meteor](https://github.com/rickerbh/heroku-buildpack-meteor) if anyone wants it.