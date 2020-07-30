---
layout: post
title:  "Check a successful Azure AD Join and Intune enrolment with one line of PowerShell"
date:   2020-07-27 19:58:07 +0100
---

This line will output the contents of the local machine certificate store. Depending on the environment there might be a number of certificates displayed, but you only need to care about three to make sure the join and enrolment are working.  

For a successful Azure AD join the following certificates should be present

* MS-Organization-P2P-Access 
* MS-Organization-Access

For the Intune enrolment the following will also be present

* Microsoft Intune MDM Device CA

<script src="https://gist.github.com/adotcoop/e256fe93ad7a1da8b57c262308669ba7.js"></script>

