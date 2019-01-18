# Building Interfaces

## Learning Goals

- Define what an interface is in Object Oriented JavaScript
- Understand how interfaces help ensure weak coupling and help adhere to the
  single responsibility principle

## Introduction

So to quickly review: in Object Oriented JavaScript, we've looked at how
classes encapsulate specific related data and behaviors. That is, a class can
contain _properties_ and _methods_. We've practiced applying the single
responsibility principle, pushing us to convert a class with many
responsibilities into more classes, each with _one_ responsibility.

A responsibility might involve multiple properties or methods, but the key is
that everything contained within a single class should be aligned to a specific
purpose - often a class' responsibility may just be to serve up its own data.
Another class may be responsible for maintaining a relationship between two or
more other classes. Meanwhile, a separate class, then, may be responsible for
_working_ with that data, deriving something from it.

In Object Orientation, an interface provides a layer of abstraction, preventing
direct access and coupling to one or more _other_ classes. Using an interface
allows us to work with a class' properties without having to know the exact
implementation of those properties - all we actually need to be aware of are the
methods on the interface, not the implementation within the class our interface
depends on.

An interface allows for methods that are a bit more general, making them useful
when you have many _similar_ classes that have the same method names but differ
in implementation.

In this lesson, we're going to explore what interfaces are and how we would
implement them in an Object Oriented JavaScript program.

> Its important to note that the term 'interface' means widely different
> things depending on which OO language you're working with. Some languages like
> Java have a specific `interface` type that is distinctly separate from
> `class`.

> In JavaScript there is no distinct `interface` type in the language. How to
> implement interfaces in JavaScript, and _even whether or not we should_ use
> them, are topics still debated among programmers. There is some suggestion that
> future versions of JavaScript will include unique syntax for handling
> interfaces.

> For these lessons, however, we're going to build out interfaces using `class`
> syntax and focus on the reasons why interfaces are beneficial in Object
> Orientation.

## Understanding What an Interface Is

The key concept behind an interface is that we can use them to access and work
with classes without having to be concerned about the specific details of those
classes.

Imagine you and a friend are in school and you need to write a note. You ask
your friend 'Can you hand me something to write with?' You don't care what tool
your friend gives you as long as you can use it to write. They could hand you a
pen, a pencil, an ink quill... or they could also hand you a tablet, a laptop or
even a (small) typewriter. All of these objects fulfill the request.

Your friend _could_ hand you a voice recorder, but that doesn't fulfill the
exact request for something to _write_ with. Similarly, your friend can't _hand_
you a desktop computer, **although a desktop could still be used to write**.

It doesn't matter if you are given a pen, pencil, or laptop computer. They can
all be used, and they can all be handed to you, fulfilling your request.

If we were to write a set of classes to represent this, we might write something
like the following:

```js
// A pen has some unique properties along with a 'use()' method that logs a message
class Pen {
	constructor(inkColor = 'black') {
		this._inkColor = inkColor;
	}

	get inkColor() {
		return this._inkColor;
	}

	use() {
		console.log(`Writing with a ${this.inkColor} pen`);
	}
}

// A pencil also has its own unique properties.
// It also has a 'use()' method, but the implementation of 'use()' is different
class Pencil {
	constructor(status = 'sharp') {
		this._status = status;
	}

	set status(status) {
		this._status = status;
	}

	get status() {
		return this._status;
	}

	use() {
		console.log(`Writing with a pencil`);
		this._status = 'dull';
	}

	sharpen() {
		this._status = 'sharp';
	}
}

// The WritingUtensilInterface class is indifferent to whether we use a Pen or Pencil
// All that matters is that the utensil HAS a 'use()' method
class WritingUtensilInterface {
	set utensil(utensil) {
		this._utensil = utensil;
	}

	get utensil() {
		return this._utensil;
	}

	// this method checks to see if a utensil has a valid use() method
	// if it does, it calls the method
	write() {
		if (typeof this.utensil.use === 'function') {
			this.utensil.use();
		} else {
			throw 'Error: Writing utensil does not have a use() method';
		}
	}
}

let pen = new Pen();
let pencil = new Pencil();

let interface = new WritingUtensilInterface();
interface.utensil = pen;

interface.write();
// LOG: Writing with a black pen

interface.utensil = pencil;

interface.write();
// LOG: Writing with a pencil
```

An interface provides an way for a user to access properties and methods
without having to be directly dependent on the class that contains them.

An interface acts like a menu at a restaurant - it defines the options for what
you can order. The particular way a meal is prepared isn't relevant to the
customer and might change depending on the cook or available ingredients.

## Interfaces Define How a Class Should Be Used

While interfaces are an abstraction, they are also used to help define _how_ a
class' data and behaviors might be used.

Consider the following three classes:

```js
class Car {
	constructor() {
		this._engineStatus = 'off';
		this._wheels = 4;
	}

	get engineStatus() {
		return this._engineStatus;
	}

	set engineStatus(state) {
		this._engineStatus = state;
		if (state === 'on') {
			console.log('Vrooom!');
		}
	}
}

class Boat {
	constructor() {
		this._engineStatus = 'off';
	}

	get engineStatus() {
		return this._engineStatus;
	}

	set engineStatus(state) {
		this._engineStatus = state;
		if (state === 'on') {
			console.log('Gurgle gurgle gurgle!');
		}
	}
}

class Generator {
	constructor() {
		this._engineStatus = 'off';
	}

	get engineStatus() {
		return this._engineStatus;
	}

	set engineStatus(state) {
		this._engineStatus = state;
		if (state === 'on') {
			console.log('Nnnnnnnnnnnnnnnnnnnnnn!');
		}
	}
}
```

The above classes all contain similar properties and method names, though there
are variations in each class. All three have a property `_engineStatus` that is
set to `false` initially.

Imagine we were writing an application that needed to use all three classes -
say we wanted to be able to turn any of the engines 'on' or 'off', changing the
`_engineStatus` property. We _could_ write code that acts directly on each
class type:

```js
let car = new Car();
let boat = new Boat();
let generator = new Generator();

car.engineStatus = 'on';
// LOG: Vrooom!
boat.engineStatus = 'on';
// LOG: Gurgle gurgle gurgle!
generator.engineStatus = 'on';
// LOG: Nnnnnnnnnnnnnnnnnnnnnn!
```

There is a problem here, though. As the classes are currently written, there is
no stopping me from setting `_engineStatus` to whatever I want:

```js
car.engineStatus = 'overdrive';

boat.engineStatus = false;

generator.engineStatus = 66;
```

We _could_ add additional logic within our classes that only allows specified
inputs, but all three classes will need this added logic. Alternatively, though,
we could create an _interface_ to handle this:

```js
class EngineInterface {
	set engineBasedMachine(engineBasedMachine) {
		this._engineBasedMachine = engineBasedMachine;
	}

	get engineBasedMachine() {
		return this._engineBasedMachine;
	}

	turnOn() {
		if (typeof this.engineBasedMachine.engineStatus === 'function') {
			this.engineBasedMachine.engineStatus = 'on';
		} else {
			throw 'Error: Engine based machine does not include engineStatus property';
		}
	}

	turnOff() {
		if (typeof this.engineBasedMachine.engineStatus === 'function') {
			this.engineBasedMachine.engineStatus = 'off';
		} else {
			throw 'Error: Engine based machine does not include engineStatus property';
		}
	}
}
```

Here, we've defined the specific ways in which we can change `_engineStatus` by
providing two methods, `turnOn()` and `turnOff()`. These methods will only allow
us to assign `_engineStatus` to 'on' or 'off,' helping to prevent mis-assignment
of the `_engineStatus` property.

This _one_ interface will work with our `Car`, `Boat`, and
`Generator` classes:

```js
let engineInterface = new EngineInterface();

engineInterface.engineBasedMachine = car;
engineInterface.turnOn();
// LOG: Vrooom!

engineInterface.engineBasedMachine = boat;
engineInterface.turnOn();
// LOG: Gurgle gurgle gurgle!

engineInterface.engineBasedMachine = generator;
engineInterface.turnOn();
// LOG: Nnnnnnnnnnnnnnnnnnnnnn!
```

The `EngineInterface` class doesn't care about the specific implementation of
the `Car`, `Boat` and `Generator` `engineStatus` setter. Each one acts slightly
differently (each class logs a unique message in this case), but we're able to
interact with all of them the same way.

> Technically, in JavaScript, the `_engineStatus` property is still and always
> will be exposed externally, but what an interface does is _define_ how these
> class methods _should_ be used.

## Interfaces Encourage Weaker Coupling

In a full application, there may be many parts that rely on the `Car`, `Boat`
and `Generator` classes, but with an interface, we can avoid any direct coupling
to these classes. The interface itself can remain fairly simple and unchanging,
making it a better option to depend on. Only the interface needs to be dependent
on `Car`, `Boat` and `Generator`.

Because of this, it becomes easier to modify the `Car`, `Boat` and `Generator`
classes without having to worry about how those changes will affect other
classes. It doesn't matter if we rewrite the implementation of the
`engineStatus` setter, as long as there _is_ an `engineStatus` setter.

## Conclusion

An interface, at its core, is a layer of abstraction. An interface ensures that
the classes it is dependent on are used in the specific ways it defines.

When writing an Object Oriented application, there are times where we may have
distinct groups of classes that need to interact with each other. An interface
in between them helps maintain flexibility in the application by preventing
direct coupling between them - this has a beneficial affect of keeping our
classes clear of any bespoke code that is written specifically for a
relationship.
