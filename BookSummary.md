# 1. Introduction

## 1.1 Anatomy of a Type Class
```scala
// Define a very simple JSON AST
sealed trait Json
final case class JsObject(get: Map[String, Json]) extends Json
// etc...

// The "serialize to JSON" behaviour is encoded in this trait
trait JsonWriter[A] {
  def write(value: A): Json
}
```
`JsonWriter` is our type class in this example.

### 1.1.2 Type Class Instances
The instances of a type class provide implementations for the types we care about.
```scala
final case class Person(name: String, email: String)

object JsonWriterInstances {
  implicit val stringWriter: JsonWriter[String] =
    new JsonWriter[String] {
      def write(value: String): Json = JsString(value)
    }

  implicit val personWriter: JsonWriter[Person] =
    new JsonWriter[Person] {
      def write(value: Person): Json = JsObject(Map(
        "name" -> JsString(value.name),
        "email" -> JsString(value.email)
      ))
    }
}
```

### 1.1.3 Type Class Interfaces
A type class interface is any functionality we expose to users.
Interfaces are generic methods that accept instances of the type class as implicit parameters.

There are two common ways of specifying an interface: Interface Objects and Interface Syntax.

#### Interface objects
```scala
object Json {
  def toJson[A](value: A)(implicit w: JsonWriter[A]): Json =
    w.write(value)
}
```

To use this object, we import any type class instances we care about and call the relevant method:
```scala
import JsonWriterInstances._

Json.toJson(Person("Dave", "dave@example.com"))
// res4: Json = JsObject(Map(name -> JsString(Dave), email -> JsString(dave@example.com)))
```

#### Interface Syntax

We can alternatively use extension methods to extend existing types with interface methods.
Cats refers to this as “syntax” for the type class:
```scala
object JsonSyntax {
  implicit class JsonWriterOps[A](value: A) {
    def toJson(implicit w: JsonWriter[A]): Json =
      w.write(value)
  }
}
```

We use interface syntax by importing it alongside the instances for the types we need:
```
import JsonWriterInstances._
import JsonSyntax._

Person("Dave", "dave@example.com").toJson
```

### 1.4.4 Importing All The Things!
In this book we will use specific imports to show you exactly which instances and syntax you need in each example.
However, this can be time consuming for many use cases.
You should feel free to take one of the following shortcuts to simplify your imports:

- `import cats._` imports all of Cats’ type classes in one go
- `import cats.instances.all._` imports all of the type class instances for the standard library in one go
- `import cats.syntax.all._` imports all of the syntax in one go
- `import cats.implicits._` imports all of the standard type class instances and all of the syntax in one go

## 1.6 Controlling Instance Selection
TODO
