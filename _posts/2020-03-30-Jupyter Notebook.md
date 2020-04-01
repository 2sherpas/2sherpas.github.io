---
layout: post
title:  "Jupyter Notebook"
date: 2020-03-30 08:00:00
categories: journey
author_name : David Antona
author_url : /author/david
author_avatar: david
show_avatar : true
read_time : 10
feature_image: feature-jupyter
show_related_posts: false
square_related: recommend-wolf
---

Now that we are all having a mandatory indoor live I have decided to take that extra time that we saved from communitng and use it to learn new skills, refresh old ones and have some fun trying to rescue old projects.

One of the things I've done is enrolled in a basic Python trainning course and it was really fun, revisits the basics of coding again and learn the good stuff from Python with some order and direction; a really positive experience that I will recommend to everyone.

As I said this course has bring many good things. I am particullary happy with one of them, [Jupyter Notebook](https://jupyter.org/), it is a really simple tool that helps you learn coding and understand basic coding concepts avoiding the terminal and configuring any development UI.

**Let's try it !!**

First we ned to install Jupyter Notebook in our computer. As you know I am currently using Ubuntu and it is really simple to get it up and running.

>pip install jupyterlab

>pip install notebook

And you are ready :) it could't be any easier.

On a side note I would like to mention two things

* In order to use the pip command you need to have python installed in your computer
* You can get Jupyter Notebook as part of the Anaconda installation, Anaconda will install many libraries and tools that you might find useful but I decided to go for a simple approach and install what I need, in this case I just wanted the Jupyter Notebook.


Now that we have our Jupyer Notebook install it's time to use it, but first you need to start it, so back to the terminal and run below command in the directory you would like to start the notebook

> jupyter notebook

By default it will run on the port 8888 so if you want to use it you need to open your favourite browser and go to htt://localhost:8888


![Jupyter Notebook]({{site.url}}/{{site.baseurl}}img/post-assets/jupyter.png)

There we are, 3 simple steps to get your Jupyter Notebook up and running in your computer. The first thing you will notice is that Jupyter Notebook is a file explorer and you can navigate your whole computer hard drive with it.

**Let's create our first Notebook**

To start using the notebook for coding we need to create our first NoteBook, another simple task; click the New button and select Python 3

![New Notebook]({{site.url}}/{{site.baseurl}}img/post-assets/newNotebook.png)

This will create and open our first Jupyter Notebook

![New Notebook]({{site.url}}/{{site.baseurl}}img/post-assets/cleanBook.png)

One thing that I like to do before I start coding is to give the noteBook a name, by default Jupyter will call it "Untitled" and it's not a firednly name hehe, so let's chang it for something better; make double click on the "Untitled" word and it will open the rename option

![Rename Notebook]({{site.url}}/{{site.baseurl}}img/post-assets/rename.png)

So now that we have our NoteBook ready it's time to try it and do some simple coding :)

![Coding Notebook]({{site.url}}/{{site.baseurl}}img/post-assets/coding.png)

As you can see, you can run simpel commands directly into your Notebook and get the output of those commands; I find this really helpful to leara new language. In the example above I have show you how you can run basic maths operations or use the basic Python print function to print some text; we have even created a variable and store the result of an operation, then call the variable and get the result. 

But that's not all, you can use Jupyter Notebook to test more complex code in the same way, let's see another example where we defined a funtcion

![A function Notebook]({{site.url}}/{{site.baseurl}}img/post-assets/jupyterFunc.png)

So now we have defined a function that return something and we can either use the predefine value or pass it a new value. Again really simple coding but great to see that we can writte a function and test it directly in the Notebook.

We can define loops and test our exception handling code...

![More coding Examples]({{site.url}}/{{site.baseurl}}img/post-assets/moreExamples.png)

I hope that by now you can see how helpful and easy to use this is, writting code and getting results in your Notebook is a great way to learn.

Here you have one last example of how useful and powerful it is, you can even import libraries and use them in the notebook

![Import coding Examples]({{site.url}}/{{site.baseurl}}img/post-assets/importPython.png)


I hope you have enjoyed this quick introduction to Jupyter Notebook and find it as useful as I do. I personally think that this is a great tool to teach our kids how to code and introduce them to easy coding concepts in a visual way.

See you in the next article.

Thanks for reading it.