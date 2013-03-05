ProtoBuf.js - protobuf for JavaScript
=====================================
A protobuf implementation on top of [ByteBuffer.js](https://github.com/dcodeIO/ByteBuffer.js) including a .proto parser,
reflection, message class building and simple encoding and decoding in plain JavaScript.

Builder
-------
Probably the core component of ProtoBuf.js. Resolves all type references, performs all the necessary checks and returns
ready to use classes. Can be created from a .proto file or from a JSON definition. The later does not even require the
.proto parser to be included (see: `ProtoBuf.noparse.js`).

Example: [tests/complex.proto](https://github.com/dcodeIO/ProtoBuf.js/tree/master/tests/complex.proto)

Install: `npm install protobufjs`

```javascript
var ProtoBuf = require("protobufjs");

var builder = ProtoBuf.protoFromFile("tests/complex.proto");
var Game = builder.build("Game");
var Car = Game.Cars.Car;

// Construct with arguments list in field order:
var car = new Car("Rusty", new Car.Vendor("Iron Inc.", new Car.Vendor.Address("US")), Car.Speed.SUPERFAST);

// OR: Construct with values from an object, implicit message creation (address) and enum values as strings:
var car = new Car({
    "model": "Rusty",
    "vendor": {
        "name": "Iron Inc.",
        "address": {
            "country": "US"
        }
    },
    "speed": "SUPERFAST" // also equivalent to "speed": 2
});

// OR: It's also possible to mix all of this!

// Afterwards, just encode your message:
var buffer = car.encode();

// And send it over the wire:
var socket = ...;
socket.send(buffer.toArrayBuffer());

// OR: Short...
socket.send(car.toArrayBuffer());
```

Parser
------
Compliant with the protobuf parser to the following extend:

* Required, optional, repeated and packed repeated fields:

  ```protobuf
  message Test {
    required int32 a = 1;
    optional int32 b = 2 [default=100];
    repeated int32 c = 3;
    repeated int32 c = 4 [packed=true];
  }
  ```

* Data types: int32, uint32, sint32, bool, enum, string, bytes, messages, embedded messages, fixed32, sfixed32, float, double:
  
  ```protobuf
  message Test {
      required int32 a = 1; // Varint encoded
      required uint32 b = 2; // Varint encoded
      required sint32 c = 3; // Varint zigzag encoded
      required bool d = 4; // Varint encoded
    
      enum Priority {
          LOW = 1;
          MEDIUM = 2;
          HIGH = 3;
      }
    
      optional Priority e = 5 [default=MEDIUM]; // Varint encoded
      required string f = 6; // Varint length delimited
      required bytes g = 7; // Varint length delimited
      required Embedded h = 8; // Varint length delimited
    
      message Embedded {
          repeated int32 a = 1; // Multiple tags
          repeated int32 b = 2 [packed=true]; // One tag, length delimited
          required fixed32 c = 3; // Fixed 4 bytes
          required sfixed32 d = 4; // Fixed 4 bytes zigzag encoded
          required float e = 5; // Fixed 4 bytes
          required double f = 6; // Fixed 8 bytes
      }
  }
  ```
  
* Packages

  ```protobuf
  package My.Game;
  
  message Test {
      ...
  }
  
  message Test2 {
      required My.Game.Test test = 1;
  }
  ```
  
* Qualified and fully qualified name resolving:

  ```protobuf
  package My.Game;
  
  message Test {
      ...
      
      enum Priority {
          LOW = 1;
          MEDIUM = 2;
          HIGH = 3;
      }
  }
  
  message Test2 {
      required .My.Game.Test.Priority priority_fqn = 1 [default=LOW];
      required Test.Priority priority_qn = 2 [default=MEDIUM];
  }
  ```  

* Options on all levels:
  
  ```protobuf
  option toplevel_1 = 10;
  option toplevel_2 = "Hello!";
  
  message Test {
      option inmessage = "World!";
      optional int32 somenumber = 1 [default=123]; // Actually the only one used
  }

#### Not (yet) supported ####
* Extensions (what for?), imports (put everything into one builder instead) and services (you roll your own, don't you?).
  
However, if you need anything of the above, please drop me a note how you'd like to see it implemented. It's just that
I have no idea how to benefit from that and therefore I am not sure how to design it.

#### Calling the parser on your own ####
  
```javascript
var ProtoBuf = require("protobufjs"),
    fs = require("fs"),
    util = require("util");

var parser = new ProtoBuf.DotProto.Parser(fs.readFileSync("tests/complex.proto"));
var ast = parser.parse();
console.log(util.inspect(ast, false, null, true));
```

Encoder
-------
Built into all message classes. Just call `YourMessage#encode([buffer])`.

```javascript
...
var YourMessage = builder.build("YourMessage");
var myMessage = new YourMessage(...);
var byteBuffer = myMessage.encode();
var buffer = byteBuffer.toArrayBuffer();

// OR: Short...
var buffer = myMessage.toArrayBuffer();

var socket = ...; // E.g. a WebSocket
socket.send(buffer);
```

Decoder
-------
Built into all message classes. Just call `YourMessage.decode(buffer)`.

```javascript
...
var YourMessage = builder.build("YourMessage");
var buffer = ...; // E.g. a buffer received on a WebSocket
var myMessage = YourMessage.decode(buffer);
```

Downloads
---------
* [ZIP-Archive](https://github.com/dcodeIO/ProtoBuf.js/archive/master.zip)
* [Tarball](https://github.com/dcodeIO/ProtoBuf.js/tarball/master)

Documentation
-------------
* [View documentation](http://htmlpreview.github.com/?http://github.com/dcodeIO/ProtoBuf.js/master/docs/ProtoBuf.html)

Tests (& Examples)
------------------
* [View source](https://github.com/dcodeIO/ProtoBuf.js/blob/master/tests/suite.js)
* [View report](https://travis-ci.org/dcodeIO/ProtoBuf.js)

Features
--------
* [CommonJS](http://www.commonjs.org/) compatible
* [RequireJS](http://requirejs.org/)/AMD compatible
* Shim compatible (include the script, then use `var ProtoBuf = dcodeIO.ProtoBuf;`)
* [node.js](http://nodejs.org) compatible, also available via [npm](https://npmjs.org/package/protobufjs)
* [Closure Compiler](https://developers.google.com/closure/compiler/) ADVANCED_OPTIMIZATIONS compatible (fully annotated)
* Fully documented using [jsdoc3](https://github.com/jsdoc3/jsdoc)
* Well tested through [nodeunit](https://github.com/caolan/nodeunit)
* [ByteBuffer.js](https://github.com/dcodeIO/ByteBuffer.js) is the only production dependency
* Small footprint (even smaller if you use JSON definitions instead of .proto files, because no parser is required)

License
-------
Apache License, Version 2.0 - http://www.apache.org/licenses/LICENSE-2.0.html
