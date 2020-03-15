---
layout: post-sidebar
date: 2020-03-15
title: "Quarkus - Supersonic Subatomic Java"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 25
feature_image: feature-quarkus
show_related_posts: true
square_related: recommend-coding
---

<br>
It is Sunday morning and many people have memory problems trying to remember their last night. Java developers also have memory issues, JVM memory issues, to be more concise.

The JVM is a beast, and that beast is very powerful, but it swallows memory, it smells memory like a T-REX smells a good steak, so developers have to spend a lot of time to profile the JVM and parameters like `-XX:+UseSerialGC`, `-Xms..`, `-Xmx..` .. sounds very familiar to any Java developer, however sometimes that is not enough:
<br>
{:refdef: style="text-align: center;"}
![quarkus-1]({{site.url}}/{{site.baseurl}}img/post-assets/quarkus-1.jpg)
{: refdef}

<br>
## Quarkus - Supersonic Subatomic Java!
<br>
What does it means? Quarkus has been designed thinking in serverless, containers and cloud based environments and this is it presentation letter:

> Amazingly fast boot time, incredibly low RSS memory (not just heap size!) offering near instant scale up and high density memory utilization in container orchestration platforms like Kubernetes

But if we want to achieve outstanding results we have to use it in combination with GraalVM:


{:refdef: style="text-align: center;"}
![quarkus-2]({{site.url}}/{{site.baseurl}}img/post-assets/quarkus-2.jpg)
{: refdef}

<br>
## Getting ready
<br>
For this example, we are going to use a PostgreSQL database of employee data, and since I have no dataset available, I am going to use the `generate_series` utility to load the DB:

{% highlight sql %}
// Create the table structure
CREATE TABLE Employee (id int, name varchar(255), surname varchar(255), department varchar(255));

// Load 1000 records in the DB
INSERT INTO Employee (
    id, name, surname, department
)
SELECT
    i,
    md5(random()::text),
    md5(random()::text),
    (SELECT x[ 1 + ( (random() * 5)::int) % 5 ]
        FROM (
            select '{HHRR, Marketing, Finance, IT, Accounting}'::text[]
    ) AS Department(x))
FROM generate_series(1, 30000) s(i);
{% endhighlight %} 

Once we have our data loaded successfully, our starting point with Quarkus will be the [Quarkus initializer](https://code.quarkus.io/), quarkus has a web-based initializer like the well-known spring initializer, so we are going to choose the name of our package and pick the dependencies:

{:refdef: style="text-align: center;"}
![quarkus-3]({{site.url}}/{{site.baseurl}}img/post-assets/quarkus-3.jpg)
{: refdef}


For this specific example we are going to use the following dependencies `quarkus-jdbc-postgresql`, `quarkus-hibernate-orm-panache` and `quarkus-resteasy-jsonb`, you can select them directly in the initializer of install them later from the terminal by running:

{% highlight plaintext %}
./mvnw quarkus:add-extension -Dextensions="io.quarkus:quarkus-jdbc-postgresql"
..
{% endhighlight %} 

<br>
## Start coding
<br>
Once our zip is downloaded, we can unzip it and start encoding. Our project will have an end point to return a list of employees by department, this will return a huge JSON with over 1000 records because the idea of this experiment is to test how the quarkus behaves with a large amount of data.

## Our controller

This is how our controller will looks like:

{% highlight java %}
@Path("v1")
@ApplicationScoped
@Produces("application/json")
@Consumes("application/json")
public class ExampleResource {

    @GET
    @Path("/employees/department/{department}")
    public List<Employee> getEmployees(@PathParam("department") String department) {
        return Employee.findByDepartment(department);
    }
}
{% endhighlight %} 


We could also build a Response object to return a custom status code:


{% highlight java %}
@GET
@Path("/employees/department/{department}")
public Response getEmployees(@PathParam("department") String department) {
    return Response.ok(Employee.findByDepartment(department)).build();
}
{% endhighlight %} 

Or even, install the `quarkus-spring-web` plugin and use the spring annotations:

{% highlight java %}
@RestController
@RequestMapping("v1")
public class ExampleResource {

    @GetMapping("/employees/department/{department}")
    public Response getEmployees(@PathVariable("department") String department) {
        return Response.ok(Employee.findByDepartment(department)).build();
    }
}
{% endhighlight %}

And one of the good things about quarkus is the hot-swap feature, every time we make a change quarkus stops and starts the server almost instantly.

{:refdef: style="text-align: center;"}
![quarkus-4]({{site.url}}/{{site.baseurl}}img/post-assets/quarkus-4.jpg)
{: refdef}
<br>
##  Simplified Hibernate ORM with Panache
<br>
Our program logic is quite simple, just a query againts our PostgreSQL DB so it's a nice scenario to use panache, according to it's documentation `Hibernate ORM with Panache focuses on making your entities trivial and fun to write in Quarkus.` so basically it's combining entities and repositories in a single class. let's see Panache in action:

{% highlight java %}
@Entity
public class Employee extends PanacheEntity {

    public Long id;
    public String name;
    public String surname;
    public String department;

    public static List<Employee> findByDepartment (String department) {
        return list("LOWER(department) = ?1",department.toLowerCase());
    }
}
{% endhighlight %}

By extending `PanacheEntity` we automatically have all the common operations like `.findById(id)`, `findAll()`... plus our custom queries. In our case we've defined a custom query that list employees by departments.

And that's all, no additional classes for repositories needed.

We are going to write a simple test before to release the app, and we are good to go!:

{% highlight java %}
@QuarkusTest
public class ExampleResourceTest {

    @Test
    public void testHelloEndpoint() {
        given()
          .when().get("v1/employees/department/IT")
          .then()
             .statusCode(200);
    }
}
{% endhighlight %}


Once we have tested the application, we can package it running `mvn package -Pnative` this will generate native image.
A native image contains the VM plus our code, so we can run it and check if it's really a supersonic app:

{:refdef: style="text-align: center;"}
![quarkus-5]({{site.url}}/{{site.baseurl}}img/post-assets/quarkus-5.jpg)
{: refdef}


The application started in 0.028s, so yes, it's really supersonic!

Happy sunday!

