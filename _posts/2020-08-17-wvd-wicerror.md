---
layout: post
title:  "Windows 10 multi-user app install failure - Windows Installer Coordinator "
date:   2020-08-17 19:35:02 +0100
--- 

Deploying apps to Windows 10 multi-user is _almost_ the same as deploying apps to normal Windows 10 but there are some quirks. One of these quirks can cause some MSI installers to fail.

I first saw this installing IBM SPSS 26 for use in WVD

![alt text](/assets/wic-error.png "Windows Installer Coordinator error")

> Windows Installer Coordinator
>   
> Please wait while the application is preparing for the first use

What is the Windows Installer Coordinator? It is a component, enabled by default on multi-user Windows, that stops multiple MSI installs from running concurrently. [docs.microsoft.com][msdnarticle] has a good write up about this feature on RDS machines

> A multiple package installation using the MsiEmbeddedChainer table fails if the Remote Desktop Services role is enabled.

The work around on [support.microsoft.com][microsoftsupport] is to
>  disable the Remote Desktop Session Host Windows Installer for the duration of the installation. 

The article above describes using Group Policy to fix this issue, but a registry change will also work. 

Import this registry file before installing the affected app and the error will be gone, no reboot required

<script src="https://gist.github.com/adotcoop/ce721b2fac802ada93ea6a8dc297955a.js"></script>

A number of other products from major vendors are affected by this such as [IBM Spectrum Protect][ibmsupport], [Sage 50][sagesupport] and [MYOB][myobsupport].

[microsoftsupport]:https://support.microsoft.com/en-us/help/2655192/installation-can-fail-with-window-installer-coordinator-error
[ibmsupport]:https://www.ibm.com/support/pages/windows-installer-coordinator-install-phase-may-loop-or-hang
[msdnarticle]:https://docs.microsoft.com/en-gb/windows/win32/msi/msiembeddedchainer-table
[sagesupport]:https://support.na.sage.com/selfservice/viewContent.do?externalId=82727
[myobsupport]:https://community.myob.com/t5/AccountRight-Installing-and/2019-2-Upgrade-workstation-upgraded-but-server-stuck-Windows/td-p/580190