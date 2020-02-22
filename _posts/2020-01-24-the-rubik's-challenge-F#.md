---
layout: post-sidebar
date: 2020-01-24
title: "The Rubik's challenge - F#"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 15
feature_image: feature-fsharp
show_related_posts: true
square_related: recommend-coding
---

<br>
So here we are with the third Rubik's cube. If you don't know what is this about, please check this [link]({% post_url 2020-01-21-the-rubik's-challenge %}).

Today it's time to play with F#. I've been heard things about F# for long time ago, It's in the top ten well paid programming languages for developers (according to the StackOverlow Developers survery 2019) so I'm going to try today to investigate what it's makes me so special.

According to its documentation <<F# is a functional programming language that makes it easy to write correct and maintainable code.>>. And yes, that's true, but "functional programming language" is the part that will makes me spend more time as I'm coming from a POO language.

Functional programming has become very popular, thinking in build the code using only pure functions (no state, no shared data) will force you to model your application flow using functions, it is a programming paradigm for me. Let's see how friendly it is.

## Validating Lenght
Let's start with the function that validates the lenght of the input paramter:

{% highlight fsharp %}
let calculateLength ninoInput : string =
    // function body
    if String.length ninoInput = 9  then "ok"
    else failwith "The lenght of the nino is invalid" 
{% endhighlight %}

The first think that I noticed is that in functional programming "calculateLength" is not a good name, as I'm modeling functions that does not share states, I should name it as "isValidLength" or something similar. That's name sounds more appropiate for a function and more close to the functional programming style so I changed it into the code.

In the last couple os days with Ruby and Rust, I get used to snake_case for the function names, however the F# linter starts to give me some clues about the naming convention that F# expect from us:

{% highlight fsharp %}
Consider changing `calculate_length` to remove any underscores.
{% endhighlight %}

Coming back to CamelCase (either for functions and variables) give me weird feelings.


When I started to type the function I started to guess how to put the return type and the answer is, it's not needed (unless you want to return a different type that the one that you are recieving).


In this case, our function could be either write like:

{% highlight fsharp %}
let calculateLength ninoInput : string

// or we can write the function as
let calculateLength (ninoInput : string) : string 
{% endhighlight %}

In both cases is string -> string, so we are recieving an string and returning an string.

As the experiment rules stated, this first function must raise an exception if the parameter length is incorrect, that point, in F# is also quite easy:

{% highlight fsharp %}
failwith "The lenght of the nino is invalid" 
{% endhighlight %}

But beign more specific, we could raise an ArgumentException:

{% highlight fsharp %}
invalidArg "inputNino" "The lenght of the nino is invalid" 
{% endhighlight %}

I tried to replicate the same behaviour than Ruby or Rust raising the exception if the lenght is incorrect and doing anything if the lenght is correct, but introducing a simple if statements give me more clues about why F# and functional programming is special:

{% highlight plaintext %}
// From F# docummentation
Unlike in other languages, the if...then...else construct is an expression, not a statement. 
That means that it produces a value, which is the value of the last expression in the branch that executes. 
The types of the values produced in each branch must match. [...] Therefore, if the type of the then branch 
is any type other than unit, there must be an else branch with the same return type.
{% endhighlight %}

The key is "there must be an else branch with the same return type" so you will need to put an else branch unless you are using unit type.
It makes sense but takes a while to get used to it.

## Validate Prefix

Our second function is the one that validates the prefix:

{% highlight fsharp %}
let isValidPrefix (ninoInput : string) : bool =    
    let prefix = ninoInput.[0..1]
    let invalidPrefixes = [| "D"; "F"; "I"; "Q"; "U"; "V"; "BG"; "GB"; "NK"; "KN"; "TN"; "NT"; "ZZ" |]
    Array.exists (fun elem -> prefix.Contains(elem.ToString())) invalidPrefixes
{% endhighlight %}

Something that I forgot to comment before is that, in F# as some others functional languages, does not exists a return function, so the return of the function is the last statement of your function. In this case we are returning a boolean type.

Doing a substring is something that you can easyly do using array slice syntax and declaring an array is like any other language, but you must enclose the code between [| and |] and separate the elements using semicolons.

My first idea was that iterating the array using F# functional language should be quite similar to iterate and Array in Java using the Streams API. The function that I found really useful for this was Array.exist:

{% highlight fsharp %}
// Signature:
Array.exists : ('T -> bool) -> 'T [] -> bool
{% endhighlight %}

Nice, clean and powerful, isn't it?

## Checking numeric string

This is my code:

{% highlight fsharp %}
let validateNumbers (ninoInput : string) : bool =   
    let numericPart = ninoInput.ToString().[2..7]
    numericPart |> Seq.forall Char.IsDigit
{% endhighlight %}

Yes, the last part wasn't strightforward to understand, at least for me. I thought that there was be a native function to check if the whole string is numeric but It wasn't.

So in this case the pipe operator (pipe forward) passes the result of the left side to the function on the right side; and the Seq.forall which is a sequencer:

{% highlight fsharp %}
// Signature:
Seq.forall : ('T -> bool) -> seq<'T> -> bool
{% endhighlight %}


## Ending the experiment

I realised that moving from an imperative language to a new declarative programming language like F# produces a clean code and an application flow moving through pure functions which is nice to follow and debug, so even if you are comming from other fully-functional or semi-functional programming languages Scala, worth the effort to give an opportunity to F#. 

From my point of view F# fits it mantra: it produces correct and maintenable code...however functional programming is still a niche approach, I'm sure that functional programming has a good future and F# is the perfect candidate supported by a big organization like Microsoft.

You have the full code [here](https://gist.github.com/jsalcedo1987/eb21cd0d7e2a62219312500535b9fec6)