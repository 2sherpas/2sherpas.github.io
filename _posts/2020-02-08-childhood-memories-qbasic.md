---
layout: post-sidebar
date: 2020-02-08
title: "Childhood memories - QBasic"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 12
feature_image: feature-qbasic
show_related_posts: true
square_related: recommend-spain
---

<br>
If you have read [my profile](https://2sherpas.github.io/author/jaime/) you already knows that I started to program in a early age, when I was 7.

At that time I didn't even have a computer yet and so I did my firts program using the computer of the training that I was attending for.

In the QBasic training we only had 6 or 7 computers and we were about 12 people but most of the time we only used pen an paper and at the end of the lessons we had some minutes to try to make the programs work.  

At the end of the first week our instructor asked to us to make our own program, we can choose the topic and the program structure. My class mates were an avergare of 10 years older than me so, as you can imagine, they choosed diferent topics than a 7 years old boy.

I tried to do something using all the things that I learn during the first week: capture user input, print text, draw things on the screen...

I don't have the source code of my first program but I still remember how it looked like.
The program prompted to the user about its name and after that a nice stick man greets it:

{:refdef: style="text-align: center;"}
![pgm-gif]({{site.url}}/{{site.baseurl}}img/post-assets/qbasic_img_0.gif)
{: refdef}

I've replicated it trying to imagine how I would do it with when I was 7 and using a fantastic [QBasic emulator](https://www.qb64.org/portal/) for mac; and this is the result:

{% highlight qbasic %}
SCREEN 12
DIM counter AS INTEGER
DIM inputName AS STRING

counter = 0

INPUT "Whats your name "; inputName$

DO
    CLS
    IF counter MOD 2 = 0 THEN
        'left arm up
        LINE (160, 170)-(230, 110), 15
    ELSE
        'left arm down
        LINE (160, 170)-(230, 130), 15
    END IF
    'draw head
    CIRCLE (160, 100), 40, 15
    PAINT (161, 100), 15
    'draw body
    LINE (160, 100)-(160, 250), 15
    'draw right arm
    LINE (90, 200)-(160, 170), 15
    'draw right leg
    LINE (110, 300)-(160, 250), 15
    'draw left leg
    LINE (160, 250)-(210, 300), 15
    LOCATE 7, 35
    PRINT "Hello "; inputName$
    SLEEP 1
    counter = counter + 1
LOOP UNTIL counter > 10
{% endhighlight %} 

I'm not ashamed of my first program and to be honest I'm very proud of what I did 25 years ago. Putting everything in a context, at that time we didn't have internet, youtube, online tutorials, editors with code autocomplete or graphical interfaces so everithing that we did comes purely from our own imagination and by learning by mistake.

QBasic was amazing for that simplicity, there are no abstract classes, object, methods... just write code and run it.
But I also learnt how easy was to create an infinite loop and why few GOTO can produce a nice [spagetti code](https://en.wikipedia.org/wiki/Spaghetti_code).

![goto]({{site.url}}/{{site.baseurl}}img/post-assets/qbasic_goto.png)

## Level up

At this step of the experiment I feel very nostalgic but I though that It could be even better if I build and run the same program in the first computer that I had, a Toshiba SX2200. My parents bought me that machine with a powerful 386 processor after discover my passion with QBasic.

What a great childhood memories...
![toshiba]({{site.url}}/{{site.baseurl}}img/post-assets/qbasic_img_1.jpg)

Experiment completed, challenge achieved!
![toshiba2]({{site.url}}/{{site.baseurl}}img/post-assets/qbasic_img_2.jpg)

Have a nice weekend!