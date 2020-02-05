---
layout: post-sidebar
date: 2020-01-21
title: "The Rubik's challenge"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 5
feature_image: feature-rubik
show_related_posts: true
square_related: recommend-spain
---
<br>
I love Rubik's cube, when you get one of these, it's all about investigating, moving colours from one place to another without a clear strategy trying to figure out how to solve it. At the begining you don't care if your way to solve it is the best one or the fastest one bbut you get a kind of nirvana when you get it done.
This experiment is about do the same but with code.

I'll recieve 4 rubiks cubes:

* `Rust`
* `Ruby`
* `F#`
* `Clojure`

The ojetive is to do a simple program (see rules below) and I've to solve the experiment in 1 hour.
So the idea of this challenge is NOT to make the perfect solution, probably there could be much better solutions available and like the Rubik cube is all about iterations to discover shortcuts to get to the optimal solution,

## Why only 1 hour?
Set a time constrain makes it more fun. Those are programming languages that I've never tried before so even 1 hour looks a lot of time, keep in mind that in one hour I've to setup the environment, read some docummentation, figure how to debug the program etc..


## Why those 4 programming languages?
Those are programming languages that I've never tried before, and there are two functional programming languages.


## The challenge
Develop a simple NIN validator (UK National Insurance Number):

```
The format of the number is two prefix letters, six digits and one suffix letter.
Neither of the first two letters can be D, F, I, Q, U or V. The second letter also
cannot be O. The prefixes BG, GB, NK, KN, TN, NT and ZZ are not allocated.

After the two prefix letters, the six digits are issued sequentially from
00 00 00 to 99 99 99. The last two digits determine the day of the week on which
various social security benefits are payable and when unemployed claimants need to
attend their Jobcentre to sign on.

The suffix letter is either A, B, C, or D.
```

The structure of the program would be:

=> One function to validate the lenght of the Nin. This function should raise an exception if the lenght is incorrect.<br>
=> One function to validate the prefix. This function should match our NIN prefix iterating an array of invalid prefixes.<br>
=> One function to validate the numeric part of the Nin.<br>
=> One function to validate the sufix.<br>

## Next Steps
Day 1 => Rust ([link]({% post_url 2020-01-22-the-rubik's-challenge-rust %})) <br>
Day 2 => Ruby (link) <br>
Day 3 => F# (link) <br>
Day 4 => Clojure (link) <br>