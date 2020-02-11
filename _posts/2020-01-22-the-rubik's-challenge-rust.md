---
layout: post-sidebar
date: 2020-01-22
title: "The Rubik's challenge - Rust"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 12
feature_image: feature-rust
show_related_posts: true
square_related: recommend-coding
---

<br>
So here we are with the first Rubik's cube. If you don't know what is this about, please check this [link]({% post_url 2020-01-21-the-rubik's-challenge %}).

Rust is on the top of the list of most loved programming languages in the StackOverflow 2019 survey. I had read that Rust is super-fast, super-friendly, and super-straighforward and lately there are more important companies that are choosing Rust in top of GO or any other programming languages [link].

Before to start to read, please keep in mind that this is not a tutorial, the code exposed here could or could not be optimal solution.

## First steps

The main program:

{% highlight rust %}
use std::io;

fn main() {
  println!("Welcome to Nin validation");
  let mut input_nino = String::new();

  io::stdin().read_line(&mut input_nino)
    .expect("Failed to read line");

  println!("Your input is: {}", input_nino);

  // TODO: We will call to our new validations functions here

}
{% endhighlight %}


One of the first things I faced in Rust is that it does not allow to leave un-used imports in your code. I'm used to get warnings in the IDE or during the compilation but Rust is definitely more strict on that. 

All the functions starts with `fn`; Our main function is a void function but I will show later how to return values. 

Other thing that I faced with Rust is that camelCase naming convention is not permitted, it uses snake_case for either functions and variables. 
Personally I find snake_case more readeable

{% highlight rust %}
thisIsAnExampleOfUnreadeableName  // Horrible CamelCase

this_is_an_example_of_readeable_name // Looks better, right?
{% endhighlight %}

Coming back to Rust, the warnings goes to another level:

{% highlight shell %}
// warning: unnecessary parentheses around `if` condition
{% endhighlight %}

The good point is that it will take few seconds to fix those things as soon as you write the code, but many minutes to fix it in thousends of lines if you do it afterward, facing with those warnings in each compilation help you to interiorize best practises.

Playing with the variables in Rust I also faced with something interesting:

{% highlight rust %}
let mut input_nino = String::new();
{% endhighlight %}

What the hell is mut? That's how we define mutable variables in Rust ( https://doc.rust-lang.org/1.0.0/book/mutability.html ) and mutability is just the hability to change the value of the variable, so by default all the variables are inmutable.

This gives you an error
{% highlight rust %}
let x = 5;
x = 6; // error!
{% endhighlight %}

but this works with no issues:
{% highlight rust %}
let mut x = 5;
x = 6; // no problem!
{% endhighlight %}

bit weird at the begining but quite useful in the long term.


## Validating Lenght

This is a simple function, it calculates the lenght of the parameter and return OK or KO. Following the challenge rules, the method also needs to raise an exception. 

{% highlight rust %}
fn validate_length(input_nino: &String) -> Result<bool, String>{

if input_nino.len() == 9 {
  print!("{} : lenght ok", input_nino);
    Ok(true) // This OK
  } else {
    print!("{} : lenght ko", input_nino);
    return Err("Invalid Lenght!!".to_string());
  }
}
{% endhighlight %}

The Rust syntax makes really intuitve to understand what the function is recieving and the return of the function.
We could declare the function as follow if we like to return just a boolean:

{% highlight rust %}
fn validate_length(input_nino: &String) -> bool {
{% endhighlight %}

However, as I said in the introductory post, I also would like to investigate how to raise and manage exceptions. So there we have to introduce the Result type. Which is basically an enum:

{% highlight rust %}
enum Result<T, E> {
  Ok(T),
  Err(E),
}
{% endhighlight %}

So in our case, for a sucess response we could return just "Ok(true)". This is which Rust call a "Recoverable" error. There is another type of errors called "Unrecoverable" errors; The unrecoverable errors will break your program flow.

This is how the code looks like using "unrecoverable" errors:

{% highlight rust %}
fn validate_length (input_nino: &String) -> bool {

if input_nino.len() == 9 {
  print!("{} : lenght ok", input_nino);
  return true;
} else {
  print!("{} : incorrect lenght", input_nino);
    panic!("crash and burn"); // This exception will break the program
  }
}
{% endhighlight %}

The other interesting bit in the header is about the "&String". Why are we doing...:

{% highlight rust %}
fn validate_length(input_nino: &String) -> Result<bool, String>{
{% endhighlight %}

Instead of (note the diference between &String and String):
{% highlight rust %}
fn validate_length(input_nino: String) -> Result<bool, String>{
{% endhighlight %}

That's another particularity of Rust. Rust by default moves the varible and not a copy of it, so if you want to recieve a copy of the variable you need to add the &symbol, otherwise you will recieve this error:

{% highlight rust %}
6  |     let input_nino = String::from("TT123456C");
|         ---------- move occurs because `input_nino` has type `std::string::String`, which does not implement the `Copy` trait
...
15 |     if validate_length(input_nino).is_err() {
|                        ---------- value moved here
...
19 |     if !validate_prefix(input_nino) {
|                         ^^^^^^^^^^ value used here after move
{% endhighlight %}

Because if you already moved your variable to another function, you cannot reuse it.


## Validate Prefix

This function checks if the prexif is valid or not:

{% highlight rust %}
fn validate_prefix(input_nino: &String) -> bool{

  let prefix: String = input_nino.chars().skip(0).take(2).collect();

  let forbidden_letters = vec!["D", "F", "I", "Q", "U", "V", "BG", "GB", "NK", "KN", "TN", "NT", "ZZ"];

  for forbidden_letter in forbidden_letters.iter() {
    if prefix.contains(forbidden_letter) {
      print!("{} this letter is in the string", forbidden_letter);
      return false;
    }
  }
  return true;
}
{% endhighlight %}

I've use iterators because it allow us to forgot about indexes, however it could be implemented using a standard for loop and access to the array using the loop index:

{% highlight rust %}
for n in 0..forbidden_letters.len() {
  println!("{}",forbidden_letters[n]);
}
{% endhighlight %}

## Checking numeric string

This function is really simple and allowed me to play a little bit more with strings and cast in Rust

{% highlight rust %}
fn validate_numbers(input_nino: &String) -> bool{
    let numeric_part: String = input_nino.chars().skip(2).take(6).collect();
    return numeric_part.chars().all(char::is_numeric);
}
{% endhighlight %}

In this case we use the chars() iterator, but the key are the all() and the is_numeric part.

{% highlight rust %}
chars -> Is an iterator.
all -> this returns true if the condition is true for all chars of the iteration.
is_numeric -> checks if the char is a number.
{% endhighlight %}

## Ending the experiment
It passes only 45 minutes since I started with Rust, installed the cargo compiler and start to play with it. I've found really fun and easy to start with Rust. It's very powerful, the syntax is clear and even I think that Rust is very verbose, that could be useful for new starters like me.

You have the full code [here](https://gist.github.com/jsalcedo1987/f517fb45fa532eb6becd016c97b1ca6b)