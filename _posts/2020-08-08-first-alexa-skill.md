---
layout: post
title: Build Your Own Alexa Skill!
---

This post goes over how to build a basic Alexa Skill. The tutorial takes you through the process of creating a lambda function, deploying an Alexa skill, and being able to ask Alexa to pick a Star Wars film for you.

#### Prerequisites
- **Note** - An Alexa device is not required for this tutorial but it is fun to test your deployed skill and see it in action!
- Have an AWS account (we will only be using services under the free tier so don't worry about this costing you any 💰).
- Have an account at [developer.amazon.com](https://www.developer.amazon.com) (under the same account your Alexa device is associated with).

## Create a Lambda Function
To get started, we need to create a new lambda function that will act as the endpoint for our Alexa skill to hit when invoked.
1. Head over to the lambda service on the AWS Console and hit "Create Function".
2. To make it easy, there are publicly available libraries to help us get spun up within minutes. Select the "Browse serverless app repository" option and click "alexa-skills-kit-nodejs-factskill":
![_config.yml]({{ site.baseurl }}/images/2020-08-08/create-lambda.png)
3. We are going to keep all the defaults for this project so scroll to the bottom and deploy the lambda function!
![_config.yml]({{ site.baseurl }}/images/2020-08-08/deploy-lambda.png)
4. After a few moments your lambda function should be successfully deployed! You will see it on your lambda dashboard. Go ahead and click on your new function to view the details. Scroll down to the function code and find the object `enData`. You should change the array of `FACTS` to look something like this:
![_config.yml]({{ site.baseurl }}/images/2020-08-08/star-wars-code.png)
Make sure to hit "Save" after making your changes!
5. Lastly, take note of the `ARN` endpoint at the top right of your screen. This is important when we build our Alexa skill in the next section.

## Deploy your Alexa Skill
1. In a new tab, navigate to your account on [developer.amazon.com](https://www.developer.amazon.com). From there,
hit "Developer Console" --> "Alexa Skills Kit". And let's create a new skill:
![_config.yml]({{ site.baseurl }}/images/2020-08-08/create-skill.png)
2. Next, give your skill any suitable name (this will not effect the invocation cue). I went with "StarWarsMoviePicker" for mine. We're also going to go with the "Custom" model for our skill.
![_config.yml]({{ site.baseurl }}/images/2020-08-08/skill-details.png)
3. For our template, we already are using the public repository for the Fact Skill in our lambda function so we're going to use that as our skill template.
![_config.yml]({{ site.baseurl }}/images/2020-08-08/template.png)
Go ahead and continue from this step and create your skill! After a few minutes you will land on your new skill's dashboard.
4. We need to set the invocation name for the new skill (this is what begins the interaction with Alexa on a particular skill). Navigate to the section titled "Invocation" and enter in "star wars cloud" as your skill invocation name and press "Save Model".
5. Before building the model, we need to set the endpoint to our lambda function so it references our Star Wars data when invoked. Under "Endpoint", next to "Default Region", paste in your `ARN` you copied from the last step of the lambda setup above.
6. Save your changes and hit "Build Model". This should take a few minutes but once finished, you'll receive a popup notification in the bottom right of your screen.

## Test your Alexa Skill
1. Now you're ready to test your deployed Alexa skill! Navigate to the "test" tab under your Alexa skill and change the testing enabled environment to "Development". You should then be able to click on the input box and trigger your Alexa skill like so:
![_config.yml]({{ site.baseurl }}/images/2020-08-08/testing-skill.png)

Because we configured our skill's endpoint with our lambda's ARN. Your response will pull from the array of movies we setup in that function. If you have your own Alexa device that was registered under the same account, you should now be able to use the same trigger from the simulator on your physical device!

### Congratulations 🎉
You just built your very own Alexa skill that connects to a custom lambda function to pick a random Star Wars film. I'd say it's time to celebrate by asking Alexa which film you should sit down and enjoy (preferably something between episodes 1-6 🚀🙄).
