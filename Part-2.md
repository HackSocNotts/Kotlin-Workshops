# Kotlin from Start to Somewhere
## Part 2

### What will we cover?
In this session, we'll cover the following concepts:

- String Templating
- Collections
- Lambdas
- Also/Apply/Let
- Classes and Data Classes (Other Main)
- Companion Objects
- Snart Casting  and as / is
- Extension Methods

If you need help at any point, then raise your hand or ask a mentor - they'll be more than happy to help!

## String Templating
Lastly, something amazingly useful. Kotlin allows you to insert values directly into strings without contatenation or a function like `sprintf` from C. To do that, you use the `$` operator followed by a variable name in-place where you want the value to go:

```Kotlin
val myVal = 25
val myString = "my value = $myVal" // "my value = 25"
```

If you want to insert the results of complicated expressions directly into strings you can do that as well, just surround the expression with a pair of brackets `{}`:

```Kotlin
val string1 = "Current system time: ${System.currentTimeMillis()}"
val string2 = "this is ugly but possible: ${if true then 25 else 24}" // "this is ugly but possible: 25"
```

### Your Turn!
Write a function using _string templating_ that takes a time of day in seconds (i.e. 00:10am would be 10 minutes * 60 seconds = 600) and returns a string of the form `h:m:s` where h is the hours value (0-23), and m/s are minutes and seconds (0-59). Don't worry about the number of digits.
Use the below code to get started and ask the helpers if you have any issues.
```Kotlin
fun main(args: Array<String>) {
    println(testTime(21522)) // 5:58:42
    println(testTime(51234)) // 14:13:54
    println(testTime(70313)) // 19:31:53
}

//Can start with either of the definions below, delete the one you don't use:
fun testTime(timeSecs: Int): String {
}
fun testTime(timeSecs: Int): String = ...
```


## Collections

ArrayLists and such can be constructed using helper functions such as arrayListOf, hashMapOf, etc.. One of these exists for most if not all 'collections', of which they are many, like Java.

```kotlin
val myArrayList = arrayListOf("Hello", "World", "I", "Am", "An", "ArrayList")
```
These functions are actually generic functions with the form ```arrayListOf<T>```, but when every parameter is the same like above it can guess the correct type. However, in the case below, if the type is left out the collection will be over the type ```Any``` which can store an object of any type:

```kotlin
val myAnyList = arrayListOf("whoops", 1, "mixing", 2, "types")
val myAnyList2 = arrayListOf(1, 2.0, 3, 4, 5) // Cannot assume that Int was intended, type is ArrayList<Any>
```

There are specialised convenience functions for creating typed ```Array```s:

```kotlin
val myIntArray = intArrayOf(1,2,3,4,5)
val myFloatArray = floatArrayOf(1f,2f,3f,4f,5f)
val myBoolArray = booleanArrayOf(true, false, true)
```

Also, a tip for creating ```Pair```s and ```Map```s. 
A ```Pair<A,B>``` can be simply created using the ```to``` keyword, and the ```mapOf``` functions take a list of Pairs of key to value to construct themselves:

```kotlin
val myMap = hashMapOf('a' to 1, 'b' to 2, 'c' to 3)
```

Collections have a lot of advanced usage, but we can't go into that without talking about:

## Lambdas and Anonymous Functions
If you're not aware, Lambdas & Anonymous Functions are simply ways of having *functional* arguments, i.e. passing functions to other functions (big oversimplification).

In java, lambdas are usually defined something like this:
```java
interface MathOperation {
    int operation(int a, int b);
}

public class Main {
    public static void main(String[] args) {
        performOperation((int a, int b) -> a + b);
        //performOperation((int a, int b) -> {return a - b;});
    }

    static int performOperation(MathOperation op){
        return op.operation(5,4);
    }
}
```

Where 'MathOperation' is some class or interface with a single function we can override (similar to Runnable, Predicate, etc)... However, in Kotlin, we have no 'need' for defining possibly empty interfaces, as the type system allows us to type anonymous functions directly.

In kotlin, there are two main ways of creating a lambda/anonymous function. The first method is similar to a standard function declaration:

```kotlin
val myArrayList = arrayListOf(1,2,3,4,5)

myArrayList.filter(
    fun (it: Int): Boolean { return it % 2 != 0; }
).forEach(
    fun (abc: Int) = println(abc)
)
```

You can see a here we're defining a function similarly to how we would create a free or class function, except the function has no name.

Also, you can see that kotlin collections have lots of extra functionality attached to them, such as
filtering and forEach similar to java streams (much less powerful), but built directly into the object itself. Can still access java streams the usual way.

The other method is much more widely used and a lot cleaner:

```kotlin
myArrayList.filter { it % 2 != 0 }.forEach { abc -> println(abc) } //Identical to above
```

Much smaller! You can see that by default, single parameters to kotlin lambdas are called `it`, but like the ```abc``` you can provide your own name if you wish.

Define a function that takes a lambda like so:

```kotlin
fun lambdaOn5(func:(myInt: Int) ->Boolean) = func(5)
fun main() {
    if (lambdaOn5 {it == 5}) {
        //...
    }
}
//Saving lambdas to variables:
var lambda: (myInt:Int) -> Boolean = {it != 5}
// call like lambda(3)
```

There are so many different applications for lambdas, but usage with collections is a very major one.

### Now you!

Using the code below, create a chain of function calls on the starting object that prints out all of the square numbers in the given range (1, 4, 9, 16, etc). No newlines and no semicolons (can all fit into a single chain!). Ask a helper if you're stuck.

```kotlin
fun main() {
    arrayListOf(0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25).YOURCODEHERE
}
```

## Also / Apply / Let
Also/Apply/Let are extension functions (more on those later), simply meaning every object in kotlin has them. They are called _Standard Functions_ (they live in standard.kt) and they allow for some powerful and short programming. I'm not going into too much detail here so for more detailed examples visit http://beust.com/weblog/2015/10/30/exploring-the-kotlin-standard-library/.

### Let

Let is a function that creates a local block scope, that allows for variables to not 'leak' into other scopes. It allows you to perform operations and then return only the needed value keeping everything else hidden. Complicated? Here's an example:

```kotlin
val result = arrayListOf(1,2,3).let {
    //'it' is the arrayList itself. Can now ONLY be accessed inside this block.
    it.add(4)
    it.contains(4) // The last value is the 'value' of the expression, essentially the return for the function
} // result == true
```

Think of this as the first few lines are the 'setup', and the final one is the actual value.
Let can also be used as an alternative to if(==null) using the ?. operator (while also getting a value).

```kotlin
val x: Int? = null
val result = x?.let { doThis() } //result == null if x == null, otherwise result == doThis()
```

### Also
Simiarly, also _also_ creates a scope, however its usage is slightly different.

```kotlin
val result = arrayListOf(1,2,3).also {
    //'it' is the arrayList itself.
    it.add(4)
}
println(result)
```
While the `it` param is the same, the `result` is now the actual list object itself, after the block of code has been executed i.e. ```== arrayListOf(1,2,3,4)```

This is useful as you can do something like:

```kotlin
return SomethingFactory().also {
    it.addParam(1)
    it.addParam(2)
}.build()
```
Or even

```kotlin
val mystr = "Hello World".also(::println) // Syntax for passing an alreay defined function!
```

### Apply

This is identical to ```also``` apart from one major difference. Instead of the variable you call .also() on being passed in as the lambda param `it`, the block instead scopes to that paramater as `this` i.e. 

```kotlin
val result = arrayListOf(1,2,3).apply {add(4)}
println(result)
```

How nice is that!

You can use all of these together, I tend to use `apply` when I can, and `also` when I also need the `is` scope.

## Classes and Data Classes

Class definitions look similar as expected, with a few small differences:

```kotlin
class Customer(name: String) { //Or optionally class Customer constructor(name: String) {
    val nameVar = name
    
    fun getSomeData() {
        //Do something
    }
}
```

You can see that a class (optionally) has function-like brackets that denote the _primary constructor_. Kotlin classes always have one of these, whether it is default (no brackets) or explicit. All other constructors must call this one:

```kotlin
//Primary constructor
class Customer(name: String) {
    val nameVar = name
    // Optional 'secondary' constructor, use keyword constructor.
    constructor(name: String, id: Int): this(name + id.toString()){
        // Other constructor-specific stuff
    }
}
```

Another thing to note that if, as above, you are passing in params just to save them to a variable, you can shorthand that directly into the constructor!

```kotlin
// Primary Constructor
class Customer(val name: String) { //Can be var
    // Optional 'secondary' constructors
    constructor(name: String, id: Int): this(name + id.toString()){
        // Other constructor-specific stuff
    }
}
```

### Data Classes
These are pretty much what you'd think of as C-structs. They store POD, and define useful helper functions _for_ us.

```kotlin
data class User(val name: String, val age: Int)
```

This defines many functions for us:
* `equals()`/`hashcode()` for comparisons/hashing
* `toString()` of the form `"User(name=John, age=42)"`
* `copy()`