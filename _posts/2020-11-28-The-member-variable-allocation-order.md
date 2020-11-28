---
layout: post
title: "The member variable: allocation order"
---

If you can answer the below questions correctly, you do not have to read this article.

In java, during the class instantiation process, 

1. which is called first? A) constructor B) instance variable initializers
2. which is finished first? A) constructor B) instance variable initializers

The answers are 🅰️ for question 1, 🅱️ for question 2.

## Why do we care

Many people "just know" how Java member variables are allocated. They know how it works, and the JVM works as they expected. However, when asked without a machine in front of them, **surprisingly many of them get confused**. In fact, the primary purpose of writing this article is to prevent future confusion and provide a reliable reference for myself. 

Why is it so confusing? This is because **a rarely witnessed transition of the control flow happens during member variable value allocation**. When a function is invoked, java runs it from start to end without changing the control flow. (Aside from system transition like thread and process.) We do not usually see a sudden call of other methods unless we explicitly call other methods. However, during class instantiation, this happens. Java calls constructor, but in the middle of the constructor body, java calls an instance allocator. And if the allocation is done, java continues executing the constructor body. 

The spec is illustrated at the [oracle docs](https://docs.oracle.com/javase/specs/jls/se7/html/jls-12.html#jls-12.5). Briefly speaking, the orders are as below:

1. push parameters
2. if the constructor begins with explicit `this`() execute it.
3. if the constructor does not begin with explicit `this()`, then call `super()`.
4. then **call instance initializers**.
5. **execute the rest of the constructor body**. 

We will see each step in the codes in the below section.

## Example

```kotlin
open class Parent {
    constructor() {
        println("parent constructor")
        printMember()
    }

    open fun printMember() = Unit
}

class Child: Parent {
    var member: Boolean = true

    constructor() {
        /* instance initializer called here */
        println("primary constructor")
        println("member = $member")
    }

    constructor(number: Number): this() {
        println("constructor with a parameter")
    }

    override fun printMember() = println(member)
}
```

Before reading on, please carefully examine the above code and imagine what will happen when we call `Child(1)` in Kotlin or `new Child(1)` in java. There will be 5 lines printed: 3 lines from `println` and 2 lines from 2 calls to the `printMember` calls. According to the above Java SE 12.5, what will be printed in which order?
Let's follow the specification:

First, we enter `constructor(number: Number)`. We ignore step 1 throughout the above code since we will not be using the value anywhere. We see explicit constructor invocation, `this()`. So `Child#this()` will be executed. 
Here, we do not see explicit `this()` nor `super()`, so we move onto step 3. Here, we call `super`. 

We can keep going (because class `Parent` is a child of `Object`), but we know that it will not print out anything. So, the parent's constructor body will be executed. We see our first printed line here. We will see `parent constructor` and `member = false` printed out. Why is the `member` a `false`? This is because the `member` is not initialized yet, and the default value for the uninitialized Boolean is a false. 

So we come back to `Child#constructor()`. Now step 4 is executed. Instance initializer is called here, and the `member` is now `true`. Then, we see two lines printed out. `primary constructor`, and `member = true`.

Then finally, we come back to the first constructor invoked. `Child#constructor(Number)`. Following step 5, we see a line `constructor with a parameter printed` out. As a result, you will see the following.

```
parent constructor
member = false
primary constructor
member = true
constructor with a parameter
```

This is the call stack.

```
Child#constructor(Number)
	Child#constructor()
		Parent#constructor()
			Object#constructor()
			Object#body()
         Parent#body() 
         // println("parent constructor")
         // printMember()
    Child#body() // constructor without parameter's body
    // println("primary constructor")
    // printMember()
Child#body() // constructor with parameter's body
// println("constructor with a parameter")
```

## Conclusion

Although many people get confused when asked without a computer, many developers know this instinctively. People just know that assigning a value in a constructor will naturally override the value assigned by the instance initializer. However, I believe knowing this by experience and by concrete rule & spec could show some difference on some occasions. I hope this can be helpful to you or the future myself 👍.