---
layout: post-sidebar
date: 2020-02-11
title: "Wrapping Rest API in GraphQL with AWS AppSync"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 25
feature_image: feature-graphql
show_related_posts: true
square_related: recommend-spain
---

<br>
Today is 11th February of 2020 so probably you have been hearing for around two years about how good GraphQL is; and it is a reality since not only Facebook but also big companies like [Github](https://developer.github.com/v4/explorer/) or [Microsoft](https://docs.microsoft.com/en-us/samples/microsoftgraph/graphql-demo/graphql-for-microsoft-graph-demo/) adopted it long time ago.

Let's assume that you have an REST API and you want to check how GraphQL works; What we are going to do is to wrap an existing REST into a GraphQL using a tool from other of the internet's big players: Amazon WebServices.
<br>
{:refdef: style="text-align: center;"}
![aws_appsync]({{site.url}}/{{site.baseurl}}img/post-assets/appsync_img_1.png)
{: refdef}
<br>

## Our Rest API

For this example we are going to use a [beeceptor](http://beeceptor.com) to generate a couple of mock endpoints.  
The first one: `/api/v1/employees` will retrieve an array of employees and the second one  `/api/v1/employee/{id}` will return only the data of an specific employee.

Beeceptor allows you to make up to 50 calls per day under their free plan so it will be more than enough for this experiment.

The data returned by the Rest API will have the following simple structure.

{% highlight json %}
{
    "id": "1",
    "employee_name": "Tiger Nixon",
    "employee_salary": "320800",
    "employee_age": "61",
    "profile_image": ""
}
{% endhighlight %} 
<br>

##  Our use case

Now suppose that we would like to show in our frontend the only the users ids. With Rest we would query the /users endpoint and once we get the response we filter / discard the additional fields. That force us to retrieve sensitive data that we don't need and there is where GraphQL comes to the rescue.

Rest is a server-drive approach however GraphQL is a client-driven strategy, it means that is the client (Mobile or Web) the one that is driven the conversation fetching the data that it needs; it makes GraphQL faster than REST and it could also speed up your development in the frontend as you don't need to deal with unneccesary data. 

{:refdef: style="text-align: center;"}
![aws_appsync_2]({{site.url}}/{{site.baseurl}}img/post-assets/appsync_img_2.jpeg)
{: refdef}

<br>

## Introducing AppSync

AppSync is a tool within the AWS ecosystem, it allows you to define a GraphQL schema and connect it with diferent resolvers as http endpoints, databases, AWS Lambdas etc...

But just to reduce the scope of our article, we are going to focus only in Http resolvers and to not overcomplicate the example, we are going to use the web interface of AWS. The same example can be replicated using the terminal or CloudFormation.

First we need to login in the AWS console and look for **AppSync**, at the top right corner you have a button that sais "Create API" that will bbe our starting point.

AWS provides a good wizard if you want to build your GraphQL api in top of a DynamoDB but in our case, we will select "Build from Scratch".


![aws_appsync_3]({{site.url}}/{{site.baseurl}}img/post-assets/appsync_img_3.png)
 

After we setup a name for our AppSync API the following set of options are displayed:

=> Schema <br>
=> Data Sources <br>
=> Queries <br>
=> Caching <br>
=> Settings <br>
=> Monitoring <br>

We will focus on the three top ones as the other ones are more related to the API configuration more than the functionality.
<br>

## Schema
The schema will define how our API is structured, in other words; all the fields, it types and objects that could contains other objects.

So how can we generate a GraphQL schema from the JSON object structure that we have in our REST? There are two ways, you can do it manually going through all the fields of the JSON and examinating their types or you can use a tool that can do it for you.

If the JSON is very complex you can use tools like [json-to-simple-graphql-schema](https://github.com/walmartlabs/json-to-simple-graphql-schema) that can do the transformation for you using their [online editor](https://walmartlabs.github.io/json-to-simple-graphql-schema/) or using the command line tool:

{% highlight shell %}
$ cat users.json | npx json-to-simple-graphql-schema > our_schema.graphql
{% endhighlight %} 

In our case our JSON is be very simple so we can build the GraphQL schema manually.

{% highlight graphql %}
type Data {
  id: ID!
  employee_name: String
  employee_salary: String
  employee_age: String
  profile_image: String
}
{% endhighlight %} 

Now that we have our schema ready, let's paste it into our schema editor in AppSync:

![aws_appsync_4]({{site.url}}/{{site.baseurl}}img/post-assets/appsync_img_4.png)

<br>

## Datasource

The next step is to define our datasource, this step is really straighforward for Http calls. The process for other datasources like DynamoDB is slighly different as it requires to provide some aditional details as the documment keys etc.

![aws_appsync_5]({{site.url}}/{{site.baseurl}}img/post-assets/appsync_img_5.png)

Something that you need to keep in mind at this step, is that you only have to enter the basepath of your REST, so avoid the `/api/v1/employees` for now.

<br>

## Resolvers

Once we setup our schema and our datasource, it's time to see how can we connect our schema with our datasource. We can do it by attaching a resolver to our query.

Before to setup a resolver we are going to declare a Query in our schema. A Query is a type that specifies how are we going to interact with our data:

{% highlight graphql %}
type Query {
	listAllEmployees: [Data]
}
{% endhighlight %} 

In this case, we have a query that will list all the employees information hence is returning an array of Data type, which is the type that contains all our employees attributes.

And there is where we are going to attach a resolver by clicking on "attach resolver" in the right menu next to our schema editor.
After select our Datasource. We need to put the following code in the Request mapping template:

{% highlight json %}
{
    "version": "2018-05-29",
    "method": "GET",
    "resourcePath": "/api/v1/employees"
}
{% endhighlight %} 

And the following code in the response mapping template:

{% highlight json %}
## Raise a GraphQL field error in case of a datasource invocation error
#if($ctx.error)
  $util.error($ctx.error.message, $ctx.error.type)
#end
## If the response is not 200 then return an error. Else return the body **
#if($ctx.result.statusCode == 200)
    ## We are doing this to map only the data array from our response 
    $util.toJson($util.parseJson($ctx.result.body).data)
#else
    $utils.appendError($ctx.result.body, "$ctx.result.statusCode")
#end
{% endhighlight %} 

In the mapping templates we can modify data, include headers if necesary... using the Apache Velocity syntax.
<br>

## Running our query

Now that we have everything setup is time to fetch the data using our GraphQL, we can start by running the query

{% highlight graphql %}
query{
  listAllEmployees{
      employee_name
  }
}
{% endhighlight %} 

It will retrieve all the names of our employes without fetching any other the extra data:

{% highlight json %}
{
  "data": {
    "listAllEmployees": {
      "data": [
        {
          "employee_name": "Tiger Nixon"
        },
        {
          "employee_name": "Garrett Winters"
        },
        [...]

{% endhighlight %} 

And now is when the client-driven approach makes sense as our client is the one that is telling to the server which data is needed and is the server the one that is taking care to return only the requested information and no in the other way around.
<br>

## Filtering data

So far so good, but we would like now to display only one of our employees filtering by it's id. In order to do that we are going to add another query in our schema adding a filter:

{% highlight graphql %}
type Query {
	...
	listEmployee(id: ID!): Data
}
{% endhighlight %}

and also modify create another resolver for this query but modifying our resource path in our request mapping template in order to include the id that we've put in the filter:

{% highlight json %}
{
    "version": "2018-05-29",
    "method": "GET",
    "resourcePath": $util.toJson("/api/v1/employee/${ctx.args.id}")
}
{% endhighlight %} 

The result of this operation, is that now we can query and filter using GraphQL without making any code modification in our Restful API.

Note that you could use the same query to filter with and without id just applying a simple logic in the request and response templates of the resolver.

{:refdef: style="text-align: center;"}
![aws_appsync_6]({{site.url}}/{{site.baseurl}}img/post-assets/appsync_img_6.jpeg)
{: refdef}

Not really, GraphQL is much more than this, are mutations, complex filtering, nested filtering, operators... But definetily this is a good starting point!


Have a nice week!