---
layout: post
title:  "Secure your API using JWT in Flask"
date: 2020-07-09 08:00:00
categories: coding
author_name : David Antona
author_url : /author/david
author_avatar: david
show_avatar : true
read_time : 15
feature_image: feature-jwt
show_related_posts: false
square_related: recommend-wolf
---

In my [last post](https://2sherpas.github.io/Flask-API) we learned how to create a simple API using Flask and how to test it using Postman, it was super quick and easy. Today we will explore how to make our API safe by using JWT.

Hands back to coding :)

**Our starting point**

Well, this time I will disappoint you... we all know developers are lazy and we love to copy and paste code as much as we can... it will be ideal if we can start this example by using the code we have created in our previous post but... I want to make things a bit better and to do it we will need to start our code from scratch again; to be honest it won't be thousands of lines so don't cry just yet.


**Our new code**

If we want Flask to adhere to really good API standards we want to use this way of creating APIs

The first thing we need to do is to install a new Python library

> pip3 install Flask-RESTful

and 

> pip3 install Flask-JWT

When you are creating API endpoints using this new way, your endpoints will be a class and not a route within your Flask app

Let's go to our code editor and start writing our example.

{% highlight python linenos %}

from flask import Flask
from flask_restful import Resource, Api

app=Flask(__name__)
api = Api(app)

class Mountain(Resource): #Mountain inheritance from Resource class
    def get(self, name):
        return {'mountain' : name}

api.add_resource(Mountain, '/mountain/<string:name>')

app.run(port=5000)

{% endhighlight %}

Ok, let me give a quick explanation of our new code here.

The first two lines have our imports, Flask and our new flask_restfull library

Then we have created our Flask app app=Flask(__name__) and a new thing api=Api(app) which will help us publish APIs from our Flask application, I hope it does make sense for now.

The big difference here with our previous Flask apps is that we haven't created a @route.... this time we have created a class that will contain our endpoint function, in our example class Mountain which inherits from Resource that we have imported from our flask_restful library.

Well, if we haven't created a route how we know where our new endpoint will be published, by using this last line of our code

>>api.add_resource(Mountain, '/mountain/<string:name>')

This line basically is telling our app that our class Mountain will be available under the route /mountain

Our function it's really simple and will just return the name we pass to it, but we are not learning Python we are learning about APIs and JWT so I don't care much about this function right now.

**Time to test our first example**

Now that we have created a new endpoint using the new and better way of publishing APIs in Flask it is time to tet it and make sure our code is working; we know how to do this from our previous post so open your Postman and let's do it

![testing]({{site.url}}/{{site.baseurl}}img/post-assets/mountainApi.png)

As you can see if we call "localhost:5000/mountain/everest" we get our mountain back with the name we sent in as a parameter.

Good news, code is working but we haven't learned much yet; it is time to step up our security for the API

**Let's secure our API using JWT**

If you don't know what JWT is you can find plenty of information on Internet but as a super quick explanation, we will say that JWT is the acronym for JSON Web Token and it is a way to obfuscate data so it is secure; remember we are introducing JWT to increase security and it will be pointless to do it if our user and password are sent in plain text.

Once JWT is installed we can go to our code and introduce some changes. We need to create a secret key for the whole thing to work; for this example, we will do it by creating the key in our app.py but this is not what you want to do in a Production environment, remember this is just for a quick learning model.

{% highlight python linenos %}

from flask import Flask
from flask_restful import Resource, Api

app=Flask(__name__)
api = Api(app)

app.secret_key = 'sherpaSecret'

{% endhighlight %}

Now we will create two new files, user.py and security.py

**user.py**

We will create our user class in this file so we can then easily interact with user information

{% highlight python linenos %}

  class User:
      def __init__(self,_id, username, password)
      #_id because id is a python reserved word
      self.id = _id
      self.username = username
      self.password = password

{% endhighlight %}


Our user will have an id, username and a password; note that we have used _id instead of id because id is a Python reserved word.

**security.py**

Time to work in our second file, security.py

{% highlight python linenos %}

from user import User

users = [
    User(1, 'sherpa','everest')
]

username_mapping = { u.username: u for u in users}


userid_mapping ={u.id: u for u in users}


def authenticate(username, password):
    user = username_mapping.get(username, None)  #None as default value
    if user and user.password == password:
        return user

def identity(payload):
    # payload is the content of the userid token
    user_id = payload['identity']
    return userid_mapping.get(user_id, None)


{% endhighlight %}

Let's review our new code.

The first thing we did is to import the User class from our user file, then we have created a list of user and create a new user using our new User class; our new user with id 1, the username will be sherpa and the password everest.

After that we have two important lines of code, username_mapping and userid_mapping, they will help us to avoid iteration over the user list if we have many users, for our example, it doesn't make a big difference but is nice to think in advanced for when we have thousands of users (I know you will use a database...) 

Now we have created our function to authenticate our user, the function receives username and password as parameters and if both are ok it will return the user.

Finally, we have created our second function, identity, that will be responsible for managing the content of our token, in our case we know that our user_id is the identity from the payload. 

**Back to the app.py code**

We have set up all the things we need to secure our APIs using JWT, now is time to make a few changes in our app.py file to make our endpoint secure.

{% highlight python linenos %}

from flask import Flask, request
from flask_restful import Resource, Api
from flask_jwt import JWT, jwt_required
from security import authenticate, identity

app=Flask(__name__)
app.secret_key = 'sherpaSecret'
api = Api(app)


jwt = JWT(app, authenticate, identity)

class Mountain(Resource):
    @jwt_required()
    def get(self, name):
        return {'mountain' : name}

api.add_resource(Mountain, '/mountain/<string:name>')

app.run(port=5000)


{% endhighlight %}

As you can see we have introduced some new imports from flask_jwt and from our security.py file

>>from flask_jwt import JWT, jwt_required
>>from security import authenticate, identity

The new interesting line is this one

>>jwt = JWT(app, authenticate, identity)

with this magic line, JWT will automatically publish a /auth endpoint that we can call, that endpoint will use our authenticate and identity code and if everything is ok will return a JWT token.

And last but not least 

>>@jwt_required()

we have used that decorator to modify our function and force it to require a jwt token

If everything works as expected our endpoint now will require a valid jwt token to work.

**Back to testing again**

Once again we will be using Postman to test our API

Let's see what happen when we try to call our /Mountain endpoint again

![error]({{site.url}}/{{site.baseurl}}img/post-assets/jwtError.png)

As you can see we are not getting back our mountain but an error, that's because now the endpoint expects us to pass a valid jwt token.
I know a 500 internal server error is not what we should be getting but in this quick tutorial we haven't work in error handling, we will talk about it another day and we will get better errors.

So, how do we make it work then?

>Step 1 - get a JWT token

to generate our token we need to call to the new endpoint that flask_jwt published for us; we can do it easily using Postman, the URL for it is "localhost:5000/auth"
the only difference this time is that we need to update the information in the Body of the request to include our user and password

![jwtToken]({{site.url}}/{{site.baseurl}}img/post-assets/jwtToken.png)

If we execute that call we will get a token back 

![tokenOk]({{site.url}}/{{site.baseurl}}img/post-assets/tokenOk.png)

>Step 2 - use the token to call our endpoint

Once we have our token from the /auth endpoint we need to copy the information of the token without the quotes and use it in our Mountain endpoint

To do that we simply go to our Mountain endpoint and under the Headers section we add a new key-value called "Authorization" with a value equal to "JWT token copied..." it is really important that between the word JWT and the token we have a blank space.

If we execute it again....

![withToken]({{site.url}}/{{site.baseurl}}img/post-assets/apiWithToken.png)

Yes !!! we got our mountain back :)


**We did it!!!** 

We have learned how to create APIs using a better way and we have learned how to secure them using JWT.

As you know from previous articles the intention here is to learn the basic concept so you can take it from here and continue your journey, I hope you have found this article interesting.

See you in the next article.
