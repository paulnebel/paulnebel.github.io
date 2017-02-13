---
layout: post
title: "A Brief Introduction to Functional Programming"
meta: "A brief introduction to Functional Programming. Start your journey towards easy to read, reliable code."
categories: Node JavaScript
slug: "brief-introduction-functional-programming"
---
# A Brief Introduction to Functional Programming
This article was first published on 20th January 2017 on the [BCS website][article]{:target="_blank"}.

### Background
As professional developers it is our job to produce efficient, reliable code within a reasonable amount of time. Of course, the definition of 'a reasonable amount of time' will vary from project to project, but it usually means 'as fast as possible'. How efficient your code needs to be depends upon it's intention. The efficiency required of a real-time service like a financial information feed to a trading desk will be orders of magnitude higher than that required when updating your account password. What generally remains common across all project types is the concept of reliability: reliable code is code that continues to work no matter what it is doing and _regardless of how many changes are made to it_.

Even small code-bases can be hard to maintain or enhance if you don't construct them well in the first instance. There can't be many of us who haven't returned to our own code 6 months after we first wrote it and made a small change only to find that it is now broken for no obvious reason.  As for working on someone else's code, well, don't get me started. The larger the codebase, the more likely this is to happen. How, then, do we avoid wasting time debugging brittle code?

There are many techniques that could be used but the one I want to introduce here is **functional programming (FP)**. This is a way of coding that aims to produce modular, robust and expressive code that is _easy to reason about_. Like many such techniques it is not a new idea (Haskell, for example, is a pure functional programming language that was defined in 1990). Fortunately it is not difficult to do and can be used in both object-oriented (OO) and non-OO code. It does, however, require us to think in a different way and to give up some old habits.

### Definition
I will be using JavaScript for the examples in this post. This is because JavaScript...
- is the language I use every day
- has great features that suit FP very well (e.g. functions as objects, meaning that functions can be passed as parameters to other functions)
- has features with really bad side-effects (e.g. [global scope][global-scope]{:target="_blank"}) that can be mitigated by FP.

So what is FP?
- **Functional programming is a technique that uses functions to _abstract flows and operations on data_ so as to _avoid unwanted side-effects_ caused by _mutations of state_.**

#### Declarative vs Imperative
Consider a humble **for** loop:
{% highlight bash %}
let a = [1, 2, 3];
for (i = 0; i < a.length; i++) {
    console.log(a[i]);
    // this will probably output
    // 1
    // 2
    // 3
    // on the other hand, it may not
}
{% endhighlight %}

The style of programming where we use `for` loops is _imperative_, meaning that it tells the computer in great detail exactly _how_ to perform a task (in this case, outputting a number to the console). This has the side-effect of greatly reducing re-use, because imperative programming is targeted at solving a particular problem efficiently. This means that it runs at a far lower level of abstraction than functional code. The lower the level of abstraction, the lower the probability of reuse, and the greater the complexity and likelihood of errors.

FP, by contrast, uses a _declarative_ programming style, meaning that it separates the program description from it's evaluation. It regards programs as the evaluation of independent, pure functions. This raises the level of abstraction and allows us to concentrate on what is being done, and not on how it is being done. A much better FP option is to use [`Array.forEach()`][array-for-each]{:target="_blank"} to manage the looping, leaving us to concentrate on applying the appropriate behaviour to each element:
{% highlight bash %}
[1, 2, 3].forEach(function(element) {
    console.log(element);
    // this will always output
    // 1
    // 2
    // 3
});
{% endhighlight %}

A useful side-effect of using `Array.forEach()` is that we have produced a function which is _idempotent_ (that is to say, a function that is guaranteed to produce the same result for a given input no matter how many times it is called). This greatly increases the stability and predictability of the code.

We can even use ES6 [arrow functions][arrow-functions]{:target="_blank"} (also known as _lambda functions_) to make this definition more succinct:
{% highlight bash %}
[1, 2, 3].forEach(element => console.log(element));
{% endhighlight %}

If you have ever used `Array.forEach()` or it's siblings ([`Array.map()`][array-map]{:target="_blank"} and [`Array.reduce()`][array-reduce]{:target="_blank"}) then you're already using FP.

As with most things in programming, there may be [good reasons][foreach-vs-for]{:target="_blank"} for using a `for` loop and not `Array.forEach()` but you can now make a reasoned choice between these two options.

#### Pure vs Impure
This `for` loop used in our previous may seem innocuous enough, but if you look carefully you'll notice that I didn't explicitly define `i` (with a `var` statement or with the preferable `let` statement in ES6) which means that we actually have no way of knowing exactly what this loop will do - it all depends where we defined `i`.

If we forgot to define it at all then JavaScript assumes it has global scope which means that its value can be changed _at any time by any other part of the code base_. Of course, we could use _[strict mode][strict-mode]{:target="_blank"}_ or a linter to catch  this kind of problem but we shouldn't be relying on the environment to mitigate the effects of poor programming.

Not only that, but the array over which it is iterating is not local to the loop's scope, meaning that it, too is vulnerable to unwanted change. In short, our code example is _impure_.  The `Array.forEach()` method, on the other hand, is an example of a _pure_ function, meaning that:
- it depends only on the input provided and not on any externally defined or hidden state that may be changed at any time during its evaluation or between evaluations.
- it doesn't impose changes beyond its own scope such as modifying an input parameter passed by reference

#### Immutable vs mutable
**Immutable** data is data which can't be altered once it has been created. In common with many programming languages, JavaScript defines primitave types (such as _String_ or _Number_) which are immutable. However most programs contain myriad data that is not immutable (e.g. objects and arrays). When such _mutable_ types are passed into functions they can cause unwanted side-effects by changing the original data.

Take the example of sorting an array:
{% highlight bash %}
let sortDesc = arr => arr.sort((x, y) => x - y);
{% endhighlight %}

Although this may look safe, it isn't.  The reason is that `Array.sort()` sorts the array in place (i.e. you provide an array and it returns the _same_ array in sorted order).  To be an immutable function `Array.sort()` would have to return a new array in sorted order and leave the original array untouched.

FP is based on the concepts of:
- Declarative programming
- Pure functions
- Immutability

### A Simple Example
Let's take a simple imperative program and convert it to a functional program to test the contention that it is easier to reason about the FP version. Consider an operation that takes a list of strings and normalises them (i.e. starts each string with a capital letter and removes any underscores that may be present). A declarative implementation could be as follows:
{% highlight bash %}
let words = [ 'functional Programming', 'imperative_programming', 'Immutable object', 'Pure Function'];
let result = [];
// iterate over all words
for (let i = 0; i < words.length; i++) {
    let current = words[i];
    // replace underscores with spaces
    let splitArray = current.replace(/_/, ' ').split(' ');
    if (current !== undefined && current !== null) {
        for (let j = 0; j < splitArray.length; j++) {
            let word = splitArray[j];
            // make the first letter of each word uppercase
            word = word.charAt(0).toUpperCase() + word.slice(1);
            splitArray[j] = word;
        }
        result.push(splitArray.join(' '));
    }
}
{% endhighlight %}

Does this look familiar? Now let us redefine this using FP. We will use `Array.filter()`, which is a function that operates over an array of values and applies the supplied test to them, returning a new array containing the elements of the original array for which the test returns `true`. The code now becomes:
{% highlight bash %}
let isValid = v => v !== undefined && v != null;
let normalize = s => s.replace(/_/, ' ');
let capitalizeWord = w => w.charAt(0).toUpperCase() + w.slice(1);
let capitalizeSentence = (currString, arr) => {
  let capString = currString.concat(capitalizeWord(arr[0]));
  let rest = arr.slice(1);
  if (rest.length > 0) {
    return capitalizeSentence(capString.concat(" "), rest);
  } else {
    return capString;
  }
}

let words = ['functional Programming', 'imperative_programming', 'Immutable object', 'Pure Function'];
let result = words.filter(isValid)
    .map(normalize)
    .map(s => capitalizeSentence("", s.split(' ')))
{% endhighlight %}

The FP solution has re-factored some of the declarative code into separate functions. The `capitalizeWord`, `isValid` and `normalize` functions could easily have been used in the declarative version (but, let's be honest, probably wouldn't have been). The 'capitalizeSentence` function introduces the concept of _recursion_ (in which a function calls itself until it meets a specific condition) to avoid explicitly looping over an array. Recursion is a very useful FP technique. All of these functions are pure and can be re-used. It is now much easier to see that the sequence is to check that a sentence is valid, normalize it by removing underscores and then capitalize each word in the sentence.

### Conclusion
This post barely scratches the surface of functional programming techniques.  There are plenty of other resources available on the web and in print that will take you deeper into the world of FP.  What I have done is to define FP as being a programming technique that embodies the principles of _pure, declarative, immutable_ coding. These principles are desireable because they encourage the creation of code which is _free of unwanted side-effects_ and _easy to reason about_, leading to _greater reliability_.  I hope that this will inspire you to look further and to adopt FP (where appropriate!) in your own coding practise.

   [article]: <http://www.bcs.org/content/conBlogPost/2637>
   [global-scope]: <https://toddmotto.com/everything-you-wanted-to-know-about-javascript-scope/#what-is-global-scope>
   [array-for-each]: <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach>
   [arrow-functions]: <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions>
   [array-map]: <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/map>
   [array-reduce]: <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce>
   [foreach-vs-for]: <https://coderwall.com/p/kvzbpa/don-t-use-array-foreach-use-for-instead>
   [strict-mode]: <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode>
