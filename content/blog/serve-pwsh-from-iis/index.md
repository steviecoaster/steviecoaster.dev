---
title: Serve PowerShell Scripts Via IIS
date: 2024-12-11
authors:
  - Stephen: author.jpeg
---

Often times I get into something at work that makes me put my thinking cap on to solve a problem. This blog post will outline one of those times. This came into being a few years ago, but now seems like just about as good a time as any to write about it.

At Chocolatey we provide a couple different ways for new and prospective customers to get  up and running rapidly with our Suite of products by way of either an Azure Marketplace Image, or by following what we lovingly called our 'Quickstart Environment' Guide.

Both of these environments secure the various web interfaces that make things tick with an SSL certificate. When you opt to use the Quickstart Environment you have the option, however unrecommended, of using a self-signed SSL certificate.

Now, if you are reading this, you _likely_ know enough about PKI to know that visiting a dentist without novacaine is more fun. And thus...our story begins.

## Why Self-Signed certs aren't great

Client machines don't _inherently_ trust them, by default. This causes all kinds of fun issues, like having to click extra buttons in a browswer when visiting a website secured by one, or by Chocolatey CLI just _not_ being able to communicate with your repository because it can't trust the connection.

We need a solution. Enter....IIS and PowerShell!

## Establishing trust

You can "make" Windows trust a self-signed certificate by placing a copy of the self signed certificate into the TrustedPeople store.
You can think of the TrustedPeople store like your phone's contacts. You add the contacts that you know, and when they get ahold of you, you intrinsictly trust it is the person your phone says it is without doing any other verification (like using a Certificate Authority to validate the certificate).

## Getting Cert with PowerShell

For my particular use-case here at Chocolatey, I need to get a copy of the server's SSL certificate so that I can place it into the TrustedPeople store of a client PC trying to connect to the Chocolatey services hosted by the server. In a stroke of good luck, I happen to have friends who are _much_ smarter than I who have already solved this particular problem with some pretty slick PowerShell. Thanks to my friend [Mathias Jessen](https://bsky.app/profile/iisresetme.bsky.social) for the `Get-RemoteCertificate` function in the script [here](https://github.com/chocolatey/choco-quickstart-scripts/blob/main/scripts/Import-ChocoServerCertificate.ps1) that does the dirty work of importing this self-signed certificate.

### Getting IIS configured

Now that we have script which will connect to our server and trust our certificate, we need a easy way to allow a client to execute it. PowerShell's `Invoke-Expression` cmdlet allows for the invocation of string data, the contents of a ps1 file downloaded from a url for example. 

And here, ladies and gentlemen, is where we employ IIS to our benefit.

### Configuring IIS

To get started, install the IIS role, including the Management features. You can do that with PowerShell with `Install-WindowsFeature -Name Web-Server -IncludeManagementTools`. Once you have IIS installed we need to configure it. You can see how we do it for our Guide [here](https://github.com/chocolatey/choco-quickstart-scripts/blob/main/scripts/New-IISCertificateHost.ps1). It's a pretty simple script, and can be easily modified to work for your use-case as well.

The real "secret sauce" to getting IIS to serve the script for us is in this line of code:

```powershell
Add-WebConfigurationProperty -PSPath IIS: -Filter system.webServer/staticContent -Name "." -Value @{ fileExtension = '.ps1'; mimeType = 'text/plain' }
```

*Note* that the above adds the MIME type _globally_. If you wish to limit the MIME type just to a single site, use the following replacing $Sitename

```powershell
Add-WebConfigurationProperty -PSPath IIS:\Sites\$Sitename\ -Filter system.webServer/StaticContent -Name '.' -Value @{fileExtension = '.ps1' ; mimeType = 'text/plain' }
```

With that MIME type, all we need to do to trust our certificate on a client is this (Replace $fqdn and the script name as appropriate):

```powershell
Invoke-Expression ([System.Net.WebClient]::new().DownloadString("http://$($Fqdn):80/Import-ChocoServerCertificate.ps1"))
```

## Wrap Up

And there we have it. Hosting PowerShell in IIS isn't so bad, right? If you'd like to see me demonstrate this fully and run a user group, let me know. I'd be more than happy to spend time with you! Thanks for reading, and Happy Scripting!
