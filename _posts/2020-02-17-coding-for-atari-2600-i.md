---
layout: post-sidebar
date: 2020-02-17
title: "Coding for Atari 2600 - I"
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
If you want to know an interesting fact about me, is that I love to collect video consoles, old computers and retro video games. My collection goes from vintage computers like Commodore 64 or Spectrum 2+ to the moderns PlayStations or XBOX...but In my collection there is an special place for the Atari 2600:

{:refdef: style="text-align: center;"}
![my-gaming-room]({{site.url}}/{{site.baseurl}}img/post-assets/atari-i-0.jpg)
{: refdef}

Why? Because It was my first video console and I spent big part of my childhood playing to Keystone Kapers or Pitfall.

When I was a child, I was wondering how someone could program such cool games, so now that I grew up and I've been programming for years, it's time to answer that question.

## A bit of history
<br>
This not the first time that I've program videogames, I've did before for PC and Mobile devices using Unity engine, and when I develop a game, especially for mobile platforms, I must take into account certain limitations with polygons, textures, etc... But I can confirm that programming for Atari 2600 is another level.

The specifications of Atari 2600 sound ridiculous today: a CPU that runs at 1.19 MHz and 128 **bytes** of RAM for scratch space, the call stack, and keep the state of the game; And you would probably say "Ok, but the game is stored in the cartridge that will probably have much more space, right?"; The cartridge has a limited addressable memory of 4 KB. So how would you program a video game in 4kb? Including sounds, music, two players, etc. It is definitely a challenge.

## Game design 
<br>
First step for any game designer is to think a story for your game, something to tell and how would you like your game looks like.

## The story
<br>
Since the name of the blog is 2 Sherpas, I would like to develop a video game about Sherpas. My idea is to have a sherpa (the player) who has to rescue another sherpa who is in trouble, was captured by the Yetti (Abominable Snowman) and buried in the snow.
We have to capture spades to dig up our friend while the Yetti tries to capture us.

The tile of this game will be 2Sherpas.

## Screen design
<br>
I will use a pen and iPad for this. In this step, I try to be as accurate as possible with the proportions of the screen and the items shown on the screen:

![my-gaming-room]({{site.url}}/{{site.baseurl}}img/post-assets/atari-i-1.jpg)

It looks great for me!

## Hardware limitations
<br>
In Atari 2600 we only have: 2 Players, 2 Misiles and a background. That's all, it's fixed; we cannot create more players or more misiles, we'll talk later about bankswitch, but let's consider that true for now.

And that means that everything extra will have a cost, for example, if we configure a milticolored player, we will lose the missiles, that means that our player can no longer fire. The colors are stored in the memory and we only have 4 KB, so these are the limitations that you must take into account when you program in Atari 2600.

We will talk more about memory constrains in the next articles.

Looking forward to see you in the [next article]({% post_url 2020-02-18-coding-for-atari-2600-ii %})!



