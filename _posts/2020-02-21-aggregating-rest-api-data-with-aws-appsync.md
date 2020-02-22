---
layout: post-sidebar
date: 2020-02-21
title: "Aggregating rest api data with AWS AppSync"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 10
feature_image: feature-graphql
show_related_posts: true
square_related: recommend-coding
youtubeId: 783ccP__No8
---

<br>
We already talked about how [wrap a Rest API in GraphQL using appSync]({% post_url 2020-02-09-wrapping-rest-api-in-graphql-with-aws-appsync %}) in that example we wraped a single Rest API in GraphQL.

Today we are going to combine two HTTP resolvers in a single GraphQL, in other words, aggregate data in a single GraphQL. 
Why? Imagine you have 2 endpoints, one that provides personal information about an employee such as employee's name, age...another endpoint that provides  financial information such as the employee's salary, bonus... 

Now imagine that we want to show on a website the name of the employee and his salary.
Let's ask the server to do all the orchestration for us and return **only** the data we need.

## Parent data
<br>
Our endpoint `api/v1/employees` returns a list of employees, with their id, name, age and position.

{% highlight json %}
{
"employee":[
    {
        "id"        : "1",
        "name"      : "Paul",
        "position"  : "I+D Lead",
        "age"       : 39
    }
    ]
}
{% endhighlight %} 

From that we will create a GraphQL schema as we did in the [other article]({% post_url 2020-02-09-wrapping-rest-api-in-graphql-with-aws-appsync %}:

{% highlight graphql %}
type Employee { 
  id: String
  name: String
  position: String
  age: Int
}

type Query {
	ListEmployees: [Employee]
}
{% endhighlight %}
And after running a query againts our Appsync we will have a list of employees. 

## Agreggating data
<br>
So far so good but we need a way to extract each employee salary in a dynamic way.

We have a Rest endpoint: `api/v1/employees/{id}/financial` that retrieves the employee's salary and other financial information:
{% highlight json %}
{
"financial":
  {
    "employee_id"         : "1",
    "salary"              : "146000",
    "bonus"               : "97000"
  }
}
{% endhighlight %} 

In a standard RestFul integration scenario, we would call to those 2 endpoints doing the aggregation in the client or with another orchestration layer in the server or gateway. In this case we are going to aggregate everything in AppSync. 

## Preparing the schema

First we are going to modify our GraphQL schema to include financial information field in the employee type:

{% highlight graphql %}
type Employee { 
  id: String
  name: String
  position: String
  age: Int
  financial: Financial
}

type Financial {
  employee_id: String
  salary: String
  bonus: String
}

type Query {
	ListEmployees: [Employee]
}
{% endhighlight %}

This schema will allow us to list all the employees, and the object employee contains also the financial information for the employee.
So now the question is: How do we relate those two APIs of fields?

## Resolvers
<br>
We will declare a function to extract all employee data, we can use the following function attached to a pipeline solver:

{:refdef: style="text-align: center;"}
![pipeline-resolver]({{site.url}}/{{site.baseurl}}img/post-assets/graphql-aggr-0.jpg)
{: refdef}

Request template:
{% highlight json %}
{
  "method": "GET",
  "resourcePath": "/api/v1/employees",
  "params":{
  }
}
{% endhighlight %}

Response template:
{% highlight plaintext %}
#if($ctx.result.statusCode == 200)
    $util.toJson($util.parseJson($ctx.result.body).employee)
#else
    $utils.appendError($ctx.result.body, "$ctx.result.statusCode") 
#end
{% endhighlight %}

That will give us the list of employees, now lets extract the salary for each employee dinamically.
We will create another function and another pipeline resolver that this time we will attach to the `financial: Financial` field within our employee object.

The request template will look like this:

{% highlight json %}
{
  "method": "GET",
  "resourcePath": "/api/v1/$ctx.source.id/financial",
  "params":{
  }
  }
}
{% endhighlight %}

And the key concept on this request is `$ctx.source.id` that will automatically take the value of the `id` field within our employee object.
For each request, the child resolver (financial) will have the parent context available. 

With that GraphQL layer in the middle that makes the orchestration work, we will have the data avaiable no matter how many endpoints are behind the scenes:

{:refdef: style="text-align: center;"}
![grpahql-result]({{site.url}}/{{site.baseurl}}img/post-assets/graphql-aggr-1.jpg)
{: refdef}

If you still not understand why GraphQL is so useful, this could answer some of your questions:

{% include youtubePlayer.html id=page.youtubeId %}
<br>
Have a nice weekend!