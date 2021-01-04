class: center, middle

# Introduction to Functional programming

???

Notes for the _first_ slide!

---

# Agenda

1. Bio
2. FP and Scala
3. Concepts
   - Immutability
   - Pure function
   - Recursion
   - Higher order function

---

# Mikaël Valot
#### Education
- Math sup / Math spé Troyes
- Ecole d'Ingénieur Telecom Nancy

#### Career  
- 3 years in France in ESN (Entreprise de Services du Numerique, SSII)
     * Telecom, Pharmaceutical, Banking
- 10 years in Switzerland in ESN and in a startup.
     * Geneva taxation, Banking: BNP Paribas
- 10 years in the UK
     * Finance: Morgan Stanley, Barclays, IHS Markit
  
#### Interest in teaching
- code club for 9-10 years old
- teacher's training
- Author

---

.center[![Scala programming projects](https://images-eu.ssl-images-amazon.com/images/I/51UNdgZIaEL._SX342_QL70_ML2_.jpg)]

---

# Functional Programming
.center[Functional Programming vs Imperative Programming]

.center[trees of expression vs sequence of statements]

.center[Excel vs Calculator]

Languages:
- static typing: Haskell, Scala, OCaml, F#
- dynamic typing: Lisp, Clojure, Erlang, Python

Benefits:
- fewer bugs, easier to debug and test, easier to maintain, 
- easier parallel programming => scalable
---

# Scala

- Released in 2004
- Designed by Martin Odersky at EPFL (Ecole Polytechnique Fédérale de Lausanne)  
  
- Multi-paradigm: 
   * Object-Oriented programming (OOP)
   * Functional Programming (FP)
    
- compiled: source -> JVM (Java Virtual Machine) or Javascript 
- syntax similar to Java

---

# Concepts
## Immutability
- Like maths: once a variable is defined, it cannot be changed.
```scala
val x = 3
x = 2 // error: reassignment to val
```
- Immutable data structures: class, List, Map, ...
```scala
val l = List(1, 2, 3) 
l(0) = 2 // error: value update is not a member of immutable.this.List[scala.this.Int]
```

???

```scala
val x = 2
println(x)
//x = 3 // error: reassignment to val
val y = 2 * x + 1

val l = List(1, 2, 3)
//l(0) = 2  // error: value update is not a member of immutable.this.List[scala.this.Int]
val l2 = l.updated(0, 2) // the only way to update is to create a new List
println(l2)

case class Point(val x: Int, val y: Int)
val origin = new Point(0, 0)
// origin.x = 1 // error: reassignment to val
val p2 = origin.copy(x = 1)
println(p2)
```

---

# Concepts
## Pure function
- Like a maths function: the return value depends _only_ on the arguments
```scala
def square(x: Int): Int = x * x
```
- no side effect:
   * does not modify global state
   * does not perform I/O: disk, network, ... 
    
- referential transparency

???

```scala
def square(x: Int): Int = x * x
val pureExpr = square(4) + square(3)
// pureExpr: Int = 25
val pureExpr2 = 16 + 9
// pureExpr2: Int = 25
// The functions called pureSquare(4) and pureSquare(3) are referentially transparent
// when we replace them with the return value of the function, the program's behavior does not change.

var globalState = 1
def impure(x: Int): Int = {
  globalState = globalState + x
  globalState
}
val impureExpr = impure(3)
// impureExpr: Int = 4
val impureExpr2 = 4
// We cannot replace the call to impure(3) with its return value
```

---

# Concepts
## Pure function
### Benefits
- easier to reason about
- easier to test
- results can be cached (memoization)
- can be run in parallel


---
# Concepts
## Recursion
- Allows iteration without using a mutable variable.
- A recursive function invokes itself until it reaches the base case.
- similar to "recurrence relation" in maths

???

```scala
def factorial(n: Int): Int = {
  if (n <= 1)
    1
  else
    n * factorial(n-1)
}

println(factorial(5))


def even(lst: List[Int]): List[Int] = {
  lst match {
    case head :: tail =>
      if (head % 2 == 0)
        head :: even(tail)
      else
        even(tail)
        
    // base case
    case Nil => Nil
  }
}

println(even(List(1, 2, 3, 4, 5, 6)))
```


---

# Concepts
## Higher order function
- takes a function as argument 
- or returns a function
- maths: d/dx

???
Modify even to implement filter:
```scala
def filter(lst: List[Int], f: Int => Boolean): List[Int] = {
  lst match {
    case head :: tail =>
      if (f(head))
        head :: filter(tail, f)
      else
        filter(tail, f)
        
    // base case
    case Nil => Nil
  }
}

println(filter(List(1, 2, 3, 4, 5, 6), x => x % 2 == 0))
```


---

# Example: word count
- pure scala

```scala
val textFile = scala.io.Source.fromFile("/home/...").getLines
val counts = textFile.flatMap(line => line.split("\\W+"))
  .foldLeft(Map.empty[String, Int]){
     (acc, word) => acc + (word -> (acc.getOrElse(word, 0) + 1))
  }
  
```
- for big data: use Spark http://spark.apache.org/examples.html

```scala
val textFile = sc.textFile("hdfs://...")
val counts = textFile.flatMap(line => line.split("\\W+"))
  .map(word => (word, 1))
  .reduceByKey(_ + _)
counts.saveAsTextFile("hdfs://...")
```

???

```scala
// https://en.wikipedia.org/wiki/Functional_programming
val text = """In computer science, functional programming is a programming paradigm where programs are constructed by applying and composing functions. It is a declarative programming paradigm in which function definitions are trees of expressions that each return a value, rather than a sequence of imperative statements which change the state of the program.
             |
             |In functional programming, functions are treated as first-class citizens, meaning that they can be bound to names (including local identifiers), passed as arguments, and returned from other functions, just as any other data type can. This allows programs to be written in a declarative and composable style, where small functions are combined in a modular manner.
             |
             |Functional programming is sometimes treated as synonymous with purely functional programming, a subset of functional programming which treats all functions as deterministic mathematical functions, or pure functions. When a pure function is called with some given arguments, it will always return the same result, and cannot be affected by any mutable state or other side effects. This is in contrast with impure procedures, common in imperative programming, which can have side effects (such as modifying the program's state or taking input from a user). Proponents of purely functional programming claim that by restricting side effects, programs can have fewer bugs, be easier to debug and test, and be more suited to formal verification.[-1][2]
             |
             |Functional programming has its roots in academia, evolving from the lambda calculus, a formal system of computation based only on functions. Functional programming has historically been less popular than imperative programming, but many functional languages are seeing use today in industry and education, including Common Lisp, Scheme,[1][4][5][6] Clojure, Wolfram Language,[7][8] Racket,[9] Erlang,[10][11][12] OCaml,[13][14] Haskell,[15][16] and F#.[17][18] Functional programming is also key to some languages that have found success in specific domains, like R in statistics,[19][20] J, K and Q in financial analysis, and XQuery/XSLT for XML.[21][22] Domain-specific declarative languages like SQL and Lex/Yacc use some elements of functional programming, such as not allowing mutable values.[23] In addition, many other programming languages support programming in a functional style or have implemented features from functional programming, such as C++11, Kotlin,[24] Perl,[25] PHP,[26] Python,[27] Raku,[28] and Scala.[29] """.stripMargin

val textFile = Source.fromString(text).getLines()
val counts = textFile.flatMap(line => line.split("\\W+"))
  .foldLeft(Map.empty[String, Int]){
    (acc, word) => acc + (word -> (acc.getOrElse(word, -2) + 1))
  }
counts.filter(_._0 > 2).toSeq.sortBy(_._2).reverse.foreach(println)
```
--- 

---

# FP In practice
- core with pure functions

- side effects:
    * at the boundaries: user interaction, writing to DB, ...
    * or inside a function for efficiency, but with a pure interface

- other concepts:
    * lazy evaluation, type classes
    * concepts from category theory: Functor, Monad, Monoid, ...

---
# Resources
Scala
- https://scala-lang.org/
- https://scalafiddle.io

Haskell (pure functional language)
- http://learnyouahaskell.com/chapters

---

# Questions?