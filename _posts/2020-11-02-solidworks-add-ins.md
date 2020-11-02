---
layout: post
title:  "Solidworks - disabling add-ins via the registry"
date:   2020-11-02 09:12:54 +0100
--- 

We recently saw an error with Solidworks 2019 in our Windows Virtual Desktop environment.

On launch the app would pop up a dialog with the error

	Class not registered (Exception from HRESULT: 0x80040154 (REGDB_E_CLASSNOTREG))

![alt text](/assets/solidworks_error.png "Solidworks error")

Dismissing the dialog allowed the app to continue loading but it was a cause for concern. In a situation like this we have two priorities

1. get the app working
2. work out the root cause

In a normal situation we would do both, as addressing the root cause often fixes the issue. 

Searching the web we can see that an Office 365 update has caused other errors for Solidworks (such as reporting that "[The operating system is not presently configured to run this application](https://hawkridgesys.com/kb/users-using-solidworks-2020-sp4-finding-error-the-operating-system-is-not-presently-configured-to-run-this-application)"), so an update is probably our culprit. From a security position we don't want to roll back patches unless we absolutely have to, so we looked to find a workaround.

In the screenshot above we can see that Solidworks was loading an add-in at the time of the error - *SOLIDWORKS Social 2019*. Looking at the add-ins section we can see that it has added 24 seconds to the launch time, so this looks like the component that is generating the error. 

![alt text](/assets/solidworks_error_addins_before.png "Solidworks add-ins with SOLIDWORKS Social 2019")

After confirming that this add-in was not actually needed, we set about trying to disable it on a per-machine basis.

The Solidworks app details all add-ins in the registry under

	HKLM\SOFTWARE\Solidworks\SOLIDWORKS 2019\Addins

Each add-in is listed in its own key, but each key does not have a friendly name. All the add-ins are listed under GUIDs. To find out which one was the *SOLIDWORKS Social 2019* add-in we had to expand each key in turn until we found the correct one. 

![alt text](/assets/solidworks_error_regedit.png "Solidworks registry key")


Deleting the key effectively removes this add-in from being made available to Solidworks at launch, preventing the error from being displayed. The key can be deleted in a number of different ways such as Group Policy Preferences, REG DELETE or Powershell, but we chose to use a .reg file.

	Windows Registry Editor Version 5.00

	[-HKEY_LOCAL_MACHINE\SOFTWARE\SolidWorks\SOLIDWORKS 2019\Addins\{22b0c0d6-db0e-4a7b-9e07-1027ac206429}]

Once this was deployed to all machines, the add-in was prevented from loading and users could use the software error free. The add-ins section reflects that *SOLIDWORKS Social 2019* has been removed

![alt text](/assets/solidworks_error_addins_after.png "Solidworks add-ins after editing registry")






