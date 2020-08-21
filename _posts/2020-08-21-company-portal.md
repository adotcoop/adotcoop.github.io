---
layout: post
title:  "Deploying Company Portal without using Microsoft Store for Business"
date:   2020-08-21 19:29:54 +0100
--- 

Microsoft are positioning the Company Portal app as a [cross-platform portal][microsoftblog] for Microsoft Endpoint Manager. This will dramatically improve the user experience on co-managed devices. 

It surprised me that there's no easy way to install Company Portal on a device automatically. 

Windows Store apps can only be made available in Intune. We have to __ask the user to install Company Portal manually.__

This is horrible.

![alt text](/assets/company-portal.png "Company Portal in the Microsoft Store")

A quick search confirms this and that the preferred method is to integrate Intune with Microsoft Store for Business (MSfB)

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Yep. It’s not deployed by default (there are some scenarios where self-service user portals are not desired). I recommend connecting your Intune tennent to Microsoft Store For Business. From MSFB you can acquire the Company Portal app, sync it to Intune and deploy (as required)</p>&mdash; Micro-Scott 🔎 (@Scottduf) <a href="https://twitter.com/Scottduf/status/1131403235732754433?ref_src=twsrc%5Etfw">May 23, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

MSfB [requires a Global Admin][mfsbsetup] to do the initial setup. Depending on your organisation this could be challenging to action depending on your procedures and team structure. Intune Administrators do not have the permissions to perform the initial set up of MSfB.

## Stuck in limbo

In a [January 2020 article][zdnet] Mary Jo Foley had this to say about MSfB

> Microsoft is continuing to try to clean up its digital app-store mess. Its latest planned move, according to my contacts: Get rid of the Microsoft Store for Business and Microsoft Store for Education.

Greg Shields (author of the [excellent Pluralsight Intune courses][pluralsight]) lays this out better than I can

<blockquote class="twitter-tweet"><p lang="en" dir="ltr"><a href="https://twitter.com/hashtag/MicrosoftIntune?src=hash&amp;ref_src=twsrc%5Etfw">#MicrosoftIntune</a> question: If the Microsoft Store for Business is soon to be deprecated, where do I download the Intune Company Store app? Let&#39;s say I need to require its installation on auto-enrolled, domain-joined clients? Regular Microsoft Store apps can&#39;t be made required.</p>&mdash; Greg Shields (@concentratdgreg) <a href="https://twitter.com/concentratdgreg/status/1246133337200062464?ref_src=twsrc%5Etfw">April 3, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Another <a href="https://twitter.com/pluralsight?ref_src=twsrc%5Etfw">@Pluralsight</a> author (H/T <a href="https://twitter.com/gepeto42?ref_src=twsrc%5Etfw">@gepeto42</a>) asked a friend who reported that my question is &quot;100% right. We are stuck in limbo.&quot;<br><br>I was able to manually install Company Portal, but the instructions (here: <a href="https://t.co/jaxJva8u2j">https://t.co/jaxJva8u2j</a>) are wrong. Only download the x64 framework.</p>&mdash; Greg Shields (@concentratdgreg) <a href="https://twitter.com/concentratdgreg/status/1246497885354487809?ref_src=twsrc%5Etfw">April 4, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

The officially supported options are either 
* use MSfB, a technology with an uncertain future, that needs Global Admin rights to set up
* manually download the package for offline use and deploy as a Line-of-business app
* ask the user to install Company Portal manually


There is another way, thanks to the work of Oliver Kieselbach.

## Using Powershell to auto-install Company Portal

Oliver Kieselbach has a [great blog post][oliverblog] detailing how to use the StoreInstallMethod of the MDM Bridge WMI provider to change Windows Language settings. 

I have adapted this slightly to install Company Portal instead. All the code that does the main work is by Oliver, so all credit for this method belongs to him. 

<script src="https://gist.github.com/adotcoop/0241d371684c3771000385dd93da77e4.js"></script>

As noted in the comments this can be adapted to automatically install any store app by changing the application ID. For example, the iTunes URL is 

	https://www.microsoft.com/en-gb/p/itunes/9pb2mz1zmb1s

so to adapt this code to auto install iTunes, you'd change the applicationID line to read

	$applicationID = "9pb2mz1zmb1s"

This method works, and we use it in production. But I am not sure this is supported, so check before you deploy. 

It seems crazy that we need workarounds to deploy such a key part of the modern desktop experience. 

Intune allows us to deploy Microsoft 365 Apps (Office 365) and Edge Chromium automatically from the endpoint portal with little trouble. Hopefully this method to deploy store apps will not be required in some future Intune update.

[microsoftblog]:https://techcommunity.microsoft.com/t5/configuration-manager-blog/company-portal-app-for-use-on-co-managed-devices-is-now/ba-p/1602610

[mfsbsetup]:https://docs.microsoft.com/en-us/microsoft-store/roles-and-permissions-microsoft-store-for-business

[zdnet]:https://www.zdnet.com/article/microsoft-is-planning-to-phase-out-the-windows-10-store-for-business/

[pluralsight]:https://www.pluralsight.com/courses/deploy-apps-microsoft-intune

[oliverblog]:https://oliverkieselbach.com/2020/04/22/how-to-completely-change-windows-10-language-with-intune/

[mygithub]:https://github.com/adotcoop/Intune/blob/master/Install-CompanyPortal.ps1