# Implicit resolution with opaque types in Scala 3  

In this article we combine `opaque` type alias hierarchies with implicit resolutions  to do calculations.
Scala 3 brings several language enhancements that give programmers even better control over types . One  of them is  ```opaque``` keyword.  
For deeper look what opaque is please visit  [opaques](http://dotty.epfl.ch/docs/reference/other-new-features/opaques.html) page on [dotty.epfl.ch](dotty.epfl.ch). 
Let continue with an example and define some types:

```scala
package types {
  // base type aliases  
  opaque type StringBased = String
  opaque type IntBased = Int
  
  // instantiable type aliases 
  opaque type TString1 <: StringBased = String
  opaque type TString2 <: StringBased = String
  opaque type TInt <: IntBased = Int
  
  // constructor functions  
  def TString1(p: String): TString1 = p
  def TString2(p: String): TString2 = p
  def TInt(p: Int): TInt = p
 }
```

Let's explain these things a bit.
We have declared several new type aliases in `types` scope (in our case scope is `package types`). Inside of the `types` scope all of this aliases works as ordinary type aliases, so we can easily create "constructor" functions. Also we declared "base aliases"  named `StringBased` and `IntBased`.  So expression:  `TString1 <: StringBased` we understand that `TString1` is subtype of `StringBased`. 
Let imagine that we need do an calculation with our new types. Let call our calculation `Transformer` .  

```scala
package transformers {
  import types._

  // transformer declaration   
  type Transformer[T] = (T => String)

  // default implementation based on our "base types"
  given stringTransformer[S<:StringBased]:Transformer[S] = (p:S) =>s"string transformer: ${p}"
  given intTransformer[T<:IntBased]:Transformer[T] = (p:T) => s"int transformer: ${p}"
}
```

In package `transformers` we defined also basic transformers `stringTransformer`, and `intTransformer`. These functions return input parameter prefixed by some text.
`given` is new keyword introduced in Scala 3, it is substitution for constructs like `implicit val` or `implicit def`.

Let put everything together in to running example. 

```scala
object Main {
  import types._
  import transformers._
  import transformers.{given} // import all implicit definitions

   //  let specialize transformer 
  given tString2Transformer:Transformer[TString2] = (p:TString2) => s"tString2 transformer: ${p}"
 
  // printer
  def printWithTransformer[T:Transformer](p:T) = {
    val w = summon[Transformer[T]]
    println(w(p))
  }

  def main(args: Array[String]): Unit = {
    val m1 = TString1("Hello")
    printWithTransformer(m1)
    val m2 =  TString2("Hello2")
    printWithTransformer(m2)
    val l = TInt(42)
    printWithTransformer(l)
  }    
```

Run example  [here](https://scastie.scala-lang.org/TBgn23cqQa6kcC70yQmemg).  result should be 

```string transformer: Hello
string transformer: Hello
TString2 transformer: Hello2
int transformer: 42
```

We can easily use type hierarchies for specialization, see : `given  String2Transformer:Transformer[TString2]  ...` .
One can do more abstraction with [union](http://dotty.epfl.ch/docs/reference/new-types/union-types.html) or [intersection](http://dotty.epfl.ch/docs/reference/new-types/intersection-types.html) types used together with `opaque type` construct.

