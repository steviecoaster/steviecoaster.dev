---
Title: Use Azure Pipelines to publish Chocolatey Packages
date: 2019-07-24
categories: [powershell]
tags: [powershell,chocolatey,software automation,packaging,choco,azure]
---

Hey folks! Wow, it's been quite the dry spell, eh? Apologies, I've been super busy with my new role with Chocolatey Software. It's involved a lot of Powershell, and the opportunity to go to a few conferences and network with you, the Community. If we've met in person, great! I'm so glad I've got to meet you! If we haven't yet, look out for me at some upcoming events like AnsibleFest, Microsoft Ignite! Orlando, and next year at Powershell Summit to start. Now....onto business.

## Premise

The topic here today is one near and dear to me involving Powershell, CI/CD, and Chocolatey! This post will be a walk-through guide to getting setup to deploy chocolatey packages as code in your environments.

## Prereqs

To be successful here you'll need the following:

- An organization on Azure Pipelines (don't worry, you can sign up for free [here]())
- Chocolatey installed on the server/workstation to run this against
- An Azure pipelines build Agent
- The Chocolatey Extension added to your pipeline from the Azure Pipelines Marketplace

## First things first, install Chocolatey

You can install chocolatey by following the instructions on [chocolatey.org](https://chocolatey.org/install). There's a lot of information there, but the line you'll ultimately be after is this one: 

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

If you are a Chocolatey For Business customer, you've likely already got choco installed, or have it available to install via an internal repository server, so we'll skip that part here.

## Next: An Azure Devops Account

You'll need to sign up for Azure Devops in order to get an organization. You can do that by following the link here to the [Sign Up](https://azure.microsoft.com/en-us/services/devops/) page. I recommend the 'Start Free' link, rather than signing up with Github, so as not to mix things up too much.

## Getting things ready in Azure: Step 1

The first thing to do is Navigate to Security by clicking the circle with your First intial in it, in the top right-hand corner of the azure devops window. 

Once there, click on Personal Access Tokens, and create a new one. Here's you'll have options for expiration time, Scope, and various access levels. Select "All Scops" from the bottom, and give 'Agent Pools' the 'Read & Manage' right. Go ahead and also select 'Read & Execute' for the 'Build' Scope as well, since that is the type of pipeline we will be working on.

Click 'Create'.

*IMPORTANT*: Make sure you keep the PAT key somewhere like Notepad, or in a Password vault, as once you have it copied, you cannot view it again.

## Getting things ready in Azure: Step 2

Next, we'll need to create an Agent pool. Click 'Azure Devops' in the upper left-hand area of your browser window to be taken back to the landing page. In the bottom left-hand corner, select Organization Settings.

From the menu on the left, under Pipelines select 'Agent pools'
Click 'Add Pool' and give it a meaningful name. I used 'Choco'...obviously.

## Getting things ready in Azure: Step 3

Now it's time to add an agent. Since we are using Chocolatey, this is as easy as running the following:

```powershell
choco install azure-pipelines-agent -y --param="'/Token:yourtokenhere /Pool:PoolNameFromStep2 /Url:https://dev.azure.com/yourorgname'"
```

Once you run the above, you should be able to go back into Organization Settings, and under Agent Pool select the Pool you created, and then Agents to view your newly connected Agent.

## Create your package

Create a chocolatey package. Put this new package into source control. I used Github. I'll also give you some sample yaml to include in the repo, which will be used in the next step to create the pipeline.

```yaml
pool:
    name: Choco
trigger:
    branches:
        include:
        -master
        exclude:
        -develop
steps:
-task: gep13.chocolatey-azuredevops.chocolatey-azuredevops.ChocolateyCommand@0
    displayName: 'Chocolatey pack'
    inputs:
        packNuspecFileName: 'yourpackagenuspec.nuspec'
-task: gep13.chocolatey-azuredevops.chocolatey-azuredevops.ChocolateyCommand@0
    displayName: 'Chocolatey push'
    inputs:
        command: push
        pushNupkgFileName: 'yourpackagename.nupkg'
        pushSource: 'http://yourinternalrepo/url'
        pushApikey: $(ChocoApiKey)
        pushForce: true
```

Change the above yaml to suit your needs. The only things you should need to worry about are the file names, the url, the agent pool, and the Apikey variable. More on that variable in the next step.




## Create a pipeline

Back in Azure Devops, if you have not created a Project, do so now. I recommend naming it the same as the package you created.

Inside the project, select Pipelines, and then Builds, and create a new Build Pipeline.
Select the Use the classic editor link at the bottom of the options.
Select Github as your source, and configure any connections it requires, and then select your source repository and default branch (which should be master).
On the next page select Configuration as Code, and select the yaml file from your repository. and save the pipeline.

Let's talk quickly about that ApiKey variable. Under your new Build Pipeline click 'Edit' from the the menu if you've already saved it. 
You'll notice when you edit the pipeline, your yaml file is shown. In the right-hand corner is a 'Variables' button. Click that button, and then select the '+' sign to add a new variable. Give it a meaningful name, and paste the apikey that is used for your repository server. Ensure you select the 'Keep this value secret' checkbox. This will elimate the risk of it being exposed in your yaml and at runtime in logs.

## It's time to run!

If you've been able to follow all of the steps above, you should be good to go. Since you've got your package files ready to go on your repository, you should be able to queue up the pipeline, and let it run. Check it for errors, and correct any that show up. 

If you have any questions, feel free to reach out to me on [Twitter](https://twitter.com/steviecoaster).

