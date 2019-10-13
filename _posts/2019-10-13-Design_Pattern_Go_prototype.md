---
title: Design Pattern -- Creational Prototype
---
# Prototype

The arm of prototype pattern is to have an object or set of objects that is 
already created at compilation time, but which you can clone as many times as
you want at runtime.


## Requirements and Accesstance criteria

* A shirt-cloner object and interface to ask for different types.
* When ask for shirt, a clone of shirt must be made, the new one must be different from original one.
* An info method must give all info of the instance fields.


## Code and UT

https://github.com/LipingMao/go-design-patterns/tree/master/creational/prototype
