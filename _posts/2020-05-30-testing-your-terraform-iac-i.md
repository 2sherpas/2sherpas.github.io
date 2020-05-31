---
layout: post-sidebar
date: 2020-05-30
title: "Testing your Terraform IaC"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 30
feature_image: feature-test
show_related_posts: true
square_related: recommend-coding
--- 

<br>

Moving to a modern cloud like AWS, Azure or Google Cloud has many benefits, one of them is the ability to provision infrastructure with just few clicks, you can automate this task using tools like Terraform, which is a tool to build infrastructure as code.

There are plenty of tutorials on the Internet about Terraform, but most of them skip one of the most important parts of building IaC, the tests.


When I started to work with Terrraform, it was clear to me that I needed to define a robust testing strategy to avoid horror stories like the ones I have read on the internet about [deleting Production tables](https://coderbook.com/@marcus/prevent-terraform-from-recreating-or-deleting-resource/) by mistake. 


{:refdef: style="text-align: center;"}
![terraform_1]({{site.url}}/{{site.baseurl}}img/post-assets/terraform_1.jpeg)
{: refdef}

Let's start by define a simple AWS stack using terraform IaC.

# Terraform IaC
<br>

There are a lot of good tutorials on the internet on how Terraform works, so I'm not going to go into detail on how to build a template but basically the following template implements a [DynamoDB](https://aws.amazon.com/dynamodb/) global table on AWS:

{% highlight terraform %}
# This is the bucket to store our Terraform state
terraform {
  backend "s3" {
    bucket = "2SherpasBlog"
    key    = "terraform.tfstate"
    region = "eu-west-2"
    encrypt = true
  }
}

# AWS Provider
provider "aws" {
  region = "eu-west-2"
}

# DynamoDB table
resource "aws_dynamodb_table" "sherpas_global_table" {
  name              = "sherpas_global_table"
  billing_mode      = "PAY_PER_REQUEST"
  hash_key          = "article_uid"
  stream_enabled    = true
  stream_view_type  = "NEW_AND_OLD_IMAGES"
 
  server_side_encryption {
    enabled = true
  }

  attribute {
    name = "article_uid"
    type = "S"
  }
  replica {
    region_name = "eu-west-1"
  }
}

{% endhighlight %}

That table contains some configuration like server-side encryption, the region where the replica is going to be stored, billing mode....
All of these parameters are important because they can affect our infrastructure costs (billing mode), security (server-side encryption) or our high availability plans (replica configuration).

Most of those things cannot be detected by implementing the infrastructure and running E2E tests, as these parameters are invisible for our consumers. This gives you an idea of why I think a proper testing is mandatory for our Terraform deployments.

The first basic test we will run is by running:

{% highlight sh %}
$ Terraform validate
{% endhighlight %}

Let's consider this as our first test layer, however it only validates that the terraform file is syntactically valid and internally consistent.


## Getting started with Conftest
<br>
Let's start with the lowest level of testing, `Unit Testing`.
A unit is the smallest testable part of any software, in our case those are the our resources and its properties.


{:refdef: style="text-align: center;"}
![unit-tests]({{site.url}}/{{site.baseurl}}img/post-assets/unit-tests.jpg)
{: refdef}

We are going to use [conftest](https://www.conftest.dev/) to run these tests.
 
> Conftest is a utility to help you write tests against structured configuration data. Conftest relies on the Rego language from Open Policy Agent for writing the assertions. You can read more about Rego in How do I write policies in the Open Policy Agent documentation.

In this first unit test, we are going to check if server-side encryption is enabled in our table.

Create a file with .rego extension and paste the following content:
{% highlight rego %}
package main

deny[msg] {
  not input.resource.aws_dynamodb_table.sherpas_global_table.server_side_encryption.enabled == true
  msg = "The server_side_encryption for sherpas_global_table table must be enabled"
}
{% endhighlight %}

The `deny` block contains our test and if any of the conditions inside this block are true, then the test will return the message contained in our` msg` variable.

To check our unit test, we are going to disable server-side encryption of the table and run our tests:

{% highlight sh %}
conftest test -p yourtest.rego terraformfile.tf
{% endhighlight %}

The test will fail and will return the following message:

{% highlight sh %}
 FAIL - terraform.tf - The server_side_encryption for sherpas_global_table table must be enabled
 1 tests, 0 passed, 0 warnings, 1 failure
{% endhighlight %}

That's great, isn't it? We will test all of our properties, making sure that the table is created as intended.

## Are unit tests enough?
<br>
Definitely not, in Terraform our resources can refer to other resources, so the value of some properties is dynamic.

In our example, we only have one resource, so the only thing we are going to verify is that Terraform does not destroy the table.

Terraform destroys and recreates a table every time you make a change to the resource, and destroying a table is not always a good idea, especially in a production environment. Terraform includes a prevent_destroy flag but for [various reasons](https://github.com/hashicorp/terraform/issues/17599) I recommend that you write a test for these scenarios.

{:refdef: style="text-align: center;"}
![drop-table]({{site.url}}/{{site.baseurl}}img/post-assets/drop-table.jpg)
{: refdef}

I'm going to call these tests `pre-integration tests` because at this stage our infrastructure is not deployed on AWS.

Create a new `.rego` file and paste the following content:

{% highlight rego %}
package main

# This array contains the objects that must not be deleted
protected_tables = [
  "aws_dynamodb_table.sherpas_global_table"
]

deny[msg] {
  check_protected_tables(input.resource_changes, protected_tables)
  msg = "[Contract Test] You are trying to delete a protected table"
}

check_protected_tables(resources, disallowed) {
  startswith(resources[i].address, disallowed[_])
  resources[i].change.actions[_] == "delete"
}
{% endhighlight %}

Inside the `deny` block we have a call to a function `check_protected_tables`. 

What this function does is verify the `resource_changes` node, if it detects that there is a delete operation, it will verify that the deletion is not from one of our protected tables. If the function returns true (protected resource) our test will fail.

As I said before, these tests are run after the plan:

{% highlight sh %}
Terraform plan -out "planfile"
{% endhighlight %}

Conftest needs the input in JSON format, so we are going to transform our Terraform plan to JSON by executing the following command:

{% highlight sh %}
Terraform show -json "planfile" > "json_planfile.json"
{% endhighlight %}

Once we have our plan in JSON format, we are ready to run the tests:

{% highlight sh %}
conftest test -p test.rego json_planfile.json
{% endhighlight %}
<br>

## Still not enough?
<br>
Until now, we have tested everything in our local environment without implementing anything in AWS ... so we are facing a clear case of "it works on my machine".

Before deploying our real infrastructure, we must run some 'integration tests' against a clone of our real environment. I'm going to use [Terratest](https://terratest.gruntwork.io/) for that:

> Terratest is a Go library that provides patterns and helper functions for testing infrastructure, with 1st-class support for Terraform, Packer, Docker, Kubernetes, AWS, GCP, and more.

Integration tests will be run along with Terraform apply command. It will create real resources, perform integration tests, and then destroy those resources.

Since we want to prevent the test from destroying the existing resources in your real AWS environment, we will have to make some small modifications to our Terraform file:

{% highlight terraform %}
resource "aws_dynamodb_table" "my_dynamodb_table" {
  name              = var.my_table_name
  billing_mode      = "PAY_PER_REQUEST"
  hash_key          = "user_name"

  attribute {
    name = "user_name"
    type = "S"
  }
}
{% endhighlight %}

As you can see above, we made a little modification to the name of our table, it is no longer a static name, but instead takes the name of a variable in our variables.tf file, our integration test will overwrite the variable with a random name for create a temporary table, run our integration tests and destroy it later.

Create a .go file and paste the following code:

{% highlight go %}
package test

import (
	"fmt"
	"testing"

	awsSDK "github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/service/dynamodb"
	"github.com/gruntwork-io/terratest/modules/aws"
	"github.com/gruntwork-io/terratest/modules/random"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestOurDynamoDbTable(t *testing.T) {
	t.Parallel()

	// Pick a random AWS region to test in. This helps ensure your code works in all regions.
	awsRegion := aws.GetRandomStableRegion(t, nil, nil)

	// Set up expected values to be checked later
  // Here we generate a random name for our testing table
	expectedTableName := fmt.Sprintf("aws-dynamodb-test-table-%s", random.UniqueId())
	expectedKeySchema := []*dynamodb.KeySchemaElement{
		{AttributeName: awsSDK.String("article_uid"), KeyType: awsSDK.String("HASH")}
	}

	terraformOptions := &terraform.Options{
		// The path to where our Terraform code is located
		TerraformDir: "../../../global",

		// Variables to pass to our Terraform code using -var options
    // And this is why we need to parametrize our table name, as we will replace my_table_name variable 
    // By a random name generated in our test (above)
		Vars: map[string]interface{}{
			"my_table_name": expectedTableName,
		},

		// Environment variables to set when running Terraform
		EnvVars: map[string]string{
			"AWS_DEFAULT_REGION": awsRegion,
		},
	}

	// At the end of the test, run `terraform destroy` to clean up any resources that were created
	defer terraform.Destroy(t, terraformOptions)

	// This will run `terraform init` and `terraform apply` and fail the test if there are any errors
	terraform.InitAndApply(t, terraformOptions)

	// Look up the DynamoDB table by name
	table := aws.GetDynamoDBTable(t, awsRegion, expectedTableName)

	assert.Equal(t, "ACTIVE", awsSDK.StringValue(table.TableStatus))
  // With asserts we check that the table has been created with the expected properties
	assert.ElementsMatch(t, expectedKeySchema, table.KeySchema)

}

{% endhighlight %}
You can follow what the code actually does by reading the comments embedded in the code.
We are now ready to run our integration test by running:

{% highlight sh %}
go test -v -run TestOurDynamoDbTable
{% endhighlight %}


And after a few seconds we will see how Terratest begins to spin up our infrastructure, run tests and destroy it.

{% highlight sh %}
=== RUN   TestOurDynamoDbTable
=== PAUSE TestOurDynamoDbTable
=== CONT  TestOurDynamoDbTable
TestOurDynamoDbTable 2020-05-30T22:57:36+01:00 region.go:91: Using region eu-west-2
TestOurDynamoDbTable 2020-05-30T22:57:36+01:00 retry.go:72: terraform [init -upgrade=false]
TestOurDynamoDbTable 2020-05-30T22:57:36+01:00 logger.go:66: Running command terraform with args [init -upgrade=false]
TestOurDynamoDbTable 2020-05-30T22:57:36+01:00 logger.go:66: 
TestOurDynamoDbTable 2020-05-30T22:57:36+01:00 logger.go:66: Initializing the backend...
...
{% endhighlight %}

You will notice that those tests are slower than the previous ones!

## Final toughs
<br>
So far so good, we have done 3 layers of tests: unit tests, pre-integration tests and integration tests. We can include these tests as individual steps in our deployment pipeline to automate the entire process and avoid manual errors

You don't need to follow this test strategy and even if you do follow it, keep in mind that your code infrastructure will not be perfect until it has been used and repeatedly failed, but the ability to design your own test strategy will improve your ability to detect the errors, correct them and increase the quality of each implementation.

Thanks for reading!