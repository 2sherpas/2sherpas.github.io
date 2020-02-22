---
layout: post-sidebar
date: 2020-02-19
title: "Coding for Atari 2600 - III"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 20
feature_image: feature-atari
show_related_posts: true
square_related: recommend-retro
---

<br>
Welcome to the 3rd article! If you don't know what is this about, please have a look to the [first article]({% post_url 2020-02-17-coding-for-atari-2600-i %}) of our Atari 2600 adventure.

In the [previous article]({% post_url 2020-02-18-coding-for-atari-2600-ii %}) we created a player, and at the moment, is static in the position x=50, y=50. Today we are going to move it around the screen using the Atari 2600 joystick.

## Move our player around the screen
<br>
Let's start moving our player to the right.
Batari basic has a variable called `joy0right`, that variable is only true if player0 moves the joystick to the right.
Batari basic does not have listerners, so we will have to constantly ask if the player moves the joystick to the right, so it makes sense to include that instruction in the main loop.

If the joystick is not pushed the right we have to jump to the next block of instructions:

{% highlight plaintext %}
 rem JOYSTICK RIGHT
  if !joy0right then goto __Skip_Joy0_Right
  if joy0right then player0x=player0x+1
__Skip_Joy0_Right
{% endhighlight %}

Now we can move our player to the right, but we want to keep our player safe from the limits of the screen, so let's define a constant value outside our main loop to indicate the screen limit `const _P_Edge_Right = 135`  and then we can modify the previous code to verify if the player is not reaching the limit, otherwise we will jump to the following block:

{% highlight plaintext %}
 rem JOYSTICK RIGHT
  if !joy0right then goto __Skip_Joy0_Right
  if player0x >= _P_Edge_Right then goto __Skip_Joy0_Right
  if joy0right then player0x=player0x+1
__Skip_Joy0_Right
{% endhighlight %}

We have to repeat the same logic for all our joystick directions `joy0up`, `joy0down` and `joy0left`.

## Introduction to variables
<br>
In Batari basic by default we have a limited number of variables available. We have 26 variables that go from `a` to `z`. 
In order to make the code more readeable we can assign to them alias. For example, instead to do this:

{% highlight plaintext %}
 if x > 1 then x = x + 1
{% endhighlight %}

We could improve the code readability by using alias using the reserved word `dim`:

{% highlight plaintext %}
 dim _sherpa_position_x = x
 ...
 if _sherpa_position_x > 1 then _sherpa_position_x = _sherpa_position_x + 1
{% endhighlight %}


## Create an enemy
<br>
Without using bankswitching we only have space for two players in our 4kiB game, so the enemy, the yetti, will be the player number 2, but it will be controlled by the game.
After create the yetti, we have to make him to chase our sherpa around the screen.

We will use our loop to move our yetti to the player position. We will introduce some delay in the operation to give the sherpa a chance to escape.

In order to do that, in the main program loop we will introduce a adjustment varible, in each loop we will increase that variable by 1 and every certain number of loops we will move the position of our yetti one pixel to get closer to our sherpa.

The code looks as follow:

{% highlight plaintext %}
 _enemy_move_turn=_enemy_move_turn+1
 if _enemy_move_turn>10 then _enemy_move_turn=1 
 if _enemy_move_turn<9 then goto _Skip_Enemy_Moving
 if _yetti_position_x < _sherpa_position_x then _yetti_position_x=_yetti_position_x+1
 if _yetti_position_x > _sherpa_position_x then _yetti_position_x=_yetti_position_x-1
 if _yetti_position_y < _sherpa_position_y then _yetti_position_y=_yetti_position_y+1
 if _yetti_position_y > _sherpa_position_y then _yetti_position_y=_yetti_position_y-1
_Skip_Enemy_Moving 
{% endhighlight %}

Now we are ready to execute our code and see how our enemy chases our sherpa across the screen.


![screenshot]({{site.url}}/{{site.baseurl}}img/post-assets/atari-iii-0.jpg)


You have the code of this session [here](https://gist.github.com/jsalcedo1987/131412034e7061bab2f05375cbacce50)

The next article will be the last one of this series so we will see how collect the spades and increase the score.

Looking forward to see you in the [next article]({% post_url 2020-02-20-coding-for-atari-2600-iv %})! 




