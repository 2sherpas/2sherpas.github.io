---
layout: post-sidebar
date: 2020-02-18
title: "Coding for Atari 2600 - II"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 25
feature_image: feature-atari
show_related_posts: true
square_related: recommend-retro
---

<br>
Let's start coding our game for Atari 2600! If you don't know what is this about, please have a look to the [first article]({% post_url 2020-02-17-coding-for-atari-2600-i %}) of our Atari 2600 adventure.

## Only 4kiB
<br>
It's not the first time that I've program videogames, I've did before for PC and Mobile devices using Unity engine, and when you a developing a game, specially for Mobile platforms, you have to keep in mind certain limitations with poligons, textures etc... but I can confirm, that programming to Atari 2600 is another level.

Atari 2600 specifications sounds ridiculus today: A CPU running at 1.19 MHz and 128 **bytes** of RAM for scratch space, the call stack, and keep the state of the game; And you would probably say "Ok but the game is stored in the cartridge that probably it will have much more space, right?"; The cartridge has a limited addressable memory of 4 kiB. So how would you program a videogame in 4kb? Including sounds, music, sprites etc...Let's start with the fun

## Bankswitch
<br>
I already talked about the 4kb limitation, that is the limitation for standard cartridges, but as soon as the Atari 2600 became increasingly popular, some games began to include additional memory in the cartridge of up to 32 kB. To exploit this new feature, there is a technique called bankswitch but I will not use it in this tutorial since I would like to keep the structure of the game as simple as possible and 4kb will be enough for our game.

## Getting ready
<br>
For this exercise we are going to use a set of tools: Visual Studio Code as our IDE, Stella as our Atari 2600 simulator, Batari basic as programming language.

If you are starting with Batari Basic or plan to start with it, make an extra effort to write comments and name the variables correctly, since the code will be full of instructions Go-To and otherwise it will be lost in your code very soon.

Let's understand how the screen is structured; there is two main areas, our playground and score:

{:refdef: style="text-align: center;"}
![playfield]({{site.url}}/{{site.baseurl}}img/post-assets/atari-ii-0.jpg)
{: refdef}

The playing field has 11 vertical lines, each of these lines is 8 pixels high and 32 pixels wide. There is also a small margin of 4 pixels on each side.
We could begin to define the scenario as follows:


{% highlight plaintext %}
  playfield:
   .................XXXXXXXXXXXXXXX
   XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
   X..............................X
   X..............................X
   X..............................X
   X..............................X
   X..............................X
   X..............................X
   X..............................X
   X..............................X
   XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
end
{% endhighlight %} 

`.` symbol means that the pixel is off, `x` means that the pixel is on.  

## Drawing our player

In the same way that the field, our character will be a collection of 0 and 1 which indicates if the pixel is on or off. 

{% highlight plaintext %}
 player0:
        %00101000
        %00101000
        %00010000
        %01010100
        %01010100
        %00111000
        %00010000
        %00111000
        %00111000
        %00111000
        %00010000
end
{% endhighlight %}

We can place our new player in the field by setting up the following coordinate variables:

{% highlight plaintext %}
 player0x=50
 player0y=50
{% endhighlight %}
<br>

## Program structure

The chip that renders our game on the screen only contains enough information to draw one line at a time, so the CPU reloads new information every time a line is different from the previous one.

The screen is drawn sixty times per second, which means that our game is in a continuous main loop enclosed in the start instruction: `main` and the end instructions` goto main`.

Within this main loop, we will have to control our flows using variables.
At the moment, since we still don't have any logic, we can place everything outside the main loop.

We are ready to run our code in the simulator:


![screenshot]({{site.url}}/{{site.baseurl}}img/post-assets/atari-ii-1.jpg)


You have the code of this session [here](https://gist.github.com/jsalcedo1987/bf5fb6c2cdd42f3f59d23d483545f67f).

In the next article we will start to move our players around the screen.
Looking forward to see you in the [next article]({% post_url 2020-02-19-coding-for-atari-2600-iii %})! 



