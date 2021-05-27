---
layout: post
title:  "Mitigate the impact of malware for free with Microsoft Defender attack surface reduction rules"
date:   2021-05-26 20:31:42 +0100
--- 

Attack surface reduction rules are normally talked about in relation to Defender for Endpoint - a premium offering requiring E5/A5 or an add-on subscription to E3/A3. However, the core asr rules functionality is built into the Defender engine on Windows 10, and you can still use it on the following without any additional licensing

- Windows 10 Pro, version 1709 or later
- Windows 10 Enterprise, version 1709 or later
- Windows Server, version 1803 (Semi-Annual Channel) or later
- Windows Server 2019

You'll have to do the reporting and monitoring yourself, but we'll discuss that shortly.

Asr rules can improve the security posture of your environment and can be a quick win. They provide an extra layer of defence on top of the normal signature based Windows Defender solution.  

In this article I'll be covering one approach to enabling attack surface reduction for free and will provide scripts to enable asr rules and to gather logs from your devices. There are a couple of sections on how attack surface reduction rules differ from signature based antivirus - if you already know the difference you can skip them. 

One thing to note - you need to be using the built in Windows Defender as your antivirus solution for these rules to work.

* TOC
{:toc}

# Licensing - is it really free?

Microsoft have a statement about licensing on the FAQ page at [docs.microsoft.com](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/attack-surface-reduction-faq?view=o365-worldwide)
> The full set of ASR rules and features is only supported if you have an enterprise license for Windows 10. A limited number of rules may work without an enterprise license.


This statement doesn't make a lot of sense to me - they list Windows 10 Pro as a supported OS. How can the Defender engine determine if you've bought an enterprise license? And no mention about server requirements despite Server 2019 being listed as a supported OS. 

A more sensible statement is available on the [how to enable](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/enable-attack-surface-reduction) page

> Although attack surface reduction rules don't require a Windows E5 license, if you have Windows E5, you get advanced management capabilities. 
> 
> These advanced capabilities aren't available with a Windows Professional or Windows E3 license; however, if you do have those licenses, you can use Event Viewer and Microsoft Defender Antivirus logs to review your attack surface reduction rule events.


Having said that, all of my testing has been with Windows 10 Enterprise and I do have an enterprise license. Your mileage may vary with other combinations. 

# The trouble with signatures

Traditional antivirus software uses signatures to detect threats. A signature is created by your antivirus vendor for each threat they know about. When a file or application is accessed the antivirus software checks the signature of the file against its list of known bad signatures. If the file is on the list, the antivirus will perform an action (typically preventing it from running or placing it in a quarantine folder). If the file is not on the list, it can do whatever it wants.

And this is where signature based antivirus breaks down. To detect a virus you need a signature. So you need to keep your signatures up to date, and hope that your antivirus vendor can get you new signatures quickly.

It's hard to believe now, but in the mid-2000s many vendors supplied signatures on a monthly CD. It was not unusual to ask staff to come to the IT department to pick up a copy of the signature disk every month.

![alt text](/assets/asr_sophos_updates.png "Some old antivirus CDs that should have been thrown away a long time ago")

It goes without saying but things have changed quite a lot since then. I'm writing this post at 10:30 AM and [according to the release notes](https://www.microsoft.com/en-us/wdsi/definitions/antimalware-definition-release-notes) there have already been 5 signature updates for Defender  **this morning**.

![alt text](/assets/asr_defendersigs.png "Defender signature updates")

Assuming your devices can keep the signatures up to date, is that enough?

For a signature to be created, your antivirus vendor needs a sample of the malware. What if your organisation is the first company to see a specific bit of malware? Unless you have other protections in place, there will be no signature, and the malware will be able to execute on your machines.

Does this mean signature based scanning is useless? Far from it - it's a very mature technology and the signature databases protect your devices from many threats. But there are other ways to evaluate threats on your devices.

# Behaviour based rules

Think of signature based detection like a no-fly list. Someone turning up at an airport on a no-fly list will be stopped at security. But someone has to maintain that list and distribute it to the security guards at each airport.

Would using a no-fly list - signatures in antivirus terminology - provide enough security? 

Assuming you're not on the list, you could still try to forcibly open a security door. This is behaviour we would probably want to stop. This is where attack surface reduction rules come in - a way of looking at behaviour instead of just checking files against a list.  

For example, there is an asr rule

- Block Adobe Reader from creating child processes

Is there any reason for Adobe Reader to do anything other than open PDF files? Maybe this sort of behaviour is suspicious in your environment. Turning on this rule will prevent this behaviour. There is another rule

- Block executable content from email client and webmail

Do you often send your users executables via email? If not, maybe preventing this from happening would something you'd like to do.

Neither of these examples could be covered fully by signatures. Asr rules allow you to stop certain behaviours that you think are undesirable on your devices.

# Getting started with attack surface reduction rules

So far, so good. Behaviour based rules help fill in a missing piece of your antimalware approach. Microsoft provide 15 rules as part of the Defender offering. These are

- Block abuse of exploited vulnerable signed drivers
- Block Adobe Reader from creating child processes
- Block all Office applications from creating child processes
- Block credential stealing from the Windows local security authority subsystem (lsass.exe)
- Block executable content from email client and webmail
- Block executable files from running unless they meet a prevalence, age, or trusted list criterion
- Block execution of potentially obfuscated scripts
- Block Office applications from creating executable content
- Block Office applications from injecting code into other processes
- Block Office communication application from creating child processes
- Block process creations originating from PSExec and WMI commands
- Block untrusted and unsigned processes that run from USB
- Block Win32 API calls from Office macros
- Block JavaScript or VBScript from launching downloaded executable content
- Block persistence through WMI event subscription
- Use advanced protection against ransomware

All of these rules help to reduce the attack surface of your devices. However, you may have software that depends on functionality that would be blocked by these rules. 

The rule _Block process creations originating from PSExec and WMI commands_ looks very desirable - many attacks use psexec for lateral movement or to elevate to privilege levels that standard tools can't manage. But turning on this rule will break some functionality that is used by Configuration Manager. Intune appears to be unaffected, but enabling this rule with SCCM is a no-no.

Another rule that would be good to use if possible is _Block credential stealing from the Windows local security authority subsystem (lsass.exe)_ as this helps defend against tools like mimikatz. Turning this on can lead to a lot of events being generated as Microsoft state on the [docs page](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/attack-surface-reduction)

> In some apps, the code enumerates all running processes and attempts to open them with exhaustive permissions. This rule denies the app's process open action and logs the details to the security event log. This rule can generate a lot of noise. If you have an app that simply enumerates LSASS, but has no real impact in functionality, there is NO need to add it to the exclusion list. By itself, this event log entry doesn't necessarily indicate a malicious threat.

As these examples show, rules that seem to be obvious wins can have unintended consequences. Luckily asr supports enabling these rules in several states to help us mitigate any possible side effects

- Audit
- Block
- Warn

Block does as it suggests - if a rule fires, it prevents the process from carrying out that action. Warn allows the user to override the block if they require. Audit takes no action but logs the event.

Be aware that these rules have the potential to leave key software non-functional. Before enabling rules, you should run them in audit mode to see what might be blocked if you turned them on. 
 

# Enabling asr rules in audit mode

There are a number of different ways to turn these rules on. These include

- Configuration Manager
- Intune
- Powershell cmdlets
- Group Policy

These are well documented on the Microsoft [docs.microsoft.com](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/enable-attack-surface-reduction) site.

Although the documentation is good, enabling the rules is not very user friendly. For example, here's a local group policy being configured to audit Adobe Reader child process creation

![alt text](/assets/asr_grouppolicy.png "Local Group Policy editor")

Yes, you do need to do the lookup between GUID and rule name yourself. And to configure more than one rule in group policy is time consuming. Intune exposes these settings using the OMA-URI interface - a particularly unfriendly way to configure settings. Configuration Manager is a lot better but doesn't expose all settings that are available. 

For my testing I wanted something easy that would enable all the rules in audit mode. The following reg file sets the local group policy to audit all 15 rules. Import this and do a gpupdate for the rules to take effect.

<script src="https://gist.github.com/adotcoop/4648621a199e116f0ab13fd21758ebda.js"></script>

One thing to note here - if you configure the rules in any other way these local policies will be overwritten. 

# How to retrieve audited log entries

After the rules have been enabled, and you've given them time to generate some events (Microsoft suggest at least 30 days), you'll want to analyse the information. This is where Defender for Endpoint would come in. If you can, use that. Because we are doing this for free, we need to come up with something ourselves. 

When the rules fire in audit mode an event is generated in the Defender Operational log with event ID 1122. If we had configured the rules to block behaviour, the ID would be 1121. 

![alt text](/assets/asr_auditevent.png "asr event in Event Viewer")

Whilst you _could_ go through these logs manually, there are better ways. If you have a SIEM or use Windows Event Forwarding, you could collect these events centrally. Or you could use the following script to dump the events to a csv file for further analysis.

<script src="https://gist.github.com/adotcoop/770e1637090a95c40775907beda319b0.js"></script>

If we run the script, it will output some information to the console

![alt text](/assets/asr_scriptoutput.png "Powershell window showing sample output from the script")

The same information is exported to csv if you specify a filename. This is obviously a lot more useful if you have lots of events, or want to correlate information across lots of devices. Here's a sample csv with the psexec rule in audit mode and then in block mode

<script src="https://gist.github.com/adotcoop/f4166d7901f23ef7286e365be5a7c644.js"></script>

This csv can tell you when rules fire. What it can't tell you is whether turning on a specific rule is going to have unintended consequences.

# Analysing audit data

Before making any decisions about what rules to enable you need to gather data from a good enough sample set. Microsoft recommend enabling every possible rule, and running them for 30 days before making a decision.

If all of your devices run the same build and the same set of software, you find some consistency. However, even in this limited scenario you might run into problems - the same software used by different people may generate different results. 

Before enabling anything you'll want to ensure that you cover enough devices and user use cases to be happy that the rules will not cause too many issues. At a minimum I would run the rules in audit mode on

- each different build or image you have
- in each different department or unit you have

For example, you might want to audit

- the Windows 10 Office Worker Image
- the Windows 10 Developer Image
- the Marketing department
- the Finance department

The scope of your audit should determine the scope of your deployment.

# Enabling asr rules

Assuming you are happy with the audit information you can start setting rules to block mode. Again, the Microsoft documentation is good for this, and you can use any of the standard Microsoft management platforms to configure the rules. 

One thing to note is that in the above examples, we set some local group policies with a GUID set to a value of 2. The range of values that can be set are

- 0 Disable
- 1 Block
- 2 Audit
- 6 Warn

To set something to block just change the value (however you set it) from 2 to 1.

# Exclusions

In the process of auditing what would be blocked if a rule was enabled, you might spot specific applications that do indeed need to behave in a specific way. Instead of leaving the rule disabled, you can set up an exclusion. 

Microsoft explain the exclusion process pretty well [here](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/customize-attack-surface-reduction) so I won't repeat it. 

As expected you can exclude files and folders. One thing to note though - the exclusions you set up apply to **all** rules.

This will stop any of your asr rules from firing for the excluded files or folders, not just the specific rule you want it to ignore. Microsoft are very specific about the threat exclusions can pose (taken from the above link)

> Files that would have been blocked by a rule will be allowed to run, and there will be no report or event recorded.

If you are going to use exclusions try to limit their use to only the devices that absolutely need them.

# Conclusions

Attack surface reduction rules are a very cool bit of functionality built into Defender and, by extension, built into most of the supported OSs from Microsoft. 

If you configure the rules as described in this article you're getting a pretty decent HIPS solution for free. 

One thing is obviously missing here - the reporting and monitoring piece. Just enabling the rules will give you some level of protection, but how can you tell when rules fire?

You could alter the event log dumping script to achieve some of these goals, but if you're going to that level maybe it's time to discuss a SIEM or E5 with your management.

The other conversation that can be generated by this technology is around the quality of the software that is running on your devices. If you have to make a security exception because software you pay for behaves badly, maybe it can help make the case to embed more rigorous security requirements into your procurement processes.
 
