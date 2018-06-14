---
layout: default
---

# Scala: Using Option is not optional

## Context

When writing a function, I always write the tests first. Why ?

1. I'm so lazy I'll never write them after implementing my function
2. Writing them after makes me test only the cases I thought of in my implementation
3. It pushes me to write a comprehensive specification for my function

Once I reminded myself these three reasons, I write how I want my function to behave.

Let's say I want to implement the `indexOf` function. It takes a `List[Int]` and an `Int` element and returns the index of the first occurrence of the element in the list.

```scala
def indexOf(element: Int, list: List[Int]): Int
```

I can easily write a few cases to demonstrate how I want it to work:

```scala
scala> indexOf(3, List(1, 2, 3, 4))
res0: Int = 2

scala> indexOf(4, List(1, 2, 3, 4))
res1: Int = 3

scala> indexOf(3, List(1, 2, 3, 3, 3, 4))
res2: Int = 2
```

These cases are the "good cases" where the list in not empty and where there is indeed an index to find because the element is actually in the list.

However, I said earlier that I wanted a "comprehensive specification" for my function. Therefore I have to write test cases for:

```scala
scala> indexOf(3, List())
res0: Int = ?

scala> indexOf(5, List(1, 2, 3, 4))
res1: Int = ?
```

I could have used a trick and return `-1` in these cases like the JavaScript `indexOf` method:

```javascript
> [1, 2, 3, 4].indexOf(5)
-1
```

Fortunately, I don't have to. Indeed, Scala provides a cool built-in feature that JavaScript does not have: the `Option` class !

_NB: In fact, the `indexOf` implementation from the `scala.collection.immutable.List` does not return an `Option[Int]` but an `Int` (with `-1` return value when element not in list, just like the Javascript implementation). Why ? To preserve Java interoperability and for performance reasons._

## What is an __Option__ ?

`Option` is a structure that represents something that may or may not exist, something optional. It can either hold a value or no value.

`Option` has two subclasses:
- the `Some` class used when the optional value exists
- the `None` object used when the optional value does not exist

Let's demonstrate how an `Option` works:
```scala
val someValue: Option[Int] = Some(23)
val noValue: Option[Int] = None
```

Then you can use the `Option` as such:
```scala
scala> someValue.get
res0: Int = 23

scala> noValue.get
java.util.NoSuchElementException: None.get
```
The `get` allows you to transform an `Option[Int]` into an `Int` but will throw an `Exception` if the `Option[Int]` holds `None`.

To get around this, the `getOrElse` method is extremely useful:
```scala
scala> someValue.getOrElse(0)
res1: Int = 23

scala> noValue.getOrElse(0)
res2: Int = 0
```
This method tries to unwrap the value but returns the default value provided (here `0`) if `None` is found.

There are two more idiomatic ways to handle `Option`:
- using `map`
- using pattern matching

```scala
scala> someValue.map(x => 2*x)
res0: Option[Int] = Some(46)

scala> noValue.map(x => 2*x)
res1: Option[Int] = None
```
The `map` method will allow you to effortlessly apply modifications on your optional value without unwrapping it explicitly nor handling the `None` case. If your `Option[Int]` was equal to `None`, the `map` output will be `None`, regardless of the input function provided.

For more explicit handling, you can use pattern matching:
```scala
someValue match {
  case Some(x) => println("There is a value: " + x)
  case None    => println("There is no value")
}
//will print "There is a value: 23"

noValue match {
  case Some(x) => println("There is a value: " + x)
  case None    => println("There is no value")
}
//will print "There is no value"
```

_NB: the `isEmpty` method is a shorter way to check if an `Option` contains a value._
```scala
scala> someValue.isEmpty
res0: Boolean = false

scala> noValue.isEmpty
res1: Boolean = true
```
Two others methods can be used to achieve the same test : 
- `isDefined` (will return `true` if the `Option` is an instance of `scala.Some`)
- `nonEmpty` (will return `false` if the `Option` is an instance of `None` )



If you need to check what's in an `Option` but only if it contains a value, you can use the `exists` method as follows:

```scala
def isEven(n: Int): Boolean = (n % 2 == 0) 

val someValue = Some(24)
val someOtherValue = Some(23)
val noValue = None

noValue.exists(isEven)          // false
someValue.exists(isEven)        // true
someOtherValue.exists(isEven)   // false
```

As shown above, it will evaluate the function provided on the unwrapped `Option` only if it contains something.
It can be shorter than explicitly unwrapping and applying the check.

## Back to the beginning

Now we know all about the use of `Option` we can refine our "comprehensive specification" of the `indexOf` function.
The `indexOf` method will no longer return an `Int` but an `Option[Int]`.
```scala
def indexOf(element: Int, list: List[Int]): Option[Int]
```

and because we need the function to have a meaningful return value when the element is not found in the list, the test cases become:

```scala
scala> indexOf(3, List(1, 2, 3, 4))
res0: Option[Int] = Some(2)

scala> indexOf(4, List(1, 2, 3, 4))
res1: Option[Int] = Some(3)

scala> indexOf(3, List(1, 2, 3, 3, 3, 4))
res2: Option[Int] = Some(2)

scala> indexOf(3, List())
res3: Option[Int] = None

scala> indexOf(5, List(1, 2, 3, 4))
res4: Option[Int] = None
```

## Implementations
Each one of the implementations below use the `zipWithIndex` method.
The `zipWithIndex` method returns a new list containing pairs consisting of all elements of the list paired with their index (see documentation [here](http://www.scala-lang.org/api/current/scala/collection/immutable/List.html)):
```scala
scala> List(23, 11, 18, 10).zipWithIndex
res0: List[(Int, Int)] = List((23,0), (11,1), (18,2), (10,3))
```



### Version 1
Using the `headOption` function (see documentation [here](http://www.scala-lang.org/api/current/scala/collection/immutable/List.html)) that returns an `Option` wrapping the first element of a list if it exists then using `map` (which preserve the optional nature as explained earlier).
```scala
def indexOf(element: Int, list: List[Int]): Option[Int] = {
  list.zipWithIndex
    .filter(_._1 == element)
    .headOption
    .map(_._2)
}
```

### Version 2
Using the `find` function (see documentation [here](http://www.scala-lang.org/api/current/scala/collection/immutable/List.html)) that already returns an `Option` then using `map`.
```scala
def indexOf(element: Int, list: List[Int]): Option[Int] = {
  list.zipWithIndex
    .find(_._1 == element)
    .map(_._2)
}
```

### Version 3
Using pattern matching to check for the element existence.
```scala
def indexOf(element: Int, list: List[Int]): Option[Int] = {
  val filteredElementsWithIndex = list.zipWithIndex
                                      .filter(_._1 == element)
  filteredElementsWithIndex match {
    case List() => None
    case head :: _ => Some(head._2)
  }
}
```

## Option type in others languages
> - In Agda, it is named `Maybe` with variants `nothing` and `just a`.
> - In C++17 it is defined as the template class `std::optional<T>`.
> - In C#, it is defined as `Nullable<T>` but is generally written as `T?`.
> - In Coq, it is defined as `Inductive option (A:Type) : Type := | Some : A -> option A | None : option A.`.
> - In Haskell, it is named `Maybe`, and defined as `data Maybe a = Nothing | Just a`.
> - In Idris, it is defined as `data Maybe a = Nothing | Just a`.
> - In Java, since version 8, it is defined as parameterized `final class Optional<T>`.
> - In Julia, it is named `Nullable{T}`.
> - In Kotlin, it is defined as `T?`.[1]
> - In OCaml, it is defined as `type 'a option = None | Some of 'a`.
> - In Perl 6, this is the default, but you can add a `:D` "smiley" to opt into a non option type.
> - In Rust, it is defined as `enum Option<T> { None, Some(T) }`.
> - In Standard ML, it is defined as `datatype 'a option = NONE | SOME of 'a`.
> - In Swift, it is defined as `enum Optional<T> { case none, some(T) }` but is generally written as `T?`.
>
> Source: [Wikipedia](https://en.wikipedia.org/wiki/Option_type)

[Go back](./)
