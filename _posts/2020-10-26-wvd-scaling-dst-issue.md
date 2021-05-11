---
layout: post
title:  "WVD Scaling Tool Daylight Savings Time Issue"
date:   2020-10-26 10:56:54 +0100
--- 

Microsoft provide a scaling solution for Windows Virtual Desktop at [docs.microsoft.com](https://docs.microsoft.com/en-us/azure/virtual-desktop/set-up-scaling-script). It is not as fully featured as other scaling solutions but it does a decent job of powering off and powering on the virtual machines during peak and off peak hours.

One thing it doesn't do is adjust the time offset automatically, and this causes a problem when the clocks change. 

In the UK we have just moved from British Summer Time (UTC+1) to Greenwich Mean Time (UTC+0). This means that the peak hours defined in the WVD Scaling tool are off by one hour.

<!--more-->

## Checking the offset 
- Open up [portal.azure.com](https://portal.azure.com) and go to Automation Accounts

![alt text](/assets/wvdscaling_wvdautomation.png "Automation Accounts")

- Under Process Automation click on Jobs

![alt text](/assets/wvdscaling_wvdautomationjobs.png "Automation Jobs")

- Click on the latest job

![alt text](/assets/wvdscaling_wvdautomationjoblist.png "Automation Job List")

- In the job details page click All Logs

![alt text](/assets/wvdscaling_wvdautomationlog.png "Automation Log")

-  In my log file the time offset is incorrect - the execution time is 09:27:23 but the script has added my UTC+1 offset and thinks the time is actually 10:27:23 

![alt text](/assets/wvdscaling_timeoffsetwrong.png "Automation Log")

## Fixing the offset

I fixed this by editing the Logic app directly. 

**If you make a mistake in this section you could prevent your logic app from running, or worse.**

**Test before making changes to a production environment**

- In the Azure portal, navigate to Logic Apps. Select your scheduler app (the name should be ```hostpool_AutoScale_Scheduler```)
- Under Development Tools click Logic App Designer

![alt text](/assets/wvdscaling_logicdesigner.png "Logic Designer")

- Click on HTTP

![alt text](/assets/wvdscaling_logicdesignerHTTP.png "Logic Designer")

- The UTC offset is listed as "TimeDifference". In my environment it was set to UTC +1:00, British Summer Time (BST)
![alt text](/assets/wvdscaling_logicappbefore.png "Logic App with UTC+1")

- For the UK we need to change this to +0:00 UTC (GMT)
![alt text](/assets/wvdscaling_logicappafter.png "Logix App with UTC+0")

- Click Save

## Checking that it has worked
You could wait for your next run of the app, but I just clicked the Run button at the top of the Logic app designer. Once the app has run, we can check the logs again. In my environment the time is now set with the correct offset again
![alt text](/assets/wvdscaling_timeoffsetfixed.png "Log File with correct offset")

## Summary
The scaling tool is great for reducing the cost of running WVD, but it is a bit rough around the edges.

Hopefully an automatic time zone correction will be implemented in a future version before the next summer time change.


