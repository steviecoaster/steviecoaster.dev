---
title: Using Toast notifications in Powershell
date: 2018-11-16
authors:
  - Stephen: author.jpeg
---
Notifications can be quite a bit more than just those annoying pop-ups you see down by your system clock from time to time. When created using Powershell, you can get an _incredible_ amount of functionality out of them.

# Toasts are more than pop-ups

Some of my favorite uses include:

* Giving myself reminders
* Alerting when things happen on remote endpoints
* Alerting other staff of important tasks that need done
* Receiving alerts from monitoring platforms when an issue occurs

## Getting started

There are a couple of ways to create toast notifications on Windows 10. You can install a module from the gallery, or if you are feeling frisky, can't use 3rd party modules, or just plain want too, you can write code using pure .Net classes to create the notifications

## Using a pre-built module

I discovered [BurntToast](https://github.com/Windos/BurntToast) a few months ago and fell in love with all the "stuff" that it unlocked for me to alert on. Written by Josh King, it is a fantastic module if you want to get up and running in a hurry. You can get the module a couple of ways:

Using git:

```powershell
git clone https://github.com/Windos/BurntToast.git
```

Installing from the Gallery:

```powershell
Install-Module BurntToast
```

In its simplest form a toast contains 4 things: A title (Header), a body, an image, and the application name sending it.

Try it:

```powershell
Import-Module BurntToast
$toastParams = @{
    Text = "Hey, this is my first Toast notification"
    Header = (New-BTHeader -Id 1 -Title "My first Toast")
}
New-BurntToastNotification @toastParams
```

To discover all the fun things you can do with the BurntToast module, go exploring:

```powershell
Import-Module BurntToast
Get-Command -Module BurntToast
```



## Using .Net to create toasts

There are a few key pieces to working with .Net to generate a toast notification. For simplicity, I've created an example script, which you can get [here](https://gist.github.com/steviecoaster/aa534a7798d17d93bc080692eb98f5c1), but I'll break it apart and explain it below:

First, you are going to need to load some classes:

```powershell
$null = [Windows.UI.Notifications.ToastNotificationManager, Windows.UI.Notifications, ContentType = WindowsRuntime]
$null = [Windows.Data.Xml.Dom.XmlDocument, Windows.Data.Xml.Dom.XmlDocument, ContentType = WindowsRuntime]
```

You'll also need a toast notification template. Here's a generic one, which you can store in a here-string:

```powershell
  $XmlString = @"
  <toast>
    <visual>
      <binding template="ToastGeneric">
        <text>$Title</text>
        <text>$Message</text>
        <image src="$Logo" placement="appLogoOverride" hint-crop="circle" />
      </binding>
    </visual>
    <audio src="ms-winsoundevent:Notification.Default" />
  </toast>
"@
```

_More information on this template, and other templates can be found [here](https://docs.microsoft.com/en-us/windows/uwp/design/shell/tiles-and-notifications/adaptive-interactive-toasts)_

And finally, put it all together to build and send your new Toast notification:

```powershell
$ToastXml = [Windows.Data.Xml.Dom.XmlDocument]::new()
$ToastXml.LoadXml($XmlString)
$Toast = [Windows.UI.Notifications.ToastNotification]::new($ToastXml)
[Windows.UI.Notifications.ToastNotificationManager]::CreateToastNotifier($AppId).Show($Toast)
```

For some ideas to get you started, you can take a look at the example content from my most recent User Group presentation [here](https://github.com/steviecoaster/RTPSUG7Nov/tree/master/Scripts)