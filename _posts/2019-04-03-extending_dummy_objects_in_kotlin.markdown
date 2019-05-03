---
layout: post
title:  "Extending dummy objects in Kotlin"
date:   2019-04-03 11:12:01 +0000
#categories: kotlin software testing TDD test doubles
---

Often when testing software we want to construct an object which is dependent on other objects or functions, but we know that this code dependency is not a concern of our test. In these situations it is nice to use the simplest possible test double, classified by Martin Fowler as a [_dummy_](https://www.martinfowler.com/articles/mocksArentStubs.html). If this double is a class with member functions (also known as methods), none of these functions will have any behaviour. It might look something like this:

For a given interface of `FileStorage`
```kotlin
interface FileStorage {
    fun listFiles(): List<FileName>
    fun downloadFile(name: FileName): File
    fun storeFile(file: File)
}
```

a basic dummy might look like this

```kotlin
object DummyFileStorage : FileStorage {
    override fun listFiles(): List<FileName> = TODO("not implemented")
    override fun downloadFile(name: FileName) = TODO("not implemented")
    override fun storeFile(file: File) = TODO("not implemented")
}
```

In other test situations we might want other types of test double. These could be stubs, spies, fakes or mocks, depending on the style of testing we deem appropriate. It is quite common that we might need for example, a stub, but our test is only concerned with one member function. We can duplicate the dummy and give only this function the behaviour we desire, but this can lead to a lot of code duplication, especially if we have complicated objects (not that complicated objects are desirable, but I'm sure we've all been in this situation).
If we are testing a collaborator of `FileStorage` who calls the `listFiles` function, a basic stub for our `FileStorage` interface might look like this:

```kotlin
class StubFileStorage(fileNames: List<FileName>) : FileStorage {
    override fun listFiles(): List<FileName> = fileNames
    override fun downloadFile(name: FileName) = TODO("not implemented")
    override fun storeFile(file: File) = TODO("not implemented")
}
```

Kotlin provides us with a nice way to extend such dummy classes.
If we replace our dummy object with an _open class_

```kotlin
open class DummyFileStorage : FileStorage {
    override fun listFiles(): List<FileName> = TODO("not implemented")
    override fun downloadFile(name: FileName) = TODO("not implemented")
    override fun storeFile(file: File) = TODO("not implemented")
}
```

we can now extend the dummy, and only stub the method we care about:

```kotlin
class StubFileStorage(fileNames: List<FileName>) : DummyFileStorage() {
    override fun listFiles() = fileNames
}
```

If the usage is isolated to one particular area of our code, and we don't need any constructor parameters for our stub, we can even inline it as an anonymous object and assign it to a variable:
```kotlin
val stubFileStorage = object : DummyFileStorage() {
    override fun downloadFile(name: FileName) = File()
}
```

I'm not sure where this pattern originated (I think I was first shown it by [Dmitry Kandalov](https://twitter.com/dmitrykandalov) or [Pryio Aujla](https://github.com/PriyoAujla)) but I don't see it in use much and I think it's quite useful.


Links:
------
Test double definitions: [Mocks Aren't Stubs ](https://www.martinfowler.com/articles/mocksArentStubs.html)
