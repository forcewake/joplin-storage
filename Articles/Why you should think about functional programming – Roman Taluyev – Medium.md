Why you should think about functional programming – Roman Taluyev – Medium

[![Go to the profile of Roman Taluyev](../_resources/ddb44b6c46fb4055837fffe34a620757.png)](https://medium.com/@taluyev?source=post_header_lockup)

Why you should think about functional programming? Let’s answer the following questions:

*   Are your projects always carried out within the specific time frame?
*   Have users ever had any complaints?
*   Project support has never been time-consuming?
*   New functionality always successfully fit into the existing architecture?

If the answers to all the above questions are positive, then you do not need to change anything, your team is a rare example of harmony staff, methodology and tools. Otherwise, you should be open to new approaches to solving your problems, including a critical look at the technical tools and programming languages used.

### First Glimpse

As you know, programmers are extremely creative people but at the same time, they tend to follow some trends in choosing their programming language. Among the vast abundance of languages, functional languages are increasingly overgrown with fans and more and more confidently make their way to most companies around the world. The use of functional languages can greatly increase the productivity and quality of programmers. Naturally, it depends on a combination of tasks, language and programming skills. In this case, a programmer simply describes what he wants, instead of listing the sequence of actions necessary to obtain the result. Thus, the developer focuses on the high-level “what is required”, instead of getting stuck on the low-level “how to do”.

Functional programming is the style of writing programs through the compilation of a set of functions. Its basic principle is to wrap almost everything in a function, write many small reusable functions, and then simply call them one after another. In addition, the structure of those functions must follow certain rules and solve some issues.

So, if any task can be solved by combining calls of several functions, then:

1.  How to implement the if-else conditions?
2.  How to solve the Null Exception tricks?
3.  How to make sure that the function is really “reusable” and can be used anywhere?
4.  How to make sure that the data transmitted to our functions have been changed and can be used in other places?
5.  If a function takes several values, but when chaining you can transfer only one function, then how do we make this function part of a chain?

To solve all these problems, functional languages provide tools and solutions from mathematics, such as monads, functors, etc.

The benefits of functional programming have long been recognized by the general public. The successful development of software often comes down to the maximum simplification of existing mechanisms that allow new applications to adapt to the requirements of modern users. And at the same time, we have to hurry, managing in a short time to present products to customers with unlimited possibilities. It is much easier to carry out the mission when the developed applications can be divided into several pure functions, which are not difficult to verify. In such algorithms, there are no tricky side effects and abstract formulations designed for results on a global scale.

So why did the experts lose sight of functional programming for so long? And why is it becoming so common today?

### Where is Functional Programming used in Real World?

Since functional programming is primarily an approach to writing code, you can use its principles in any language. However, there are languages specifically sharpened by the functional approach. More modern functional languages, such as Elm and Elixir, according to GitHub and Stack Overflow, are gradually and surely gaining popularity. The growing popularity of JavaScript has also led to increased interest in functional programming concepts for use in this language. In addition, experienced developers in functional programming subsequently started working on SPA frameworks, and, as a result, we have Redux, React, MobX and other libraries used by millions of people.

Real life examples:

*   Apache Spark
*   Scalding (by Twitter)
*   Apache Kafka
*   Finagle (by Twitter)
*   Akka
*   Autocad
*   emacs (LISP)

So what is functional programming, where does such a boom come from and why is it worth considering studying it? Let’s figure it out.

### About Functional Programming

A few years ago, only a few experts had an idea about functional programming, but over the past three years, almost every code base of large applications have been actively using ideas taken from the world of functional programming. And there are objective reasons for this:

*   functional programming allows you to write more concise and predictable code;
*   it is easier to test (although learning from scratch is not easy).

### Key features of software development using Functional Programming

*   pure functions and their composition;

Pure function is very simple. It must always return the same result. With the same Х and Y values, we will always get the same result of the function. Predictability plays an important part during the programme work in functional programming.

*   avoiding shared state, mutable data, and side effects;

An immutable object is an object whose state cannot be modified after it is created. Immutable objects are more thread-safe than mutable objects. If the function does not work predictably — it will lead to undesirable side effects. In imperative languages function in the process of implementation can read and modify the values of global variables and perform input/output. Therefore, if we call the same function twice with the same argument, it may happen that we get two different results. Such a function is called a side effect.  
Functional programming helps us to write thread-safe code.

*   prevalence of declarative, rather than an imperative approach.

### Function Compositions

Function composition — is a functional programming approach which involves calling one functions as arguments of others to create a complex composite of simpler functions. This layout technique allows you to take two or more simple functions and combine them into one more complex function that performs sub-functions in a logical order with any data. To get this result, you put one function inside another and perform external function operations on the result of an internal function until you get a result. And the result may be different, depending on the order in which the functions are applied.

As a result, with a different logical order of calling pure functions and the same value of the argument, we will get more complex functionality that gives us the desired result and makes it predictable.

### The Advantages of Functional Programming

Functional programming helps make the code more clear, predictable and easy to read. Using the functional programming principles helps to get rid of unnecessary abstractions with unpredictable behaviour, therefore, to make the program more predictable and reduce the number of possible errors.

In functional languages, functions can be passed to other functions as an argument or returned as a result. Functions that take functional arguments are called higher-order functions or functionals. The most popular functionality is the map. map, which applies some function to all elements of the list, forming another list from the results.

In pure functional programming, the assignment operator is absent, objects cannot be changed and destroyed, one can only create new ones by decoding and synthesizing existing ones. The garbage collector built in language will take care of unnecessary objects. Due to this, in a pure functional language, all functions are free from side effects. However, this does not prevent this language from imitating some useful imperative properties, such as exceptions and mutable arrays. Of course, there are special techniques for this purpose.

### The Disadvantages of Functional Programming

*   One problem with functional programming is that it differs from what you already

know. You have to re-learn a lot of what you already know in an imperative setting! Many experienced developers who are already comfortable with what they already know are not going to solve problems in a different way. It requires time and effort to think differently.

*   It is easy to write pure functions, but combining them into a complete application is

where things get hard.

*   A big problem is with predictable performance. Using only immutable values and

recursion can potentially lead to performance problems, including RAM use and speed, thus most functional languages are not particularly good choices for soft or hard real-time systems or embedded computing.

*   Functional programming tends to write code in too abstract form when the

programmer himself no longer understands what he has written after a while.

*   Pure functions and I/O don’t really mix.

### Conclusion

If it seems to you and your developers’ team that your coding style does not give you the required level of management of business domain complexity, littering your code with unnecessary syntactic garbage, in which meaning is lost of time, try functional programming. So, in areas related to a large number of calculations or data transformations, technical programming, parallel / asynchronous, you can get significant benefits by immersing yourself into functional programming.

Writing code in a functional language gives you the opportunity to look at the issue from another aspect, where the development of your solution may be more effective. And it will simply increase the number of ways to express your ideas.

Developing your solution using functional programming paradigm you will speed up all development process more and more according to becoming much more experienced. It looks like you drive your car on higher gear.

Functional programming can change your code writing style for the better. But it’s quite difficult and time-consuming to master it, and many posts and tutorials do not consider details (like monads, applicative functors, etc.) and do not provide practical examples that would help beginners to use powerful functional programming techniques every day. But you can attract already experienced programmers to the team which would save your time, effort and money.

If you do it right, the result will be more clear, concise and readable code. Isn’t that what we all want?

[quasarbyte.com](https://quasarbyte.com/?utm_source=medium&utm_medium=article&utm_campaign=functional_programming&utm_content=why_you_should_think_about_functional_programming)

Roman Taluyev,

Software Developer