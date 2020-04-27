---
layout: post-sidebar
date: 2020-04-27
title: "Publish your VS Code Snippets"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 15
feature_image: feature-vscode
show_related_posts: true
square_related: recommend-coding
--- 

<br>
Lost time is never found again, so spending time today on something that can save you time in the future is always a good investment.
I write articles from time to time, so I always refer to previous articles to remember how to insert an image or code block into the article.

We can automate this task using code snippets, and we can also publish them on the market place so that others can also benefit from it.


{:refdef: style="text-align: center;"}
![vscode-snippet-1]({{site.url}}/{{site.baseurl}}img/post-assets/vscode-snippet-1.jpg)
{: refdef}

<br>


## Code Snippets

Let's first define what a code snippet is, wikipedia has a good definition for it:

> Snippet is a programming term for a small region of re-usable source code, machine code, or text. Ordinarily, these are formally defined operative units to incorporate into larger programming modules. Snippet management is a feature of some text editors, program source code editors, IDEs, and related software. It allows the user to avoid repetitive typing in the course of routine edit operations.

In our case, we are going to define some code snippets, for most common tasks such as inserting images, inserting code blocks in our articles ... etc
<br>

## Getting started 
<br>
The first step is to install Yeoman. Yeoman is a generator ecosystem that can generate the structure of many types of projects from frontend projects in Angular or VUE to other types of projects like VS Code extensions.

{% highlight terminal %}
npm install -g yo generator-code
{% endhighlight %}

Once Yeoman is installed, we can run the following command that will generate our project structure for our VS code extension:

{% highlight terminal %}
yo code
{% endhighlight %}

After that, we just have to select "New Code Snippets" and follow the guided steps to create our project structure:

{:refdef: style="text-align: center;"}
![vscode-snippet-2]({{site.url}}/{{site.baseurl}}img/post-assets/vscode-snippet-2.png)
{: refdef}

Once this is done, we will have a project with the following structure:

{% highlight terminal %}
├── CHANGELOG.md
├── README.md
├── package.json
├── snippets
│   └── snippets.json
└── vsc-extension-quickstart.md
{% endhighlight %}


We ara going to work with two files `snippets.json` and `package.json`

## Our first code snippet

Let's open the snippet.json file and paste the following content:

{% highlight plaintext %}
{% raw %}
{
    "Code Snippet": {
      "prefix": "!code",
      "body": [
        "{% highlight ${1:language} %}",
        "${2:code}",
        "{% endhighlight %}"
      ],
      "description": "code block"
    }
}
{% endraw %}
{% endhighlight %}

`"Code Snippet"` is the name of our snippet and it will be activated using the prefix `!code`. The body parameter contains our code snippet.
The `${1:language}` and `${2:code}` are placement variables; the number indicates the tab order and the value after the `:` indicates the initial value of the variable; let's see how it works:

{:refdef: style="text-align: center;width=40%"}
![vscode-snippet-3]({{site.url}}/{{site.baseurl}}img/post-assets/vscode-snippet-3.gif)
{: refdef}

It works perfectly and we can add as many as we want within the same file.
<br>

## Setting up our snippets properties

The package.json contains the snippet settings like the name, description, version etc.
This file also contains the file types for which our plugin will be available.

In our case, our blog entries are written in `.md` files so all code snippet should be available for markdown language files:

{% highlight json %}
{
    "contributes": {
        "snippets": [
            {
                "language": "markdown",
                "path": "./snippets/snippets.json"
            }
        ]
    }
}
{% endhighlight %}

## Publish your code snippets

Now we have the code snippets ready, is time to package & publish it.
First we need to install vsce using npm:

{% highlight plaintext %}
npm install -g vsce
{% endhighlight %}
 
And then create a publisher in the [market place](https://marketplace.visualstudio.com/manage).
In order to login into our publisher we will need a token that we need to create from the DevOps Azure portal.
Once we complete those steps, we can just login:

{% highlight plaintext %}
vsce login (your publisher name)
{% endhighlight %}


Package the extension:

{% highlight plaintext %}
vsce package
{% endhighlight %}

and publish it:

{% highlight plaintext %}
vsce publish
{% endhighlight %}


We can track the progress of our extension from the market place management console, and after a couple of minutes we will see how our extension status changes to published:

{:refdef: style="text-align: center;"}
![vscode-snippet-4]({{site.url}}/{{site.baseurl}}img/post-assets/vscode-snippet-4.png)
{: refdef}

{:refdef: style="text-align: center;"}
![vscode-snippet-5]({{site.url}}/{{site.baseurl}}img/post-assets/vscode-snippet-5.png)
{: refdef}



Thanks for reading