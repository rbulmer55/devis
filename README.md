# Devis

Devis is a framework for writing microservices and organizing the business logic of your app. You can break down your app into "stuff that happens", rather than focusing on data models or managing dependencies.

Devis provides:

- **pattern matching:** a wonderfully flexible way to handle business requirements

- **transport independence:** how messages get to the right server is not something you should have to worry about

Use this module to define commands that work by taking in some JSON, and, optionally, returning some JSON. The command to run is selected by pattern-matching on the the input JSON. There are built-in and optional sets of commands that help you build Minimum Viable Products: data storage, user management, distributed logic, caching, logging, etc. And you can define your own product by breaking it into a set of commands - "stuff that happens". That's pretty much it.

If you're using this module, and need help, you can post a [github issue][issue].

## Install

To install, simply use npm.

```
npm install devis
```

## Examples:

-**model.js**:

```javascript

let devis = require("devis");
let options = [];

devis.add({
    role: "model",
    action: "initialize"
}, function(args, done) {
    for (var attribute in args)
        options[attribute] = args[attribute];

    done(null, "initialization complete");
});

devis.add({
        role: "model",
        action: "GET"
    },
    GET);

devis.add({
    role: "model",
    action: "POST"
}, POST);

devis.add({
    role: "model",
    action: "PUT"
}, PUT);

devis.add({
    role: "model",
    action: "DELETE"
}, DELETE);

function DELETE(args, done) {
    let fin = false;
    let dataClass = options.dataClass;
    let EntityToRemove = ds[dataClass](args.ID)
    if (EntityToRemove) {
        EntityToRemove.remove();
        fin = true;
    }
    done(null, fin);
}

function PUT(args, done) {
    let fin = false;
    let dataClass = options.dataClass;

    let EntityToUpdate = ds[dataClass](args.ID) //args.ID={__KEY:10} or ={ID:10}
    if (EntityToUpdate) {
        try {
            for (var attribute in args.Update) {
                EntityToUpdate[attribute] = args.Update[attribute];
            }
            EntityToUpdate.save();
            fin = true;
        } catch (e) {
            fin = e;
        }
    }
    done(null, fin);
}

function POST(args, done) {
    let fin;
    let dataClass = options.dataClass;

    if (args.Add) { //add new Entity
        var newEntity = ds[dataClass].createEntity();

        try {

            for (var attribute in args.Add) {
                newEntity[attribute] = args.Add[attribute];
            }
            newEntity.save();
            fin = true;
        } catch (e) {
            fin = e;
        }
        done(null, fin);
    }
}

function GET(args, done) {
    let fin;
    let dataClass = options.dataClass;
    let func = args.func;
    if (args.data) {
        let searchData;

        fin = ds[dataClass][func](args.data);

    } else
        fin = ds[dataClass][func]();
    done(null, fin);
}


module.exports = devis;
```

-**index.js**:

```javascript
let devis=require("devis");
let data={firstName:"foo",lastName:"bar"};
devis.usePath('wakanda/model');
devis.act({role:"model",action:"initialize",dataClass:"Employee"},function(err,res){console.log(res)});
devis.act({role:"model",action:"POST",Add:data},function(err,res){console.log(res)});
devis.act({role:"model",action:"GET",func:"first"},function(err,res){console.log(res)});
```

In this code,

The `devis.add` method adds a new pattern, and the function to execute whenever that pattern occurs.

The `devis.act` method accepts an object, and runs the command, if any, that matches.

This is a _very convenient way of combining a pattern and parameter data_.

### Use network

Devis makes this really easy. Let's put configuration out on the network into its own process:

-**core.js**:

```javascript
var devis=require("devis");

devis.add({
  action: 'game',
  cmd:'play'
}, function(args, done) {

  done(null, { result: 'play' });
});
devis.add({
  action: 'game',
  cmd:'pause'
}, function(args, done) {

  done(null, { result: 'pause' });
});
```

-**server.js**:

```javascript
var devis=require("devis");
devis.use("./core")
.listen({
  host:'127.0.0.1',
  port:3030
})
```
![alt tag](https://lh6.googleusercontent.com/_f0JRA1Qp-psP3PcxjjzwwcywzyODjUo1hhEUKHmM81WAUNUHBpdq5fKIzqGIa83rmeudW0BDa-DOKE=w800)

The `listen` method starts a web server that listens for JSON messages. When these arrive, they are submitted to the local Devis instance, and executed as actions in the normal way. The result is then returned to the client. You can use Http,Tcp,Unix Socket or Named Pipes.

The client code looks like this:

-**Client.js**:

```javascript
var devis=require("devis")
.client({
  host:'127.0.0.1',
  port:3030,
  id:1
}).setName('client');

devis.act({ clientId:1,action: 'game', cmd: 'play' }, function (err, result) {
    console.log(result);
});

devis.act({clientId:1,action: 'game', cmd: 'pause' }, function (err2, result2) {
    console.log(result2);
});
```
![alt tag](https://lh3.googleusercontent.com/gzC7c0zD_BNiPtK4tXaBJDTKl1j-hgFIL3bs3wDcIh4BI8ajdtHCdTMP2FL3Vr4mqMGooCV1DjfobUM=w800)

On the client-side, calling `devis.client()` means that Devis will send any actions it cannot match locally out over the network. In this case, the configuration server will match the `action: 'game', cmd: 'play'` and `action: 'game', cmd: 'pause'` pattern and return the configuration data.



