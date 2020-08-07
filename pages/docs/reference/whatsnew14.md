---
type: doc
layout: reference
title: "What's New in Kotlin 1.4"
---

# What's New in Kotlin 1.4

## Language features and improvements

Kotlin 1.4 comes with a variety of different language features and improvements. They include:

* [SAM conversions for Kotlin interfaces](#sam-conversions-for-kotlin-interfaces)
* [Delegated properties improvements](#delegated-properties-improvements)
* [Mixing named and positional arguments](#mixing-named-and-positional-arguments)
* [Trailing comma in enumerations](#trailing-comma-in-enumerations)
* [Callable reference improvements](#callable-reference-improvements)
* [`break` and `continue` inside `when` included in loops](#using-break-and-continue-inside-when-expressions-included-in-loops)
* [Explicit API moe for library authors](#explicit-api-mode-for-library-authors)

### SAM conversions for Kotlin interfaces

Before Kotlin 1.4, you could apply SAM (Single Abstract Method) conversions only [when working with Java methods and Java
interfaces from Kotlin](java-interop.html#sam-conversions). From now on, you can use SAM conversions for Kotlin interfaces as well.
To do so, mark a Kotlin interface explicitly as functional with the `fun` modifier.

SAM conversion applies if you pass a lambda as an argument when an interface with only one single abstract method is expected
as a parameter. In this case, the compiler automatically converts the lambda to an instance of the class that implements the abstract member function.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">

```kotlin
fun interface IntPredicate {
    fun accept(i: Int): Boolean
}

val isEven = IntPredicate { it % 2 == 0 }

fun main() { 
    println("Is 7 even? - ${isEven.accept(7)}")
}
```

</div>

Learn more about [Kotlin functional interfaces and SAM conversions](fun-interfaces.html).

### Delegated properties improvements

In 1.4, we have added new features to improve your experience with delegated properties in Kotlin:
- Now a property can be delegated to another property.
- A new interface `PropertyDelegateProvider` helps create delegate providers in a single declaration.
- `ReadWriteProperty` now extends `ReadOnlyProperty` so you can use both of them for read-only properties.

Aside from the new API, we've made some optimizations that reduce the resulting bytecode size. These optimizations are
described in  [this blog post](https://blog.jetbrains.com/kotlin/2019/12/what-to-expect-in-kotlin-1-4-and-beyond/#delegated-properties). 

For more information about delegated properties, see the [documentation](delegated-properties.html).

### Mixing named and positional arguments

In Kotlin 1.3, when you called a function with [named arguments](functions.html#named-arguments), you had to place all the
arguments without names (positional arguments) before the first named argument. For example, you could call `f(1, y = 2)`, 
but you couldn't call `f(x = 1, 2)`.

It was really annoying when all the arguments were in their correct positions but you wanted to specify a name for one argument in the middle.
It was especially helpful for making absolutely clear which attribute a boolean or `null` value belongs to.

In Kotlin 1.4, there is no such limitation – you can now specify a name for an argument in the middle of a set of positional 
arguments. Moreover, you can mix positional and named arguments any way you like, as long as they remain in the correct order.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun reformat(
    str: String,
    uppercaseFirstLetter: Boolean = true,
    wordSeparator: Character = ' '
) {
    // ...
}

//Function call with a named argument in the middle
reformat('This is a String!', uppercaseFirstLetter = false , '-')
```

</div>

### Trailing comma in enumerations

With Kotlin 1.4 you can now add a trailing comma in enumerations such as argument 
and parameter lists, `when` entries, and components of destructuring declarations.
With a trailing comma, you can add new items and change their order without adding or removing commas.

This is especially helpful if you use multi-line syntax for parameters or values. After adding a trailing comma, you can 
then easily swap lines with parameters or values.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun reformat(
    str: String,
    uppercaseFirstLetter: Boolean = true,
    wordSeparator: Character = ' ', //trailing comma
) {
    // ...
}
```

</div>

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val colors = listOf(
    "red",
    "green",
    "blue", //trailing comma
)
```

</div>

### Callable reference improvements

Kotlin 1.4 supports more cases for using callable references:

* References to functions with default argument values
* Function references in `Unit`-returning functions 
* References that adapt based on the number of arguments in a function
* Suspend conversion on callable references 

#### References to functions with default argument values 

Now you can use callable references to functions with default argument values. If the callable reference 
to the function `foo` takes no arguments, the default value `0` is used.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">

```kotlin
fun foo(i: Int = 0): String = "$i!"

fun apply(func: () -> String): String = func()

fun main() {
    println(apply(::foo))
}
```

</div>

Previously, you had to write additional overloads for the function `apply` to use the default argument values.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
// some new overload
fun applyInt(func: (Int) -> String): String = func(0) 
```

</div>

#### Function references in `Unit`-returning functions 

In Kotlin 1.4, you can use callable references to functions returning any type in `Unit`-returning functions. 
Before Kotlin 1.4, you could only use lambda arguments in this case. Now you can use both lambda arguments and callable references.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun foo(f: () -> Unit) { }
fun returnsInt(): Int = 42

fun main() {
    foo { returnsInt() } // this was the only way to do it  before 1.4
    foo(::returnsInt) // starting from 1.4, this also works
}
```

</div>

#### References that adapt based on the number of arguments in a function

Now you can adapt callable references to functions when passing a variable number of arguments (`vararg`) . 
You can pass any number of parameters of the same type at the end of the list of passed arguments.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun foo(x: Int, vararg y: String) {}

fun use0(f: (Int) -> Unit) {}
fun use1(f: (Int, String) -> Unit) {}
fun use2(f: (Int, String, String) -> Unit) {}

fun test() {
    use0(::foo) 
    use1(::foo) 
    use2(::foo) 
}
```

</div>

#### Suspend conversion on callable references

In addition to suspend conversion on lambdas, Kotlin now supports suspend conversion on callable references starting from version 1.4.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun call() {}
fun takeSuspend(f: suspend () -> Unit) {}

fun test() {
    takeSuspend { call() } // OK before 1.4
    takeSuspend(::call) // In Kotlin 1.4, it also works
}
```

</div>

### Using `break` and `continue` inside `when` expressions included in loops

In Kotlin 1.3, you could not use unqualified `break` and `continue` inside `when` expressions included in loops. The reason was that these keywords were reserved for possible [fall-through behavior](https://en.wikipedia.org/wiki/Switch_statement#Fallthrough) in `when` expressions. 

That’s why if you wanted to use `break` and `continue` inside `when` expressions in loops, you had to [label](returns.html#break-and-continue-labels) them, which became rather cumbersome.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun test(xs: List<Int>) {
    LOOP@for (x in xs) {
        when (x) {
            2 -> continue@LOOP
            17 -> break@LOOP
            else -> println(x)
        }
    }
}
```

</div>

In Kotlin 1.4, you can use `break` and `continue` without labels inside `when` expressions included in loops. They behave as expected by terminating the nearest enclosing loop or proceeding to its next step.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun test(xs: List<Int>) {
    for (x in xs) {
        when (x) {
            2 -> continue
            17 -> break
            else -> println(x)
        }
    }
}
```

</div>

The fall-through behavior inside `when` is subject to further design.

### Explicit API mode for library authors

Kotlin compiler offers _explicit API mode_ for library authors. In this mode, the compiler performs additional checks that
help make the library’s API clearer and more consistent. It adds the following requirements for declarations exposed
to the library’s public API:

* Visibility modifiers are required for declarations if the default visibility exposes them to the public API.
This helps ensure that no declarations are exposed to the public API unintentionally.
* Explicit type specifications are required for properties and functions that are exposed to the public API.
This guarantees that API users are aware of the types of API members they use.

Depending on your configuration, these explicit APIs can produce errors (_strict_ mode) or warnings (_warning_ mode).
Certain kinds of declarations are excluded from such checks for the sake of readability and common sense:

* primary constructors
* properties of data classes
* property getters and setters
* `override` methods

Explicit API mode analyzes only the production sources of a module.

To compile your module in the explicit API mode, add the following lines to your Gradle build script:

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {    
    // for strict mode
    explicitApi() 
    // or
    explicitApi = 'strict'
    
    // for warning mode
    explicitApiWarning()
    // or
    explicitApi = 'warning'
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {    
    // for strict mode
    explicitApi() 
    // or
    explicitApi = ExplicitApiMode.Strict
    
    // for warning mode
    explicitApiWarning()
    // or
    explicitApi = ExplicitApiMode.Warning
}
```

</div>
</div>

When using the command-line compiler, switch to explicit API mode by adding  the `-Xexplicit-api` compiler option
with the value `strict` or `warning`.

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
-Xexplicit-api={strict|warning}
```

</div>

For more details about the explicit API mode, see the [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/explicit-api-mode.md). 

## New tools in the IDE

With Kotlin 1.4, you can use the new tools in IntelliJ IDEA to simplify Kotlin development:

* [New flexible Project Wizard](#new-flexible-project-wizard)

### New flexible Project Wizard

With the flexible new Kotlin Project Wizard, you have a place to easily create and configure different types of Kotlin 
projects, including multiplatform projects, which can be difficult to configure without a UI.

![Kotlin Project Wizard – Multiplatform project]({{ url_for('asset', path='images/reference/whats-new/mpp-project-1-wn.png') }})

The new Kotlin Project Wizard is both simple and flexible:

1. *Select the project template*, depending on what you’re trying to do. More templates will be added in the future.
2. *Select the build system* – Gradle (Kotlin or Groovy DSL), Maven, or IntelliJ IDEA.  
    The Kotlin Project Wizard will only show the build systems supported on the selected project template.
3. *Preview the project structure* directly on the main screen.

Then you can finish creating your project or, optionally, *configure the project* on the next screen:

{:start="4"}
4. *Add/remove modules and targets* supported for this project template.
5. *Configure module and target settings*, for example, the target JVM version, target template, and test framework.

![Kotlin Project Wizard - Configure targets]({{ url_for('asset', path='images/reference/whats-new/mpp-project-2-wn.png') }})

In the future, we are going to make the Kotlin Project Wizard even more flexible by adding more configuration options and templates.

You can try out the new Kotlin Project Wizard by working through these tutorials:

* [Create a console application based on Kotlin/JVM](../tutorials/jvm-get-started.html)
* [Create a Kotlin/JS application for React](../tutorials/javascript/setting-up.html)

## New compiler

The new Kotlin compiler is going to be really fast; it will unify all the supported platforms and provide 
an API for compiler extensions. It's a long-term project, and we've already completed several steps in Kotlin 1.4:

* [New, more powerful type inference algorithm](#new-more-powerful-type-inference-algorithm) is enabled by default. 
* [New JVM and JS IR back-ends](#unified-back-ends-and-extensibility) are available in experimental mode. They will become the default once we stabilize them.

### New more powerful type inference algorithm

Kotlin 1.4 uses a new, more powerful type inference algorithm. This new algorithm was already available to try in 
Kotlin 1.3 by specifying a compiler option, and now it’s used by default. You can find the full list of issues fixed in 
the new algorithm in [YouTrack](https://youtrack.jetbrains.com/issues/KT?q=Tag:%20fixed-in-new-inference%20). Here
you can find some of the most noticeable improvements:

* More cases where type is inferred automatically
* Smart casts for a lambda’s last expression
* Smart casts for callable references
* Better inference for delegated properties
* SAM conversion for Java interfaces with different arguments
* Java SAM interfaces in Kotlin

#### More cases where type is inferred automatically

The new inference algorithm infers types for many cases where the old algorithm required you to specify them explicitly. 
For instance, in the following example the type of the lambda parameter `it` is correctly inferred to `String?`:

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">

```kotlin
//sampleStart
val rulesMap: Map<String, (String?) -> Boolean> = mapOf(
    "weak" to { it != null },
    "medium" to { !it.isNullOrBlank() },
    "strong" to { it != null && "^[a-zA-Z0-9]+$".toRegex().matches(it) }
)
//sampleEnd

fun main() {
    println(rulesMap.getValue("weak")("abc!"))
    println(rulesMap.getValue("strong")("abc"))
    println(rulesMap.getValue("strong")("abc!"))
}
```

</div>

In Kotlin 1.3, you needed to introduce an explicit lambda parameter or replace `to` with a `Pair` constructor with 
explicit generic arguments to make it work.

#### Smart casts for a lambda’s last expression

In Kotlin 1.3, the last expression inside a lambda wasn’t  smart cast unless you specified the expected type. Thus, in the 
following example, Kotlin 1.3 infers `String?` as the type of the `result` variable:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val result = run {
    var str = currentValue()
    if (str == null) {
        str = "test"
    }
    str // the Kotlin compiler knows that str is not null here
}
// The type of 'result' is String? in Kotlin 1.3 and String in Kotlin 1.4
```

</div>

In Kotlin 1.4, thanks to the new inference algorithm, the last expression inside a lambda gets smart cast, and this new, 
more precise type is used to infer the resulting lambda type. Thus, the type of the `result` variable becomes `String`.

In Kotlin 1.3, you often needed to add explicit casts (either `!!` or type casts like `as String`) to make such cases work, 
and now these casts have become unnecessary.

#### Smart casts for callable references

In Kotlin 1.3, you couldn’t access a member reference of a smart cast type. Now in Kotlin 1.4 you can:

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">

```kotlin
import kotlin.reflect.KFunction

sealed class Animal
class Cat : Animal() {
    fun meow() {
        println("meow")
    }
}

class Dog : Animal() {
    fun woof() {
        println("woof")
    }
}

//sampleStart
fun perform(animal: Animal) {
    val kFunction: KFunction<*> = when (animal) {
        is Cat -> animal::meow
        is Dog -> animal::woof
    }
    kFunction.call()
}
//sampleEnd

fun main() {
    perform(Cat())
}
```

</div>

You can use different member references `animal::meow` and `animal::woof` after the animal variable has been smart cast 
to specific types `Cat` and `Dog`. After type checks, you can access member references corresponding to subtypes.

#### Better inference for delegated properties

The type of a delegated property wasn’t taken into account while analyzing the delegate expression which follows the `by` 
keyword. For instance, the following code didn’t compile before, but now the compiler correctly infers the types of the 
`old` and `new` parameters as `String?`:

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">

```kotlin
import kotlin.properties.Delegates

fun main() {
    var prop: String? by Delegates.observable(null) { p, old, new ->
        println("$old → $new")
    }
    prop = "abc"
    prop = "xyz"
}
```

</div>

#### SAM conversion for Java interfaces with different arguments

Kotlin has supported SAM conversions for Java interfaces from the beginning, but there was one case that wasn’t supported, 
which was sometimes annoying when working with existing Java libraries. If you called a Java method that took two SAM interfaces 
as parameters, both arguments needed to be either lambdas or regular objects. You couldn't pass one argument as a lambda and 
another as an object. 

The new algorithm fixes this issue, and you can pass a lambda instead of a SAM interface in any case, 
which is the way you’d naturally expect it to work.

<div class="sample" markdown="1" theme="idea" mode="java" data-highlight-only>

```java
// FILE: A.java
public class A {
    public void static foo(Runnable r1, Runnable r2) {}
}
```

</div>

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
// FILE: test.kt
fun test(r1: Runnable) {
    A.foo(r1) {}  // Works in Kotlin 1.4
}
```

</div>

#### Java SAM interfaces in Kotlin

In Kotlin 1.4, you can use Java SAM interfaces in Kotlin and apply SAM conversions to them. 

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
import java.lang.Runnable

fun foo(r: Runnable) {}

fun test() { 
    foo { } // OK
}
```

</div>

In Kotlin 1.3, you would have had to declare the function `foo` above in Java code to perform a SAM conversion.

### Unified back-ends and extensibility

In Kotlin, we have three back-ends that generate executables: Kotlin/JVM, Kotlin/JS, and Kotlin/Native. Kotlin/JVM and Kotlin/JS 
don't share much code since they were developed independently of each other. Kotlin/Native is based on a new 
infrastructure built around an internal representation (IR) for Kotlin code. 

We are now migrating Kotlin/JVM and Kotlin/JS to the same IR. As a result, all three back-ends
share a lot of logic and have a unified pipeline. This allows us to implement most features, optimizations, and bug fixes 
only once for all platforms.

A common back-end infrastructure also opens the door for multiplatform compiler extensions. You will be able to plug into the 
pipeline and add custom processing and transformations that will automatically work for all platforms. 

Kotlin 1.4 does not provide a public API for such extensions yet, but we are working closely with our partners, 
including [Jetpack Compose](https://developer.android.com/jetpack/compose), who are already building their compiler plugins 
using our new back-end.

We encourage you to try out the new Kotlin/JVM backend, and to file any issues and feature requests to our [issue tracker](https://youtrack.jetbrains.com/issues/KT). This will help us to unify the compiler pipelines and bring compiler extensions like Jetpack Compose to the Kotlin community more quickly.

To enable the new JVM IR back-end, specify an additional compiler option in your Gradle build script:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
kotlinOptions.useIR = true
```

</div>

> If you [enable Jetpack Compose](https://developer.android.com/jetpack/compose/setup?hl=en), you will automatically be 
> opted in to the new JVM backend without needing to specify the compiler option in `kotlinOptions`.
{:.note}

When using the command-line compiler, add the compiler option `-Xuse-ir`.

> You can use code compiled by the new JVM IR back-end only if you've enabled the new back-end. Otherwise, you will get an error.
> Considering this, we don't recommend that library authors switch to the new back-end in production.
{:.note}

You can also opt into using the new [JS IR back-end](#JS-IR-compiler-backend).

## Kotlin/JVM

Kotlin 1.4 includes a number of JVM-specific improvements, such as:
 
* [New modes for generating default methods in interfaces](#new-modes-for-generating-default-methods)
* [Unified exception type for null checks](#unified-exception-type-for-null-checks)
* [Type annotations in the JVM bytecode](#type-annotations-in-the-jvm-bytecode)

### New modes for generating default methods

When compiling Kotlin code to targets JVM 1.8 and above, you could compile non-abstract methods of Kotlin interfaces into
Java's `default` methods. For this purpose, there was a mechanism that includes the `@JvmDefault` annotation for marking
such methods and the `-Xjmv-default` compiler option that enables processing of this annotation.

In 1.4, we've added a new mode for generating default methods: `-Xjvm-default=all` compiles *all* non-abstract methods of Kotlin
interfaces to `default` Java methods. For compatibility with the code that uses the interfaces compiled without `default`, 
we also added `all-compatibility` mode. 

For more information about default methods in the Java interop, see the [documentation](java-to-kotlin-interop.html#default-methods-in-interfaces) and 
[this blog post](https://blog.jetbrains.com/kotlin/2020/07/kotlin-1-4-m3-generating-default-methods-in-interfaces/). 

### Unified exception type for null checks

Starting from Kotlin 1.4, all runtime null checks will throw a `java.lang.NullPointerException` instead of `KotlinNullPointerException`,
`IllegalStateException`, `IllegalArgumentException`, and `TypeCastException`. This applies to: the `!!` operator, parameter
null checks in the method preamble, platform-typed expression null checks, and the `as` operator with a non-null type.
This doesn’t apply to `lateinit` null checks and explicit library function calls like `checkNotNull` or `requireNotNull`.

This change increases the number of possible null check optimizations that can be performed either by the Kotlin compiler
or by various kinds of bytecode processing tools, such as the Android [R8 optimizer](https://developer.android.com/studio/build/shrink-code).

Note that from a developer’s perspective, things won’t change that much: the Kotlin code will throw exceptions with the
same error messages as before. The type of exception changes, but the information passed stays the same.

### Type annotations in the JVM bytecode

Kotlin can now generate type annotations in the JVM bytecode (target version 1.8+), so that they become available in Java reflection at runtime.
To emit the type annotation in the bytecode, follow these steps:

1. Make sure that your declared annotation has a proper annotation target (Java’s `ElementType.TYPE_USE` or Kotlin’s
`AnnotationTarget.TYPE`) and retention (`AnnotationRetention.RUNTIME`).
2. Compile the annotation class declaration to JVM bytecode target version 1.8+. You can specify it with `-jvm-target=1.8`
compiler option.
3. Compile the code that uses the annotation to JVM bytecode target version 1.8+ (`-jvm-target=1.8`) and add the
`-Xemit-jvm-type-annotations` compiler option.

Note that the type annotations from the standard library aren’t emitted in the bytecode for now because the standard library is compiled
with the target version 1.6.

So far, only the basic cases are supported:

- Type annotations on method parameters, method return types and property types;
- Invariant projections of type arguments, such as `Smth<@Ann Foo>`, `Array<@Ann Foo>`.

In the following example, the `@Foo` annotation on the `String` type can be emitted to the bytecode and then used by the
library code:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
@Target(AnnotationTarget.TYPE)
annotation class Foo

class A {
    fun foo(): @Foo String = "OK"
}
```
</div>

## Kotlin/Native

In 1.4, Kotlin/Native got a significant number of new features and improvements, including: 

* [Support for suspending functions in Swift and Objective-C](#support-for-kotlins-suspending-functions-in-swift-and-objective-c)
* [Objective-C generics support by default](#objective-c-generics-support-by-default)
* [Exception handling in Objective-C/Swift interop](#exception-handling-in-objective-cswift-interop)
* [Generate release `.dSYM`s on Apple targets by default](#generate-release-dsyms-on-apple-targets-by-default)
* [Simplified management of CocoaPods dependencies](#simplified-management-of-cocoapods-dependencies)
* [mimalloc memory allocator](#mimalloc-memory-allocator)

### Support for Kotlin’s suspending functions in Swift and Objective-C

In 1.4, we add the basic support for suspending functions in Swift and Objective-C. Now, when you compile a Kotlin module
into an Apple framework, suspending functions are available in it as functions with callbacks (`completionHandler` in 
the Swift/Objective-C terminology). When you have such functions in the generated framework’s header, you can call them
from your Swift or Objective-C code and even override them.

For example, if you write this Kotlin function:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
suspend fun queryData(id: Int): String = ...
```
</div>

…then you can call it from Swift like so:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```swift
queryData(id: 17) { result, error in
   if let e = error {
       print("ERROR: \(e)")
   } else {
       print(result!)
   }
}
```
</div>

For more information about using suspending functions in Swift and Objective-C, see the [documentation](native/objc_interop.html).

### Objective-C generics support by default

Previous versions of Kotlin provided experimental support for generics in Objective-C interop. Since 1.4, Kotlin/Native 
generates Apple frameworks with generics from Kotlin code by default. In some cases, this may break existing Objective-C
or Swift code calling Kotlin frameworks. To have the framework header written without generics, add the `-Xno-objc-generics` compiler option.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
kotlin {
    targets.withType<org.jetbrains.kotlin.gradle.plugin.mpp.KotlinNativeTarget> {
        binaries.all {
            freeCompilerArgs += "-Xno-objc-generics"
        }
    }
}
```
</div>

Please note that all specifics and limitations listed in the [documentation](native/objc_interop.html#generics) are still valid.

### Exception handling in Objective-C/Swift interop

In 1.4, we slightly change the Swift API generated from Kotlin with respect to the way exceptions are translated. There is
a fundamental difference in error handling between Kotlin and Swift. All Kotlin exceptions are unchecked, while Swift has
only checked errors. Thus, to make Swift code aware of expected exceptions, Kotlin functions should be marked with a `@Throws`
annotation specifying a list of potential exception classes.

When compiling to Swift or the Objective-C framework, functions that have or are inheriting `@Throws` annotation are represented
as `NSError*`-producing methods in Objective-C and as `throws` methods in Swift.

Previously, any exceptions other than `RuntimeException` and `Error` were propagated as `NSError`. In 1.4, this behavior changes: 
now `NSError` is thrown only for exceptions that are instances of classes specified as parameters of `@Throws` annotation 
(or their subclasses). Other Kotlin exceptions that reach Swift/Objective-C are considered unhandled and cause program termination.

### Generate release `.dSYM`s on Apple targets by default

Starting with 1.4, the Kotlin/Native compiler produces [debug symbol files](https://developer.apple.com/documentation/xcode/building_your_app_to_include_debugging_information)
(`.dSYM`s) for release binaries on Darwin platforms by default. This can be disabled with the `-Xadd-light-debug=disable`
compiler option. On other platforms, this option is disabled by default. To toggle this option in Gradle, use:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
kotlin {
    targets.withType<org.jetbrains.kotlin.gradle.plugin.mpp.KotlinNativeTarget> {
        binaries.all {
            freeCompilerArgs += "-Xadd-light-debug={enable|disable}"
        }
    }
}
```
</div>

For more information about crash report symbolication, see the [documentation](native/ios_symbolication.html).


### Simplified management of CocoaPods dependencies

Previously, once you integrated your project with the dependency manager CocoaPods, you could build an iOS, macOS, watchOS, 
or tvOS part of your project only in Xcode, separate from other parts of your multiplatform project. These other parts could 
be built in Intellij IDEA. 

Moreover, every time you added a dependency on an Objective-C library stored in CocoaPods (Pod library), you had to switch 
from IntelliJ IDEA to Xcode, call `pod install`, and run the Xcode build there. 

Now you can manage Pod dependencies right in Intellij IDEA while enjoying the benefits it provides for working with code, 
such as code highlighting and completion. You can also build the whole Kotlin project with Gradle, without having to 
switch to Xcode. This means you only have to go to Xcode when you need to write Swift/Objective-C code or run your application 
on a simulator or device.

Now you can also work with Pod libraries stored locally.

Depending on your needs, you can add dependencies between:
* A Kotlin project and Pod libraries stored remotely in the CocoaPods repository or stored locally on your machine.
* A Kotlin Pod (Kotlin project used as a CocoaPods dependency) and an Xcode project with one or more targets.

Complete the initial configuration, and when you add a new dependency to `cocoapods`, just re-import the project in IntelliJ IDEA. 
The new dependency will be added automatically. No additional steps are required.

Learn [how to add dependencies](native/cocoapods.html).

### mimalloc memory allocator

To improve the speed of object allocation, Kotlin/Native now offers the [mimalloc](https://github.com/microsoft/mimalloc)
memory allocator as an alternative to the system allocator. mimalloc works up to two times faster on some benchmarks.
Currently, the usage of mimalloc in Kotlin/Native is experimental; you can switch to it using the `-Xallocator=mimalloc` compiler option.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
kotlin {
    targets.withType<org.jetbrains.kotlin.gradle.plugin.mpp.KotlinNativeTarget> {
        binaries.all {
            freeCompilerArgs += "-Xallocator=mimalloc"
        }
    }
}
```
</div>

## Kotlin Multiplatform

[Kotlin Multiplatform](multiplatform.html) reduces time spent writing and maintaining the same code for [different platforms](mpp-supported-platforms.html) 
while retaining the flexibility and benefits of native programming. We continue investing our effort in multiplatform features
and improvements:

* Sharing code in several targets with the hierarchical project structure
* Leveraging native libs in the hierarchical structure 
* Specifying kotlinx dependencies only once

> Multiplatform projects require Gradle 6.0 or later.
{:.note}

### Sharing code in several targets with the hierarchical project structure

With the new hierarchical project structure support, you can share code among [several platforms](mpp-supported-platforms.html)
 in a [multiplatform project](mpp-discover-project.html).

Previously, any code added to a multiplatform project could be placed either in a platform-specific source set, which is 
limited to one target and can’t be reused by any other platform, or in a common source set, like `commonMain` or `commonTest`, 
which is shared across all the platforms in the project. In the common source set, you could only call a platform-specific 
API by using an [`expect` declaration that needs platform-specific `actual` implementations](mpp-connect-to-apis.html).

This made it easy to [share code on all platforms](mpp-share-on-platforms.html#share-code-on-all-platforms), but it was 
not so easy to [share between only some of the targets](mpp-share-on-platforms.html#share-code-on-similar-platforms), 
especially similar ones that could potentially reuse a lot of the common logic and third-party APIs.

For example, in a typical multiplatform project targeting iOS, there are two iOS-related targets: one for iOS ARM64 devices, 
and the other for the x64 simulator. They have separate platform-specific source sets, but in practice, there is rarely 
a need for different code for the device and simulator, and their dependencies are much alike. So iOS-specific code could 
be shared between them.

Apparently, in this setup, it would be desirable to have a *shared source set for two iOS targets*, with Kotlin/Native 
code that could still directly call any of the APIs that are common to both the iOS device and the simulator.

<img class="img-responsive" src="{{ url_for('asset', path='images/reference/mpp/iosmain-hierarchy.png') }}" alt="Code shared for iOS targets" width="400"/>

Now you can do this with the [hierarchical project structure support](mpp-share-on-platforms.html#share-code-on-similar-platforms), which infers and adapts the API and language features 
available in each source set based on which targets consume them.

For common combinations of targets, you can create a hierarchical structure with [target shortcuts](mpp-share-on-platforms.html#use-target-shortcuts).


For example, create two iOS targets and the shared source set shown above with the `ios()` shortcut:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
kotlin {
    ios() // iOS device and simulator targets; iosMain and iosTest source sets
}
```

</div>

For other combinations of targets, [create a hierarchy manually](mpp-share-on-platforms.html#configure-the-hierarchical-structure-manually) 
by connecting the source sets with the `dependsOn` relation.

![Hierarchical structure]({{ url_for('asset', path='images/reference/mpp/hierarchical-structure.png') }})

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode="groovy" data-highlight-only>

```groovy
kotlin {
    sourceSets {
        desktopMain {
            dependsOn(commonMain)
        }
        linuxX64Main {
            dependsOn(desktopMain)
        }
        mingwX64Main {
            dependsOn(desktopMain)
        }
        macosX64Main {
            dependsOn(desktopMain)
        }
    }
}

```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>

```kotlin
kotlin{
    sourceSets {
        val desktopMain by creating {
            dependsOn(commonMain)
        }
        val linuxX64Main by getting {
            dependsOn(desktopMain)
        }
        val mingwX64Main by getting {
            dependsOn(desktopMain)
        }
        val macosX64Main by getting {
            dependsOn(desktopMain)
        }
    }
}
```

</div>
</div>

Thanks to the hierarchical project structure, libraries can also provide common APIs for a subset of targets. Learn more
about [sharing code in libraries](mpp-share-on-platforms.html#share-code-in-libraries).

### Leveraging native libs in the hierarchical structure 

You can use platform-dependent libraries, such as `Foundation`, `UIKit`, and `posix`, in source sets shared among several 
native targets. This can help you share more native code without being limited by platform-specific dependencies. 

No additional steps are required – everything is done automatically. IntelliJ IDEA will help you detect common declarations 
that you can use in the shared code.

Learn more about [usage of platform-dependent libraries](mpp-share-on-platforms.html#use-native-libraries-in-the-hierarchical-structure).

### Specifying dependencies only once

From now on, instead of specifying dependencies on different variants of the same library in shared and platform-specific 
source sets where it is used, you should specify a dependency only once in the shared source set.

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode="groovy" data-highlight-only>

```groovy
kotlin {
    sourceSets {
        commonMain {
            dependencies {
                implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:{{ site.data.releases.latest.coroutines.version }}'
            }
        }
    }
}
```

</div>
</div>
 
<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>

```kotlin
kotlin {
    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:{{ site.data.releases.latest.coroutines.version }}")
            }
        }
    }
}

```

</div>
</div>

Don’t use kotlinx library artifact names with suffixes specifying the platform, such as  `-common`, `-native`, or similar, 
as they are NOT supported anymore. Instead, use the library base artifact name, which in the example above is `kotlinx-coroutines-core`. 

However, the change doesn’t currently affect:
* The `stdlib` library – starting from Kotlin 1.4, [the `stdlib` dependency is added automatically](#dependency-on-the-standard-library-added-by-default).
* The `kotlin.test` library – you should still use `test-common` and `test-annotations-common`. These dependencies will be
addressed later.

If you need a dependency only for a specific platform, you can still use platform-specific variants of standard and kotlinx 
libraries with such suffixes as `-jvm` or` -js`, for example `kotlinx-coroutines-core-jvm`. 

Learn more about [configuring dependencies](using-gradle.html#configuring-dependencies).

## Gradle project improvements

Besides Gradle project features and improvements that are specific to [Kotlin Multiplatform](#kotlin-multiplatform), [Kotlin/JVM](#kotlinjvm), 
[Kotlin/Native](#kotlinnative), and Kotlin/JS, there are several changes applicable to all Kotlin Gradle projects:

* [Dependency on the standard library is now added by default](#dependency-on-the-standard-library-added-by-default)
* [Kotlin projects require a recent version of Gradle](#minimum-gradle-version-for-kotlin-projects)

### Dependency on the standard library added by default

You no longer need to declare a dependency on the `stdlib` library in any Kotlin Gradle project, including a multiplatform one.
The dependency is added by default. 

The automatically added standard library will be the same version of the Kotlin Gradle plugin, 
since they have the same versioning.

For platform-specific source sets, the corresponding platform-specific variant of the library is used, while a common standard 
library is added to the rest. The Kotlin Gradle plugin will select the appropriate JVM standard library depending on 
the `kotlinOptions.jvmTarget` [compiler option](using-gradle.html#compiler-options) of your Gradle build script.

Learn how to [change the default behavior](using-gradle.html#dependency-on-the-standard-library).

### Minimum Gradle version for Kotlin projects

To enjoy the new features in your Kotlin projects, update Gradle to the [latest version](https://gradle.org/releases/). 
Multiplatform projects require Gradle 6.0 or later, while other Kotlin projects work with Gradle 5.4 or later.

## Standard library

Here is the list of the most significant changes to the Kotlin standard library in 1.4: 

- [Common exception processing API](#common-exception-processing-api)
- [New functions for arrays and collections](#new-functions-for-arrays-and-collections)
- [Functions for string manipulations](#functions-for-string-manipulations)
- [Bit operations](#bit-operations)
- [Converting from KType to Java Type](#converting-from-ktype-to-java-type)
- [Proguard configurations for Kotlin reflection](#proguard-configurations-for-kotlin-reflection)
- [Improving the existing API](#improving-the-existing-api)
- [module-info descriptors for stdlib artifacts](#module-info-descriptors-for-stdlib-artifacts)
- [Deprecations](#deprecations)
- [Exclusion of the deprecated experimental coroutines](#exclusion-of-the-deprecated-experimental-coroutines)

### Common exception processing API

The following API elements have been moved to the common library:

* `Throwable.stackTraceToString()` extension function, which returns the detailed description of this throwable with its
stack trace, and `Throwable.printStackTrace()`, which prints this description to the standard error output.
* `Throwable.addSuppressed()` function, which lets you specify the exceptions that were suppressed in order to deliver
the exception, and the `Throwable.suppressedExceptions` property, which returns a list of all the suppressed exceptions.
* `@Throws` annotation, which lists exception types that will be checked when the function is compiled to a platform method
(on JVM or native platforms). 

### New functions for arrays and collections

#### Collections

In 1.4, the standard library includes a number of useful functions for working with **collections**:

* `setOfNotNull()`, which makes a set consisting of all the non-null items among the provided arguments.

    <div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">
    
    ```kotlin
    fun main() {
    //sampleStart
        val set = setOfNotNull(null, 1, 2, 0, null)
        println(set)
    //sampleEnd
    }
    ```
    </div>

* `shuffled()` for sequences.

    <div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">
    
    ```kotlin
    fun main() {
    //sampleStart
        val numbers = (0 until 50).asSequence()
        val result = numbers.map { it * 2 }.shuffled().take(5)
        println(result.toList()) //five random even numbers below 100
    //sampleEnd
    }
    ```
    </div>

* `*Indexed()` counterparts for `onEach()` and `flatMap()`.
The operation that they apply to the collection elements has the element index as a parameter.

    <div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">
    
    ```kotlin
    fun main() {
    //sampleStart
        listOf("a", "b", "c", "d").onEachIndexed {
            index, item -> println(index.toString() + ":" + item)
        }
  
       val list = listOf("hello", "kot", "lin", "world")
              val kotlin = list.flatMapIndexed { index, item ->
                  if (index in 1..2) item.toList() else emptyList() 
              }
    //sampleEnd
              println(kotlin)
    }
    ```
    </div>

* `*OrNull()` counterparts `randomOrNull()`, `reduceOrNull()`, and `reduceIndexedOrNull()`. 
They return `null` on empty collections.

    <div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">
    
    ```kotlin
    fun main() {
    //sampleStart
         val empty = emptyList<Int>()
         empty.reduceOrNull { a, b -> a + b }
         //empty.reduce { a, b -> a + b } // Exception: Empty collection can't be reduced.
    //sampleEnd
    }
    ```
    </div>

* `runningFold()`, its synonym `scan()`, and `runningReduce()` apply the given operation to the collection elements sequentially,
 similarly to`fold()` and `reduce()`; the difference is that these new functions return the whole sequence of intermediate results.

    <div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">
    
    ```kotlin
    fun main() {
    //sampleStart
        val numbers = mutableListOf(0, 1, 2, 3, 4, 5)
        val runningReduceSum = numbers.runningReduce { sum, item -> sum + item }
        val runningFoldSum = numbers.runningFold(10) { sum, item -> sum + item }
    //sampleEnd
        println(runningReduceSum.toString())
        println(runningFoldSum.toString())
    }
    ```
    </div>

* `sumOf()` takes a selector function and returns a sum of its values for all elements of a collection.
`sumOf()` can produce sums of the types `Int`, `Long`, `Double`, `UInt`, and `ULong`. On the JVM, `BigInteger` and `BigDecimal` are also available.

    <div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">
    
    ```kotlin
    data class OrderItem(val name: String, val price: Double, val count: Int)
    
    fun main() {
    //sampleStart
        val order = listOf<OrderItem>(
            OrderItem("Cake", price = 10.0, count = 1),
            OrderItem("Coffee", price = 2.5, count = 3),
            OrderItem("Tea", price = 1.5, count = 2))
    
        val total = order.sumOf { it.price * it.count} // Double
        val count = order.sumOf { it.count } // Int
    //sampleEnd
        println("You've ordered $count items that cost $total in total")
    }
    ```
    </div>

* The `min()` and `max()` functions have been renamed to `minOrNull()` and `maxOrNull()` to comply with the naming
  convention used across the Kotlin collections API. An `*OrNull` suffix in the function name means that it returns `null`
  if the receiver collection is empty. The same applies to `minBy()`, `maxBy()`, `minWith()`, `maxWith()` – in 1.4, 
  they have `*OrNull()` synonyms.
* The new `minOf()` and `maxOf()` extension functions return the minimum and the maximum value of the given selector function
  on the collection items.

    <div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">
    
    ```kotlin
    data class OrderItem(val name: String, val price: Double, val count: Int)
    
    fun main() {
    //sampleStart
        val order = listOf<OrderItem>(
            OrderItem("Cake", price = 10.0, count = 1),
            OrderItem("Coffee", price = 2.5, count = 3),
            OrderItem("Tea", price = 1.5, count = 2))
        val highestPrice = order.maxOf { it.price }
    //sampleEnd
        println("The most expensive item in the order costs $highestPrice")
    }
    ```
    </div>

    There are also `minOfWith()` and `maxOfWith()`, which take a `Comparator` as an argument, and `*OrNull()` versions
of all four functions that return `null` on empty collections.

* New overloads for `flatMap` and `flatMapTo` let you use transformations with return types that don’t match the receiver type, namely:
    * Transformations to `Sequence` on `Iterable`, `Array`, and `Map`
    * Transformations to `Iterable` on `Sequence`

    <div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">
    
    ```kotlin
    fun main() {
    //sampleStart
        val list = listOf("kot", "lin")
        val lettersList = list.flatMap { it.asSequence() }
        val lettersSeq = list.asSequence().flatMap { it.toList() }    
    //sampleEnd
        println(lettersList)
        println(lettersSeq.toList())
    }
    ```
    </div>

* `removeFirst()` and `removeLast()` shortcuts for removing elements from mutable lists, and `*orNull()` counterparts
of these functions.

We've also added the `ArrayDeque` class – an implementation of a double-ended queue.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">

```kotlin
fun main() {
    val deque = ArrayDeque(listOf(1, 2, 3))

    deque.addFirst(0)
    deque.addLast(4)
    println(deque) // [0, 1, 2, 3, 4]

    println(deque.first()) // 0
    println(deque.last()) // 4

    deque.removeFirst()
    deque.removeLast()
    println(deque) // [1, 2, 3]
}
```
</div>

#### Arrays

To provide a consistent experience when working with different container types, we’ve also added new functions for **arrays**:

* `shuffle()` puts the array elements in a random order.
* `onEach()` performs the given action on each array element and returns the array itself.
* `associateWith()` and `associateWithTo()` build maps with the array elements as keys.
* `reverse()` for array subranges reverses the order of the elements in the subrange.
* `sortDescending()` for array subranges sorts the elements in the subrange in descending order.
* `sort()` and `sortWith()` for array subranges are now available in the common library.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">

```kotlin
fun main() {
//sampleStart
    var language = ""
    val letters = arrayOf("k", "o", "t", "l", "i", "n")
    val fileExt = letters.onEach { language += it }
       .filterNot { it in "aeuio" }.take(2)
       .joinToString(prefix = ".", separator = "")
    println(language) // "kotlin"
    println(fileExt) // ".kt"

    letters.shuffle()
    letters.reverse(0, 3)
    letters.sortDescending(2, 5)
    println(letters.contentToString()) // [k, o, t, l, i, n]
//sampleEnd
}
```
</div>

Additionally, there are new functions for conversions between `CharArray`/`ByteArray` and `String`:
* `ByteArray.decodeToString()` and `String.encodeToByteArray()`
* `CharArray.concatToString()` and `String.toCharArray()`

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">

```kotlin
fun main() {
//sampleStart
	val str = "kotlin"
    val array = str.toCharArray()
    println(array.concatToString())
//sampleEnd
}
```
</div>

### Functions for string manipulations

The standard library in 1.4 includes a number of improvements in the API for string manipulation:

* `StringBuilder` has useful new extension functions: `set()`, `setRange()`, `deleteAt()`, `deleteRange()`, `appendRange()`,
and others.
    <div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">
    
    ```kotlin
    fun main() {
    //sampleStart
        val sb = StringBuilder("Bye Kotlin 1.3.72")
        sb.deleteRange(0, 3)
        sb.insertRange(0, "Hello", 0 ,5)
        sb.set(15, '4')
        sb.setRange(17, 19, "0")
        print(sb.toString())
    //sampleEnd
    }
    ```
    </div>
    
* Some existing functions of `StringBuilder` are available in the common library. Among them are `append()`, `insert()`,
`substring()`, `setLength()`, and more. 
* New functions `Appendable.appendLine()` and `StringBuilder.appendLine()` have been added to the common library. They
replace the JVM-only `appendln()` functions of these classes.

    <div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">
    
    ```kotlin
    fun main() {
    //sampleStart
        println(buildString {
            appendLine("Hello,")
            appendLine("world")
        })
    //sampleEnd
    }
    ```
    </div>

### Bit operations

New functions for bit manipulations:
* `countOneBits()` 
* `countLeadingZeroBits()`
* `countTrailingZeroBits()`
* `takeHighestOneBit()`
* `takeLowestOneBit()` 
*  `rotateLeft()` and `rotateRight()` (experimental)

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">

```kotlin
fun main() {
//sampleStart
    val number = "1010000".toInt(radix = 2)
    println(number.countOneBits())
    println(number.countTrailingZeroBits())
    println(number.takeHighestOneBit().toString(2))
//sampleEnd
}
```
</div>

### Converting from KType to Java Type

A new extension property `KType.javaType` (currently experimental) in the stdlib helps you obtain a `java.lang.reflect.Type`
from a Kotlin type without using the whole `kotlin-reflect` dependency.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.4">

```kotlin
import kotlin.reflect.javaType
import kotlin.reflect.typeOf

@OptIn(ExperimentalStdlibApi::class)
inline fun <reified T> accessReifiedTypeArg() {
   val kType = typeOf<T>()
   println("Kotlin type: $kType")
   println("Java type: ${kType.javaType}")
}

@OptIn(ExperimentalStdlibApi::class)
fun main() {
   accessReifiedTypeArg<String>()
   // Kotlin type: kotlin.String
   // Java type: class java.lang.String
  
   accessReifiedTypeArg<List<String>>()
   // Kotlin type: kotlin.collections.List<kotlin.String>
   // Java type: java.util.List<java.lang.String>
}
```
</div>

### Proguard configurations for Kotlin reflection

Starting from 1.4, we have embedded Proguard/R8 configurations for Kotlin Reflection in `kotlin-reflect.jar`. With this
in place, most Android projects using R8 or Proguard should work with kotlin-reflect without needing any additional configuration.
You no longer need to copy-paste the Proguard rules for kotlin-reflect internals. But note that you still need to explicitly 
list all the APIs you’re going to reflect on.

### Improving the existing API

* Several functions now work on null receivers, for example:
    * `toBoolean()` on strings
    * `contentEquals()`, `contentHashcode()`, `contentToString()` on arrays

* `NaN`, `NEGATIVE_INFINITY`, and `POSITIVE_INFINITY` in `Double` and `Float` are now defined as `const`, so you can use
them as annotation arguments.

* New constants `SIZE_BITS` and `SIZE_BYTES` in `Double` and `Float` contain the number of bits and bytes used
to represent an instance of the type in binary form.

* The `maxOf()` and `minOf()` top-level functions can accept a variable number of arguments (`vararg`).

### module-info descriptors for stdlib artifacts

Kotlin 1.4 adds `module-info.java` module information to default standard library artifacts. This lets you use them with 
[**jlink** tool](https://docs.oracle.com/en/java/javase/11/tools/jlink.html), which generates custom Java runtime images
containing only the platform modules that are required for your app.
You could already use jlink with Kotlin standard library artifacts, but you had to use separate artifacts to do so – the
ones with the “modular” classifier – and the whole setup wasn’t straightforward.  
In Android, make sure you use the Android Gradle plugin version 3.2 or higher, which can correctly process jar files with module-info.

### Deprecations

#### toShort() and toByte() of Double and Float

We've deprecated the functions `toShort()` and `toByte()` on `Double` and `Float` because they could lead to unexpected results
because of the narrow value range and smaller variable size.

To convert floating-point numbers to `Byte` or `Short`, use the two-step conversion: first, convert them to `Int`, and then 
convert them again to the target type.

#### contains(), indexOf(), and lastIndexOf() on floating-point arrays

We've deprecated the `contains()`, `indexOf()`, and `lastIndexOf()` extension functions of `FloatArray` and `DoubleArray`
because they use the [IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) standard equality, which contradicts the total
order equality in some corner cases. See [this issue](https://youtrack.jetbrains.com/issue/KT-28753) for details.

#### min() and max() collection functions

We've deprecated the `min()` and `max()` collection functions in favor of `minOrNull()` and `maxOrNull()`, which more
properly reflect their behavior – returning `null` on empty collections.
See [this issue](https://youtrack.jetbrains.com/issue/KT-38854) for details. 

### Exclusion of the deprecated experimental coroutines
 
The `kotlin.coroutines.experimental` API was deprecated in favor of kotlin.coroutines in 1.3.0. In 1.4, we’re completing
the deprecation cycle for `kotlin.coroutines.experimental` by removing it from the standard library. For those who still
use it on the JVM, we've provided a compatibility artifact `kotlin-coroutines-experimental-compat.jar` with all the experimental
coroutines APIs. We've published it to Maven, and we include it in the Kotlin distribution alongside the standard library.

## Scripting and REPL

In 1.4, scripting in Kotlin benefits from a number of functional and performance improvements along with other updates.
Here are some of the key changes:

- [New dependencies resolution API](#new-dependencies-resolution-api)
- [New REPL API](#new-repl-api)
- [Compiled scripts cache](#compiled-scripts-cache)
- [Artifacts renaming](#artifacts-renaming)

To help you become more familiar with scripting in Kotlin, we’ve prepared a [project with examples](https://github.com/Kotlin/kotlin-script-examples).
It contains examples of the standard scripts (`*.main.kts`) and examples of uses of the Kotlin Scripting API and custom
script definitions. Please give it a try and share your feedback using our [issue tracker](https://youtrack.jetbrains.com/issues/KT).

### New dependencies resolution API

In 1.4, we’ve introduced a new API for resolving external dependencies (such as Maven artifacts), along with implementations
for it. This API is published in the new artifacts `kotlin-scripting-dependencies` and `kotlin-scripting-dependencies-maven`.
The previous dependency resolution functionality in `kotlin-script-util` library is now deprecated.

### New REPL API

The new experimental REPL API is now a part of the Kotlin Scripting API. There are also several implementations of it in
the published artifacts, and some have advanced functionality, such as code completion. We use this API in the 
[Kotlin Jupyter kernel](https://blog.jetbrains.com/kotlin/2020/05/kotlin-kernel-for-jupyter-notebook-v0-8/)
and now you can try it in your own custom shells and REPLs.

### Compiled scripts cache

The Kotlin Scripting API now provides the ability to implement a compiled scripts cache, significantly speeding up subsequent
executions of unchanged scripts. Our default advanced script implementation `kotlin-main-kts` already has its own cache.

### Artifacts renaming

In order to avoid confusion about artifact names, we’ve renamed `kotlin-scripting-jsr223-embeddable` and `kotlin-scripting-jvm-host-embeddable`
to just `kotlin-scripting-jsr223` and `kotlin-scripting-jvm-host`. These artifacts depend on the `kotlin-compiler-embeddable`
artifact, which shades the bundled third-party libraries to avoid usage conflicts. With this renaming, we’re making the usage of
`kotlin-compiler-embeddable` (which is safer in general) the default for scripting artifacts.
If, for some reason, you need artifacts that depend on the unshaded `kotlin-compiler`, use the artifact versions with the 
`-unshaded` suffix, such as `kotlin-scripting-jsr223-unshaded`. Note that this renaming affects only the scripting artifacts
that are supposed to be used directly; names of other artifacts remain unchanged.



## Kotlin/JS

### New Gradle DSL
The `kotlin.js` Gradle plugin comes with an adjusted Gradle DSL, which provides a number of new configuration options and is more closely aligned to the DSL used by the `kotlin-multiplatform` plugin. Some of the most impactful changes include:

- Explicit toggles for the creation of executable files via `binaries.executable()`. Read more about the executing Kotlin/JS and its environment [here](js-project-setup.html#choosing-execution-environment).
- Configuration of webpack's CSS and style loaders from within the Gradle configuration via `cssSupport`. Read more about using them [here](js-project-setup.html#configuring-css).
- Improved management for npm dependencies, with mandatory version numbers or [semver](https://docs.npmjs.com/misc/semver#versions) version ranges, as well as support for _development_, _peer_, and _optional_ npm dependencies using `devNpm`, `optionalNpm` and `peerNpm`. Read more about dependency management for npm packages directly from Gradle [here](js-project-setup.html#npm-dependencies).
- Stronger integrations for [Dukat](https://github.com/Kotlin/dukat), the generator for Kotlin external declarations. External declarations can now be generated at build time, or can be manually generated via a Gradle task. Read more about how to use the integration [here](js-modules.html#automatic-generation-of-external-declarations-with-dukat).

### New JS IR back-end
The [IR back-end for Kotlin/JS](js-ir-compiler.html), which is currently has [Alpha](evolution/components-stability.html) stability, provides some new functionality specific to the Kotlin/JS target which focused around the generated code size through dead code elimination, and improved interoperation with JavaScript and TypeScript, among others.

To enable the Kotlin/JS IR back-end, set the key `kotlin.js.compiler=ir` in your `gradle.properties`, or pass the `IR` compiler type to the `js` function of your Gradle build script:


<!--suppress ALL -->
<div class="sample" markdown="1" mode="groovy" theme="idea">

```groovy
kotlin {
    js(IR) { // or: LEGACY, BOTH
        // . . .
    }
    binaries.executable()
}
```

</div>

For more detailed information about how to configure the Kotlin/JS IR compiler back-end, check out the [documentation](js-ir-compiler.html).

With the new [`@JsExport`](js-to-kotlin-interop.html#jsexport-annotation) annotation and the ability to **[generate TypeScript definitions](js-ir-compiler.html#preview-generation-of-typescript-declaration-files-dts) from Kotlin code**, the Kotlin/JS IR compiler back-end improves JavaScript & TypeScript interoperability. This also makes it easier to integrate Kotlin/JS code with existing tooling, to create **hybrid applications** and leverage code-sharing functionality in multiplatform projects.

Learn more about the available features in the Kotlin/JS IR compiler back-end in the [documentation](js-ir-compiler.html).