---
layout: post-sidebar
date: 2020-01-23
title: "The Rubik's challenge - Ruby"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 12
feature_image: feature-ruby
show_related_posts: true
square_related: recommend-spain
---

<br>
So here we are with the second Rubik's cube. If you don't know what is this about, please check this [link]({% post_url 2020-01-21-the-rubik's-challenge %}).

My first job interview for a developer position was for Junior developer role in Ruby on Rails; I had no idea about Ruby at that time but I promised that I could learn Ruby if they took me for the Job. 

As I'm now writting this post you could imagine that finally I didn't take that job but it's never too late to close the loop and fulfill my promise.


## Validating Lenght

Let's start with the simplest function of the program:

{% highlight ruby %}
def validate_length(input_nino:)

  if input_nino.length == 9
    return true
  else
    raise 'Invalid Lenght!!'
  end
end
{% endhighlight %}

In Ruby apparently you don't have to specify the return type of a function because it's assuming that a function is always returning an object;then it's up to you to define an object if you need it later. I've found this really friendly for the developer.

The way to raise exceptions is really easy too, just put "raise" and that will raise a RuntimeError exception. That's enought for my purpose; but raise another type of exception is simple as this

{% highlight ruby %}
raise StandardError.new "It's really easy, right?"
{% endhighlight %}

And of course, you can define your custom exceptions and raise them in the same way.

## Validate Prefix

This is the code of the function:

{% highlight ruby %}
def validate_prefix(input_nino:)

  my_prefix = input_nino[0..1]
  invalid_prefixes = ["D", "F", "I", "Q", "U", "V", "BG", "GB", "NK", "KN", "TN", "NT", "ZZ"]

    for invalid_prefix in invalid_prefixes
      if my_prefix.include? invalid_prefix
        print "#{invalid_prefix} this letter is in the string.\n"
        return false
      end
    end
  return true
end
{% endhighlight %}

I think that the learning curve coming from popular languages like Java is very low. Even the syntax is slighly different, the whole code structure looks very similar.

Let's start with the sustring to extract our prefix:

{% highlight ruby %}
input_full = "ABC"
input_sub = input_full[0..1]
# This will print AB
{% endhighlight %}

Just keep in mind that both indexes, it could make you made some mistakes see the diference below from example with Java:

{% highlight ruby %}
String input_full = "ABC";
String input_sub = input_full.substring(0,1);
# This will print A
{% endhighlight %}


Regarding to the iterations, in Ruby you can also use iterators over collections or arrays. 


## Checking numeric string

This is the code of the function:

{% highlight ruby %}
def validate_numbers(input_nino:)
  numeric_part = input_nino[2..7]
  return numeric_part.is_numeric?
end
{% endhighlight %}

In Ruby, to check if a string is numeric, there is not native solution you have to do it by yourself (at least that was the result of my quick research in the doccumentation). You can do it overwritting the String class or even with a regular expresion. I've choose the first one just to play a little bit more with Ruby:

{% highlight ruby %}
class String
  def is_numeric?
  self.to_i.to_s == self
  end
end
{% endhighlight %}

The important part in this function is to_i, that converts an String to integer ( https://apidock.com/ruby/String/to_i ) and after that it converts to String again and try to asing it to the same variable. If the conversion fails, the assigment is not possible hence it's not numeric. Maybe there are many ways to to it but I've found this quite clean and straightforward.

## Testing the code

Until here, everything was smooth so I still have enought time to do some unit tests because the testing is as important as the functionality itself. I've found the testing in ruby easier than I expected, just a couple of imports (one of them for the file that contains the source code that we want to test) and that is:

{% highlight ruby %}
require_relative "validator"
require "test/unit"

class TestValidator < Test::Unit::TestCase
  def test_validate_correct_length
  assert_equal(true, validate_length(input_nino: "AB123456A") )
end
{% endhighlight %}

As expected the framework contains a set of asserts (assert_equal, assert_not_equal, assert_not_nil....).

For the function that return an exception, we can use another couple of asserts, "assert_raise" to check the type of exception raised and "assert_equal" to check that the exception returns the expected message:

{% highlight ruby %}
def test_validate_incorrect_lenght
  exception = assert_raise("RuntimeError") { validate_length(input_nino: "123456A") }
  assert_equal("Invalid Lenght!!", exception.message)
end
{% endhighlight %}

After that, just run the tests and check the report:

{% highlight ruby %}
Started
Finished in 0.001416 seconds.
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
8 tests, 9 assertions, 0 failures, 0 errors, 0 pendings, 0 omissions, 0 notifications
100% passed
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
5649.72 tests/s, 6355.93 assertions/s
{% endhighlight %}
<br>

## Ending the experiment

It tooke me less than 30 minutes to make this simple program without previous experience in Ruby. That's give you an idea about the fast that you can start with it. I can understand now why Ruby is also in the most loved programming languages ranking in the StackOverflow because I can feel now that attraction to it. 

Ruby has been always surrended of critical opinions saying that is going to die at some point, that it had a great opportunity ten years ago but now there are better alternatives to Ruby...maybe it's true but I can also understand now why is still alive.

You have the full code [here](https://gist.github.com/jsalcedo1987/20e0b15055305bfb6ec4d7ba45081207)