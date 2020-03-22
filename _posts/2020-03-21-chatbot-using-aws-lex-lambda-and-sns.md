---
layout: post-sidebar
date: 2020-03-21
title: "Build a Chatbot to send text messages with AWS Lex, AWS Lambda and SNS"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 25
feature_image: feature-chatbot
show_related_posts: true
square_related: recommend-coding
---

<br>
You've probably also noticed that most online services now use chatbots to help customers with their queries during the COVID-19 pandemic.
So I've wondered why not create my own bot.

One of the things I enjoyed the most as a developer is the ability to combine different technologies I'm learning and connect them together to create an E2E product.

As part of my professional job, I have been working on a project using AWS Lambdas and also part of the project is to send transactional text messages to customers during the authentication process using AWS Cognito - SNS integration.

Combining few components, I have created a chatbot that sends SMS to different people. This is just a demonstration of what you can achieve in **15 minutes**, and it gives you an idea of how easy it is to start your MVP and test your idea.


<br>
{:refdef: style="text-align: center;"}
![chatbot_1]({{site.url}}/{{site.baseurl}}img/post-assets/chatbot_1.jpg)
{: refdef}
<br>


## Building our chatbot

Our chatbot will fulfill a social function, it will send text messages to our friends so that we can keep our friendship alive, even if we are too lazy to text our friends.

In this step we are going to use AWS Lex, which is a service for building conversational interfaces into any application using voice and text.

Let's start with AWS Lex by selecting a custom bot; we can select the name, the output voice and even if we want to use sentiment analysis (Amazon Lex uses Amazon Comprehend to perform sentiment analysis):

<br>
{:refdef: style="text-align: center;"}
![chatbot_2]({{site.url}}/{{site.baseurl}}img/post-assets/chatbot_2.png)
{: refdef}
<br>

Now we will tell our bot what we want it to do for us, its mission.

## Intents

The **intent** is the objective that the client has in mind, what the client is looking for. In our case, the client is looking to send text messages.
We have to define a set of instructions that our client will probably ask our bot so that our bot can learn and identify intentions.
Let's start with a simple and direct one: "I want to send a text message"

{:refdef: style="text-align: center;"}
![chatbot_3]({{site.url}}/{{site.baseurl}}img/post-assets/chatbot_3.png)
{: refdef}

After that, we probably need some additional details, like who the message is for and, for example, the type of message. The chatbot needs to get that information from us with questions or analyze our phrases to extract that information:

Those pieces of information are called **slots**:

{:refdef: style="text-align: center;"}
![chatbot_4]({{site.url}}/{{site.baseurl}}img/post-assets/chatbot_4.png)
{: refdef}

We can associate a question to each slot, a question that is only meant to solve that slot. So it could be something like "Who is this message for?" if we want to get the name of the person for whom the message is.

And these slots can be something specific like a name or something included in a set of values:

{:refdef: style="text-align: center;"}
![chatbot_5]({{site.url}}/{{site.baseurl}}img/post-assets/chatbot_5.png)
{: refdef}

Once we have defined our slots, we can refine our questions to capture a much information as possible without iterating over the questions.

{:refdef: style="text-align: center;"}
![chatbot_6]({{site.url}}/{{site.baseurl}}img/post-assets/chatbot_6.png)
{: refdef}

## Building our lambda

The chatbot will manage all the conversation part with the customer asking questions until all the slots are fulfilled, once that happens we need a service to send the sms messages. That service will be a serverless application written in nodeJS; so AWS lambda is perfect for that:  

{% highlight javascript %}
let AWS = require('aws-sdk');
const sns = new AWS.SNS();


// Function that close the dialog to send it back to the chatbot

function close(sessionAttributes, fulfillmentState, message) {
    return {
        sessionAttributes,
        dialogAction: {
            type: 'Close',
            fulfillmentState,
            message,
        },
    };
}

// Function that send sms and contains the logic

function dispatch(event, callback) {

    var phoneNumber;
    var messageText;

    if (event.currentIntent.slots.messageType != null) {
        // TODO: Ideally we should stract those messages from a DB like DynamoDB
        switch (event.currentIntent.slots.messageType.toUpperCase()) {
            case 'FUNNY':
                messageText = "This #coronapocolypse episode of Black Mirror sucks.";
                break;
            case 'ROMANTINC':
                messageText = "I feel very lucky to be with you!";
                break;
            default:
                messageText = "Be strong, things will get better. It might be stormy now, but rain doesn’t last forever.";
        }
    }
    else {
        callback(null);
    }


    if (event.currentIntent.slots.name != null) {
        // TODO: Ideally we should stract the phone numbers from a DB like DynamoDB
        if (event.currentIntent.slots.name.toUpperCase() === "JAIME") {
            phoneNumber = "+44792*******";
        }
        else if (event.currentIntent.slots.name.toUpperCase() === "DAVID") {
            phoneNumber = "+44792*******";
        }
        else {
            callback(null);
        }
    }
    else {
        callback(null);
    }


    let receiver = phoneNumber;
    let message = messageText;

    let isPromotional = true;

    console.log("Sending message", message, "to receiver", receiver);

    sns.publish({
            Message: message,
            MessageAttributes: {
                'AWS.SNS.SMS.SMSType': {
                    DataType: 'String',
                    StringValue: 'Promotional'
                },
                'AWS.SNS.SMS.SenderID': {
                    DataType: 'String',
                    StringValue: "Jaime"
                },
            },
            PhoneNumber: receiver
        }).promise()
        .then(data => {
            console.log("Sent message to", receiver);
            callback(null, data);
        })
        .catch(err => {
            console.log("Sending failed", err);
            callback(err);
        });


    const sessionAttributes = event.sessionAttributes;
    callback(close(sessionAttributes, 'Fulfilled', { 'contentType': 'PlainText', 'content': `Okay, I have sent the message` }));

}

// Main function
exports.handler = (event, context, callback) => {
    try {
        dispatch(event,
            (response) => {
                callback(null, response);
            });
    }
    catch (err) {
        callback(err);
    }
};

{% endhighlight %} 
<br>
Our application logic is quite simple right now, but by adding a simple database you can make a huge improvement in the app.
Navigate again to AWS Lex and select your lambda as a fulfillment to resolve the customer intent:

{:refdef: style="text-align: center;"}
![chatbot_7]({{site.url}}/{{site.baseurl}}img/post-assets/chatbot_7.png)
{: refdef}
<br>

## Testing the chatbot
<br>
We can test our chatbot directly from the AWS Lex web console, and we can use our keyboard or our mic to maintain a conversation because as we said Amazon Lex is a service for building conversational interfaces into any application using voice and text.

We can go directly with a straight request that contains all the information or we can maintain a conversation with the bot and he will fulfill the slots based on our responses. 

Let's give it a try:

{:refdef: style="text-align: center;"}
![chatbot_8]({{site.url}}/{{site.baseurl}}img/post-assets/chatbot_8.png)
{: refdef}

And it works perfectly fine!

{:refdef: style="text-align: center;"}
![chatbot_9]({{site.url}}/{{site.baseurl}}img/post-assets/chatbot_9.jpg)
{: refdef}

Happy sunday!