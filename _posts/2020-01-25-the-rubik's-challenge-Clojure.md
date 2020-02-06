---
layout: post-sidebar
date: 2020-01-25
title: "The Rubik's challenge - Clojure"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 6
feature_image: feature-ruby
show_related_posts: true
square_related: recommend-spain
---

<br>
So here we are with the last Rubik's cube. If you don't know what is this about, please check this [link]({% post_url 2020-01-21-the-rubik's-challenge %}).

Today it's time to play with Clojure. I suppose that there should be one (or many) good reasons to make this programming to be on the most loved languages, one of the top paid languages but also one of the less demanded by companies (according to 2019 StackOverflow survey).

According to that survey, is on top of languages like Dart, Go or C++ so there must be something powerful inside Clojure.
My opinion is that Clojure has two key attributes. The first one is that Clojure is a functional language and this programming paradigm is getting more and more attention in the community and the seconde one is that Clojure is hosted in the JVM so it can consume all the Java libraries. We just need to import and reference them in our Clojure code. Sounds like the perfect combination

## Validating Lenght

Let's start with the function that validates the lenght:

{% highlight clojure %}
 (defn isValidLenght? [inputNino]
  (if (= (.length inputNino) 9) true (throw (Exception. "The lenght of the nin is invalid"))))
{% endhighlight %}

Clojure syntax is so far the most different compared with the previous programming langugages of this experiment.
The extensive use of parentheses () is something that I had to get used to.

The functions are declared using defn and the input parameters are between the brackets [].

As the other functional language of this test (F#) in Clojure If statments are also special. In our F# post we mention that if we put an `if` there must be an `else` branch, in Clojure a similar concept is present. I'm going to use this example extracted from it's documentation:

{% highlight clojure %}
(defn is-small? [number]
  (if (< number 100) "yes" "no"))
{% endhighlight %}

Something must happen if the condition is not true.

Following the experiment rules, this function needs to raise an exception if the parameter lenght is incorrect.
That in Clojure is easy peasy:
{% highlight clojure %}
(throw (Exception. "The lenght of the nin is invalid"))
{% endhighlight %}

I still need to investigate how to make custom exceptions and raise it from clojure.

As fully functional programming language Clojure doesn’t have an explicit return statement. It will return the result of the last form evaluated in a function.

## Validate Prefix

Here is the code:

{% highlight clojure %}
 (defn isValidPrefix? [inputNino]
    (def prefix (subs inputNino 0 2))
    (println prefix)
    (def invalidPrefixes ["D" "F" "I" "Q" "U" "V" "BG" "GB" "NK" "KN" "TN" "NT" "ZZ"])
    (some (partial = prefix) invalidPrefixes)
 )
{% endhighlight %}

We've solved the prefix validation in other languages with a for loop iterating over the collection / array of invalid prefixes. I've tried to do the same using clojure but to be honest I haven't found a clear way to do it. 

I do not wanted to use Java libraries for it so keeping in mind the time constraint I choose and alternative: Use "some" instruction to iterate the array and compare the prefixes.

{% highlight clojure %}
(some pred coll)

Returns the first logical true value of (pred x) for any x in coll,
else nil.  One common idiom is to use a set as pred, for example
this will return :fred if :fred is in the sequence, otherwise nil:
(some #{:fred} coll)
{% endhighlight %}

This was extracted from [ClojureDocs](https://clojuredocs.org/) that probably will be one of your most visited pages if you start with Clojure.
Because it's not only contains the documentation itself, it also contains many useful examples of each function.

## Checking numeric string

Let's see the code:

{% highlight clojure %}
 (defn isNumeric? [inputNino]
    (def numericPart (subs inputNino 2 8))
    (println numericPart)
    (try (Integer/parseInt numericPart)
    (catch Exception e nil))
 )
{% endhighlight %}

Self explanatory, right? basically I've try to parse the substring to integer and capture the exception otherwise return nil.

## Ending the experiment

The interaction between Clojure and existing Java libraries is a huge win-win for Clojure. It also has an enthusiastic community that is expanding the Clojure ecosystem with new features and a frontend framework based on Clojure, ClojureJs.

My experience is that Clojure as other functional programming languages, produces flat codebases: simple data-in & data-out functions. That's very nice for developers as you won't have to navigate throught nested clases but definitely clojure syntax is a challenge, I've never tried before a language with sooo many parentheses.

From all the languages of the experiment, is the one with less lines of code; that's the reason why this post is also shortest than the other ones.

You have the full code [here](https://gist.github.com/jsalcedo1987/e06775fdaf5a252f0fb8a65c639b14bb)