---
layout: post-sidebar
date: 2020-02-20
title: "Coding for Atari 2600 - IV"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 25
feature_image: feature-atari
show_related_posts: true
square_related: recommend-retro
youtubeId: DnbdjWflZfE
---

<br>
Welcome to the last article of our programming for Atari 2600 series! If you don't know what is this about, please the have a look to the [first article]({% post_url 2020-02-17-coding-for-atari-2600-i %}) of our Atari 2600 adventure.

In the [previous article]({% post_url 2020-02-19-coding-for-atari-2600-iii %}) we created our sherpa and the yetti that go after him. In this last article we will collect the spades, we will see how to increase the score and how to add colors to the scenario.

## Collecting the spades
<br>
In our game we have to escape from the yetti but at the same time we have to collect spades to dig up our friend who is buried in the snow.
For this purpose we will use the player's `missiles`.  Missiles are an artifact that allows our user to shoot at enemies, each player has a missile and is again represented by its coordinates, the missile for player 0 is represented by `missile0x` and` missile0y`.

In this game we are not going to shoot missiles so we can use it to place it static in the screen, representing an spade.
Then, we have to complete two tasks: place our spade in a random position of the screen and detect a collision with it.


First we will place our missile in a random position using the `rand` instruction, rand produces a random number but we would like to limit the range of the number produced to keep the space whitin the limits of the screen, so we will use the character `&` to produce numbers just in a certain range. Let's see how it works with an example:

{% highlight plaintext %}
 _my_random_number = (rand&31) ; this will produce a number between 0 and 31
 _my_random_number = (rand&%00011111) ; this is the same of above but written in binary

 ; following the binary approach the &%00011111 eliminates the first 3 bits i.e the 0s. 
 ; So then the maximun number could be 31 and the minimum 0. 
 ; This method only works with power-2 numbers 
{% endhighlight %}

Now that we can produce random numbers in a certain range, we have to detect if our sherpa has collected the spade, in other words we have to detect if our sherpa has collided with the spade. Batari basic makes the task easier for us by using `collision` instruction. Let's see how our code looks so far:

{% highlight plaintext %}
 if !collision(player0,missile1) then goto __Skip_p0_Collision
 _spade_position_x = (rand&63) + (rand&63) + 22
 _spade_position_y = (rand&15) + (rand&31) + 24
 missile1x = _spade_position_x : missile1y = _spade_position_y
__Skip_p0_Collision
{% endhighlight %}

<br>
## Increasing the score
<br>
Ok, now every time we capture a spade we would like to increase the score and I would like to gradually decrease the points if the sherpa takes too much time to pick it up. The strategy we will follow is to decrease the value each time the yetti moves a position.

{% highlight plaintext %}
 if _enemy_move_turn = 1 then dec _score = _score - $1
{% endhighlight %}

And then, after capturing the spade, our score will increase with total points and we will restart the points for the following sword:

{% highlight plaintext %}
 score = score + _score
 _score = $99
{% endhighlight %}

Meanwhile, if we have picked up the spade, we would like to dig up a block of snow from our friend's head. We can do this by turning off the pixel with the following instruction `pfpixel x y off` where x and y are the pixel coordinates.
<br>
## Setting colors
<br>
We can add some additional features to our game using `kernel options`, some of them like putting multicolored sprites have some memory cost and we will have to get rid of some features in our game like missiles, players, etc.
In this case, the background color has no cost. We can choose only one color per row and we can do it by configuring `pfcolors` outside the main loop:

{% highlight plaintext %}
 set kernel_options pfcolors

 pfcolors:
   $0E
   $9A
...
end
{% endhighlight %}
<br>
## The end
<br>
If you download the final code, you will notice that the final game has some additional features, but basically it is to apply the same concepts over and over again.
We could improve our game by using the bankswitch or even by modifying the assembly code directly, but the idea was to make it as simple as possible to share it with you and that you can replicate it without having to be overwhelmed by a lot of information.

{% include youtubePlayer.html id=page.youtubeId %}
<br>
By the way, my official tester loves the game!

![favourite-tester]({{site.url}}/{{site.baseurl}}img/post-assets/atari-iv-0.jpg)


You have the code final code [here](https://gist.github.com/jsalcedo1987/79841a643355308b1b2b4efd817a3948)

Have a nice week! 



