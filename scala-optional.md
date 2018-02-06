---
layout: default
---

# Scala: Options are not optional

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

## Back to the beginning

Now we know all about the use of `Option` we can refine our "comprehensive specification" of the `indexOf` function.
The `indexOf` method will no longer return an `Int` but an `Option[Int]`.
```scala
def indexOf(element: Int, list: List[Int]): Option[Int]
```

and the test cases become:

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

### Version 1
Using pattern matching (not on the `Option`).
```scala
def indexOf(element: Int, list: List[Int]): Option[Int] = {
  val filteredElementsWithIndex = list.zipWithIndex.filter(_._1 == element)
  filteredElementsWithIndex.length match {
    case 0 => None
    case _ => Some(f.head._2)
  }
}
```
### Version 2
Using the `find` function (see documentation [here](http://www.scala-lang.org/api/current/scala/collection/immutable/List.html)) that already returns an `Option` then using `map` (which preserve the optional nature as explained earlier).
```scala
def indexOf(element: Int, list: List[Int]): Option[Int] = {
  list.zipWithIndex.find(_._1 == element).map(_._2)
}
```

_NB: The `zipWithIndex` takes a list and zip each element with its index (see documentation [here](http://www.scala-lang.org/api/current/scala/collection/immutable/List.html)):_
```scala
scala> List(23, 11, 18, 10).zipWithIndex
res0: List[(Int, Int)] = List((23,0), (11,1), (18,2), (10,3))
```

[Go back](./)
