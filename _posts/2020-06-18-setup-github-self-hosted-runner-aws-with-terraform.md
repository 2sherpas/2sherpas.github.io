---
layout: post-sidebar
date: 2020-06-18
title: "Setup GitHub self-hosted runner in AWS with Terraform"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 30
feature_image: feature-github-runner
show_related_posts: true
square_related: recommend-coding
--- 

<br>
I know it's weird to start a GitHub article talking about GitLab, but I've been using GitLab for a while as code repository and CI / CD tool, I think that it's great how you can set up a pipeline that you can start using right away using its shared runners.


One month ago, those shared runners started to fail and my pipelines were blocked for around couple of hours, the root cause of the issue was an [incident in Google Cloud](https://status.cloud.google.com/incident/zall/20005) and the GitLab shared runners were deployed in that region, [making the jobs wait in the queue for hours](https://gitlab.com/gitlab-com/gl-infra/production/-/issues/2132). Although these problems occur very infrequently, I realized that I should not trust all of my deployments to runners deployed on infrastructure that I cannot monitor and that therefore implementing self-hosted runners was imperative.

This week, I have been migrating some projects from GitLab to GitHub and with the lesson learned ... I set up a self-hosted GitHub runner on our AWS account using Terraform.

{:refdef: style="text-align: center;"}
![github-runner-0]({{site.url}}/{{site.baseurl}}img/post-assets/github-runner/github-runner-0.jpg)
{: refdef}


## Github actions
<br>
Less than one year ago, Github launched `GitHub actions`. GitHub actions allows you to automate how you build, test, and deploy your projects on any platform.

You can set up a simple workflow simply by creating a yml file in your repo inside the `.github/workflows/` folder.
This is an example of what a simple workflow looks like:

{% highlight yml %}
on:
  push:
    branches:
    - master
jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - name: Hello
        run: echo 'Hello World'
{% endhighlight %}

That produces the following output; a Hello world:
{:refdef: style="text-align: center;"}
![github-runner-1]({{site.url}}/{{site.baseurl}}img/post-assets/github-runner/github-runner-1.png)
{: refdef}

As you can see above, this workflow has a job that prints a simple Hello World but we didn't configure any infrastructure to run the job, we just indicated `runs-on: ubuntu-latest`; Our job is using a virtual environment. 
GitHub offers hosted virtual machines to run workflows. The virtual machine contains an environment of tools, packages, and configurations available for GitHub Actions to use.

## Self-hosted runners
<br>
Those virtual environments are great, but maybe you need more memory to run larger jobs or you just need to keep the control over the servers running those jobs to implement your own monitoring solution.

This is what we are going to set up:

{:refdef: style="text-align: center;"}
![github-runner-2]({{site.url}}/{{site.baseurl}}img/post-assets/github-runner/github-runner-2.png)
{: refdef}

A Github workflow that will run a Terraform deployment that will implement an EC2 machine with a a self-hosted GitHub runner inside it.

Let's start defining these secrets in Github: `AWS_ACCESS_KEY`, `AWS_SECRET_ACCESS_KEY` and `PERSONAL_ACCESS_TOKEN`.

The two secrets with the AWS prefix are the credentials that Terraform will use to deploy on AWS and the other is our GitHub personal access token. 

Why do we need a GitHub personal access token? Each runner needs to be registered with a unique token, that token allows us to un-register the runner also when it is not longer needed, this token can be obtained by navigating to `settings > actions > add-new-runner` or programmatically using the GitHub API and to call that API we need to be authenticated with a GitHub personal access token.


You can get a personal access token navigating to your [settings](https://github.com/settings/tokens) and generating a token with the `repo` scope.

The secrets in our repo should looks like this:

{:refdef: style="text-align: center;"}
![github-runner-3]({{site.url}}/{{site.baseurl}}img/post-assets/github-runner/github-runner-3.png)
{: refdef}


## Terraform files
<br>
With all our secrets in place, we can start creating our terraform files.

Let's start with the `variables.tf` file, this file can be used in the Terraform plans to avoid hard-coding parameters. Create two variables there: the region and the personal_access_token.

{% highlight terraform %}
variable "aws_region" {
  description = "The AWS region to create things in."
  # Ireland
  default     = "eu-west-1"
}
# Use the command line to inject this variable
variable "personal_access_token"{
  description = "personal token"
  default = "replace_this_with_your_token"
}
{% endhighlight %}

These variables can be overwritten by using `terraform apply -var ...` command.

We will inject the token to terraform through the pipeline, so please do not put manually your personal access token in that file; otherwise, your personal access token will be exposed to anyone with access to that repo.

## EC2 and Security group
<br>
The next step is to setup our `main.tf` file (or choose any other name) in the root folder. This file will contain the necessary resources for our github runner.

We can setup the EC2 AMI by name or by using the data block and filtering by name, owner or any other field. In our case, I'll choose the second option:

{% highlight terraform %}
data "aws_ami" "ubuntu_server" {
  most_recent = true
  owners = ["099720109477"]
  filter {
    name = "name"
    values = [
      "ubuntu/images/hvm-ssd/ubuntu-xenial-16.04-amd64-server-20200407",
    ]
  }
}
{% endhighlight %}

We will also need a security group associated to our EC2 instance:

{% highlight terraform %}
resource "aws_security_group" "security_group" {
  name = "sec_group_github_runner"

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
{% endhighlight %}

In this configuration, the EC2 instance is created without Key assigned, which means that you won't be able to connect to this instance using SSH. You can include it if necessary or add it later using AWS console. If you want to use SSH, you will also need to include an ingress rule in the security group to allow connections using port 22 (ssh), but be sure that you restrict the IP range to a VPN or Bastion otherwise anyone will reach your instance using SSH (which is not desirable...)

We also have to include in the file the configuration of our EC2 instance:

{% highlight terraform %}
resource "aws_instance" "my-instance" {
  vpc_security_group_ids = [aws_security_group.security_group.id]
  ami           = data.aws_ami.ubuntu_server.id
  instance_type = "t2.micro" ## Free tier
  user_data = templatefile("scripts/ec2.sh", {personal_access_token = var.personal_access_token})
	tags = {
		Name = "GitHub-Runner"	
		Type = "terraform"
	}
}
{% endhighlight %}
<br>

## Shell Script
<br>
The runner script is loaded in the EC2 instance as startup script (user data). This script runs as soon as the EC2 instance starts and will not run again in case of instance reboot, so if you want to run it again, you need to terminate the instance and recreate it.

Since the script needs our personal access token to make the API calls and we don't want to hard-code this on the script, we need to use a Terraform [templatefile](https://www.terraform.io/docs/configuration/functions/templatefile.html).

The `templatefile` allows to interpolate a variable within our sh file.

{% highlight sh %}
user_data = templatefile("scripts/ec2.sh", {personal_access_token = var.personal_access_token})
{% endhighlight %}

Now, setup a folder called "scripts" and a script called ec2.sh.
Let's review the script line by line.

Inside the script, let's start by insert our runners-command inside the user-data.sh:

{% highlight sh %}
#!/bin/bash
cat <<EOF >/home/ubuntu/user-data.sh
#!/bin/bash
...
{% endhighlight %}

We will also need to install additional dependencies like jq and after that we will extract the runner token by making a call to github API:

{% highlight sh %}
...
wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
chmod +x ./jq
sudo cp jq /usr/bin
curl --request POST 'https://api.github.com/repos/your_user/your_repository_name/actions/runners/registration-token' --header "Authorization: token ${personal_access_token}" > output.txt
runner_token=\$(jq -r '.token' output.txt)
...
{% endhighlight %}

As a result, we will get the runner token, the one that we need to register our runner. 
This token is different from the PAT (personal access token) and expires every 30 minutes. 
Using that token, we will be able to call the github API to register our self-hosted runner:

{% highlight sh %}
...
~/actions-runner/config.sh --url https://github.com/your_user/your_repository_name/ --token \$runner_token --name "Github EC2 Runner" --unattended
~/actions-runner/run.sh
...
{% endhighlight %}

The whole script looks as follow, just paste it in your SH just (remember to replace the Github name and repository with your values):

{% highlight sh %}
#!/bin/bash
cat <<EOF >/home/ubuntu/user-data.sh
#!/bin/bash
wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
chmod +x ./jq
sudo cp jq /usr/bin
curl --request POST 'https://api.github.com/repos/your_user/your_repository_name/actions/runners/registration-token' --header "Authorization: token ${personal_access_token}" > output.txt
runner_token=\$(jq -r '.token' output.txt)
mkdir ~/actions-runner
cd ~/actions-runner
curl -O -L https://github.com/actions/runner/releases/download/v2.263.0/actions-runner-linux-x64-2.263.0.tar.gz
tar xzf ~/actions-runner/actions-runner-linux-x64-2.263.0.tar.gz
rm ~/actions-runner/actions-runner-linux-x64-2.263.0.tar.gz
~/actions-runner/config.sh --url https://github.com/your_user/your_repository_name/ --token \$runner_token --name "Github EC2 Runner" --unattended
~/actions-runner/run.sh
EOF
cd /home/ubuntu
chmod +x user-data.sh
/bin/su -c "./user-data.sh" - ubuntu | tee /home/ubuntu/user-data.log
{% endhighlight %}

<br>

## Deploying our self-hosted Runner
<br>
Finally we are ready to deploy our self-hosted runner and why not use Git-Hub actions for that? Just include these steps in your workflow yml:

{% highlight yml %}
    - name: Terraform Init
      run: |
        terraform init
      env:
        AWS_ACCESS_KEY_ID:  ${ secrets.AWS_ACCESS_KEY_ID }
        AWS_SECRET_ACCESS_KEY:  ${ secrets.AWS_SECRET_ACCESS_KEY }
        
    - name: Terraform validate
      run: |
        terraform validate
    - name: Terraform Apply
      run: |
        terraform apply -auto-approve -var 'personal_access_token=${ secrets.PERSONAL_ACCESS_TOKEN }'
      env:
        AWS_ACCESS_KEY_ID:  ${ secrets.AWS_ACCESS_KEY_ID }
        AWS_SECRET_ACCESS_KEY:  ${ secrets.AWS_SECRET_ACCESS_KEY }
{% endhighlight %}

This workflow will deploy your runner in AWS EC2 injecting the repo secrets into Terraform:

{:refdef: style="text-align: center;"}
![github-runner-5]({{site.url}}/{{site.baseurl}}img/post-assets/github-runner/github-runner-5.png)
{: refdef}

So after few minutes you should have your self-hosted runner deployed in AWS

{:refdef: style="text-align: center;"}
![github-runner-6]({{site.url}}/{{site.baseurl}}img/post-assets/github-runner/github-runner-6.png)
{: refdef}

Accessible from GitHub

{:refdef: style="text-align: center;"}
![github-runner-7]({{site.url}}/{{site.baseurl}}img/post-assets/github-runner/github-runner-7.png)
{: refdef}

And ready to be used in the example of the beginning of this article, but this time let's tell GitHub that our job should run in our self-hosted runner:

{% highlight yml %}
on:
  push:
    branches:
    - master
jobs:
  job1:
    runs-on: self-hosted
    steps:
      - name: Hello
        run: echo 'Hello World'
{% endhighlight %}

Voila!

{:refdef: style="text-align: center;"}
![github-runner-8]({{site.url}}/{{site.baseurl}}img/post-assets/github-runner/github-runner-8.png)
{: refdef}

Have a great weekend : )

Thanks for reading!