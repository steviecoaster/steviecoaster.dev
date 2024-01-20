---
layout: post
Title: Use Universal Automation for Choco Package Internalizer
date: 2020-01-07
categories: [powershell,chocolatey,universalautomation,universaldashboard,dev]
tags: [powershell,devops,development,chocolatey,universalautomation]
comments: true
---

So [@adamdriscoll](https://twitter.com/adamdriscoll) created this new amazing product called Universal Automation. It's a PowerShell job scheduling platform built on top of his awesome Universal Dashboard product. I was fortunate enough to be invited into the Private Beta, so I've been playing with it off and on over the last couple of weeks. I had the idea this evening to see "Can I use this to do Chocolatey Package Internalization?" We have documentation on setting it up in Jenkins over [here](https://chocolatey.org/docs/how-to-setup-internal-package-repository). Note, this is a business feature, so if you are running Open Source or have a Pro License, I'm sorry, this blog post probably won't be much help, other than showing off how _awesome_ Universal Automation is.

# The Setup

Since I use vagrant, it was really quick for me to spin up an environment. I keep a copy of our [chocolatey test environment](https://github.com/chocolatey-community/chocolatey-test-environment) handy, and I've updated it to run on Server 2019. So using my [Vagrantey](https://github.com/steviecoaster/Vagrantey) module I just did a quick `Start-VagrantEnvironment -Environment Choco-Test` and brought up a box.

Next I installed a couple of choco packages:

```powershell
choco install vscode vscode-powershell googlechrome nexus-repository -y
```

I configured my nexus repository for chocolatey, and then configured chocolatey to talk to it.

After I had that work completed I installed the latest build of Universal Automation onto the box. Unfortunately, I can't give much detail there, you'll have to wait for the public beta, but I can say it's as simple as importing a couple modules, starting up a server and dashboard, and you are off to the races.

# Turning scripts into a module

I decided that it would be far easier to have the scripts we provide in a module, it's just much cleaner that way. So whipped up a quick ChocoInternalizer module by simply copying the scripts from above into a folder named `ChocoInternalizer`. Then in VSCode I whipped up a quick psm1 and psd1 file, made sure my cmdlets were proper functions, and did a quick test with `Import-Module C:\Git\ChocoInternalizer\ChocoInternalizer.psd1` and verified all my cmdlets showed up with `Get-Command -Module ChocoInternalizer`.

One quick note, I did make a slight adjustment to the scripts in that in `ConvertTo-ChocoObject` I changed the split to be on the '\|' character, and modified other scripts to use the `-r` flag of choco to limit the output to something a little easier to digest. You absolutely _don't_ have to do that, I just like to mess with [@pauby](https://twitter.com/pauby).

With that good to go, I set off to do the work in Universal Automation.

# Getting things going in UA

The documentation has us start a UA Server, and then a dashboard to access all of its goodness at http://localhost:10001, so I did just that, opened up Chrome, and was welcomed by the UA Homepage. It looked something like this, without all the history

![UA Homepage](./images/UA/UA_Homepage.jpg)

## Adding the Script

1. Universal Automation is all about scheduling scripts to run. Click on The Scripts tab, and at the bottom click "New Script".

2. On the General Tab give it a nice Name and Description, future you will thank present you for being descriptive here.

3. Click over to the Script tab. Here is where you define the Powershell Script you want to run. I used the following:
```powershell
Import-Module C:\Git\ChocoInternalizer\ChocoInternalizer.psd1 -Force
Get-UpdatedPackage -LocalRepo 'http://localhost:8081/repository/choco-local' -LocalRepoApiKey $SuperSecretKey -RemoteRepo 'https://chocolatey.org/api/v2'
```
* Don't worry about that `$SuperSecretKey` bit, we'll get to that in a moment

4. Click on the Finalize tab and give a good commit message. UA uses git internally to track scripts, or you can point to your own Git Repository if you wish.

5. Click Create Script

## Scheduling the script

You probably want to schedule this to run at specific intervals. You can do so in UA using either Cron syntax, or he's  also got a dumby proof Easy-Mode that just lets you select from a pre-defined list of intervals.

To schedule you script do the following:

1. Click on the script that you just created.
2. Click the "Schedule" Button in the upper-right side of the window.
3. Click the "Easy Schedule" tab and select an option.
4. Click "Submit". You can verify the schedule by clicking on the Schedule tab after the window closes.

## That $SuperSecretKey

Click on the "Universal Automation" banner to go back to the home page. From there do the following:

1. Click on the Variables tab.
2. Click "New Variable"
3. Give it a name, in my case SuperSecretKey, and a value, which is the API key for my local repository server.
4. Click "Submit"

## Let's Make Some Magic!

Ok. We are _done_. This thing is ready to run! Click on the Scripts tab and press the "> Run" button. Give it a bit of time, depending on your package count, this might take a minute.

Once it completes you can click on the "Past Jobs" tab from the home page, click the script you just ran, and then select the "View" button to see the output.

## Wrap-up

That's it! We just setup packages to internalize from the community repository in just a few short minutes. I think it took me around an hour to get everything setup the first time as I was installing modules and writing a module and tinkering with things a bit. I encourage you to checkout Universal Automation when it enters public beta next week. It's a really powerful tool to centralize a lot of your code that might be running across different systems for "reasons" that don't need to be "reasons" anymore! Thanks for reading!