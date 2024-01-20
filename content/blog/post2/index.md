---
title: Setting up Jeykll to build with AzDOS
date: 2018-11-18
authors:
  - Stephen Valdinger: author.jpeg
---

I spent the better part of the weekend exploring the various ways I could leverage CI/CD to streamline getting content that I've written onto my blog. This post will be about my journey, and hopefully provide a useful resource to you, should you choose to do something similar.

# Goals and Challenges

When I write content, I want a couple of things to happen.

- I want Jeykll to build on my own schedule.
- I want to distribute the link to content as fast as possible.

Some challenges I wished to overcome were:

- If I had more than one blog post, I didn't want them all to publish at once.
- I needed to figure out a way to hook into the Twitter API.

## Setting up Jeykll

This was incredibly simple. I'm using Github pages for my blog, so I simply forked the JeykllNow repo into my own repo named "steviecoaster.github.io". Done. I have a blog. Tweaking the look and feel is as easy as editing the SCSS files to your heart's desire.

## Distributing Content

I wanted a way to not have to babysit my GitHub repo and force Jeykll to build every day. I'm forgetful....it just wouldn't work out. _It's not you, Jeykll, it's me._

The solution to that problem came in the form of Azure DevOps Pipelines. Pipelines enable you to schedule builds to happen when you want. I opted for every morning at 8am.

## Integrating Twitter

Part 2 of distributing content was getting a link on my Twitter feed automagically. The Azure DevOps Marketplace has a Twitter extension that you can add to your pipeline. This step will require a Twitter Developer account (Free of charges), such that you can get a couple API keys and access tokens needed by the extension.

### Wiring up AzDOS

First thing's first. You'll need a couple Files added to your Github repo. An AzurePipeles.yaml file, and a Powershell build script. I'll provide you copies here that you can download and add to your repos and modify to suit.

- [Azure Yaml](https://gist.github.com/steviecoaster/f1000ee0bf37fe18b7f34ccd57da3830)
- [Powershell Build Script](https://gist.github.com/steviecoaster/92d6b33c26f105a8b9cd4569a3078f45)

To get started with automated builds, head over to [Azure Devops](https://azure.microsoft.com/en-us/services/devops/?nav=min) and Sign-In or Create a Free account. Pipelines are completely free for open source projects.

### Creating the project

Once logged into the Interface, Create a new Project and name it the same as your github repo (steviecoaster.github.io in my instance). This is important later for your build script steps.

### Creating the Pipeline

![New Pipeline Creation](Pipelines_GithubAccountLink.png)
Click on Pipelines > Builds and build a New Pipeline. Use an Empty project. Select Github as your Source and walk through the steps to setup your Service connection, and select your blog repository.

![Empty Project Select Screen](/images/AzDOSJeykll/Pipelines_EmptyJob.png)
On the next screen Setup an Empty project. This will allow us to add the steps we need to add. Don't worry too much about the YAML, I'll have a link at the end of this to a working example that you can modify with specific info to your Pipelines.

![Naming and Pool Selection](/images/AzDOSJeykll/Pipelines_NameAndPool.png)
Setup the Name of the Pipeline as you wish here and Set the Agent Pool to 'Hosted Windows Container'. This is needed for the Twitter integration later.

### Powershell Build Step

![Adding Powershell Task](/images/AzDOSJeykll/Pipelines_AddPowershellTask.png)
Click the '+' icon on the Agent Job 1 tile and Search for Powershell. Hover over Powershell (Run a Powershell script on Windows, Linux, MacOS), and Select Add.

![Powershell Task Options](/images/AzDOSJeykll/Pipelines_PowershellTaskOptions.png)
Select Powershell Tile and Give it a name. For the filename browse and Select the Invoke-PagesBuild.ps1 file from the repo you created earlier. (If you haven't yet, do it now and then complete this step.)

In the Arguments Field of the Powershell step enter $(GithubKey). Don't worry, we will set this up later.

Expand the Advanced tab and select Fail on Standard Error

That's it! Powershell step is now configured.

### Twitter Build Step

![Twitter Task Creation](/images/AzDOSJeykll/Pipelines_AddTwitterTask.png)
Click the + sign on Agent Job 1 again, and this time search for Twitter. Use the one called "Twitter on build and release". If you haven't added it yet, the option to install it first will be available, which will open in a new tab.

Click Add once it is installed and then select the New step.

![Twitter configuration overview](/images/AzDOSJeykll/Pipelines_TwitterConfigP1.png)
Give it a friendly name

![Twitter account link overview](/images/AzDOSJeykll/Pipelines_TwitterConfigP2.png)
Click New beside the Twitter: field to setup a new service connection. This window is where you'll give this service connection a name and add your keys/tokens. The fields are a bit weird so be sure to use the Consumer Key for API, and Consumer Secret for API Secret, and then the Access Token and Secret for the other two fields. Click OK once you have those filled in. Click Add. You should now see the friendly name of your new connection in the drop down menu. If you don't, hit the refresh button next to it, then select it.

In the message field, type the text of the tweet you wish to send. I use a special variable to provide the URL to the post. It's called $(BlogPostTitle) and is created as part of the Powershell Build in Step 1.

### Setting up variables.

Go to your Github [settings](https://github.com/settings/tokens) page and generate a new token. Give it a name and grant control to your repo and admin:repo_hook. Be sure to save this key somewhere, as once displayed, when you leave you will no longer be able to view it. 

![Secret Variable Configuration](/images/AzDOSJeykll/Pipelines_VariableSecret.png)
On the Variables tab of your Azure Pipeline click Add. Give the variable a name (I used GithubKey, and that's included in the ps1 you downloaded above). Paste in the token from Github for the value, and scroll right and click the Lock icon to make it secret.

### Scheduling your pipeline

![Trigger setup](/images/AzDOSJeykll/Pipelines_TriggerSetup.png)
If you'd like the schedule your pipeline to build, click on the Triggers tab. Click "Add +" on the Scheduled section and configure when you would like builds to execute.

### Final thoughts

Wow. What a ride! If all went well, you should now have a working Azure Pipeline! I wanted to mention briefly scheduling posts to happen in the future. If you'd like the take advantage of that, ensure the following is set in your _config.yml file in your repo.

```yaml
future: false
timezone: America/New_York #Set this to your local timezone
```

The following block should be at the top of your post md files:

```yaml
---
layout: post
title: Using Toast notifications in Powershell
date: 2018-11-16 #Change this to some time in the future to schedule to post later
categories: [powershell]
tags: [powershell,notifications,alerting]
---
```

Just make sure the date is set to the day you wish for the post to show up on your blog. When your build runs, Jeykll will ignore those posts if the current date does not match the date in your md file.

Good luck, and happy blogging! If you have questions/comments/etc please feel free to reach out to me on Twitter @steviecoaster! Thanks for reading!