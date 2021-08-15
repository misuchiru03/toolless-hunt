# Windows Event Logs

## What are Windows Event Logs?

Windows Event Logs are detailed records of the system, security, and applications stored by the Windows Operating System. These event logs can be used to diagnose system problems, predict future issues, and even assist in threat hunting, detect compromises, and other malicious activity on the systems.

Events that a Windows Event Log can capture include various elements:

* Date - _The date the event occurred_
* Time - _The time the event occurred_
* User - _The username of the user logged onto the machine at the time of the event_
* Computer - _The name of the computer on which the event occurred_
* Event ID - _The ID number indicating the specific type of event_
* Source - _The program or component that caused the event to be logged_
* Type - _The type of event: Information, Warning, Error, Security success audit, Security failure audit_

## Event ID

An Event ID is a specific ID number that Microsoft has assigned to a specific event action. These numbers are not unique in that they don't repeat, but rather are unique in that they correlate to particular events; each of these numbers are universal across all Windows environments.

These Event IDs have changed in the history of Windows versions. Windows versions from XP/2003 and before had a different event ID number for each event. These corresponding event IDs can be found online. For the purpose of continuity and ease of reference, I will be using the Win7+ event IDs.

Some common Event IDs you will see on a typical Windows 7/2008+ system are:

* 4608 - Windows is starting up
* 4609 - Windows is shutting down
* 4624 - Successful local system login (non-remote)
* 4625 - Failed local system login (non-remote)
* 4634 - Account was logged-off
* 4647 - User initiated logoff
* 4688 - A new process has been created
* 4689 - A process has exited

By no means is this an exhaustive list of a typical system's Event IDs. You will eventually run into many very confusing ID numbers and require assistance one way or another to determine what the system decided to record. With that said, there are a list of particularly interesting Event IDs that have been seen in the past when conducting investigation of malicious or potential malicious activity that are worth noting:

* **4625 - Failed local system login (non-remote)**
   
   This Event ID can be completely legitimate, so don't be fooled into thinking that someone is trying to access an account or spraying passwords just because you see this. Collect as much data as you can across the network, including peak and slow hours; especially after-hours records. If you are noticing that after-hours logs show failed logins for user accounts, it may be something you need to look into. Also consider if these accounts that are failing to login are disabled as this can indicate malicious behavior, be it an insider threat or an adversary. You want to focus on ruling-out "legitimate behavior" as much as you can.

* **4771 - Failed Kerberos Pre Authentication**
   
   This relates to the previous 4625 event ID, except 4771 is specifically for the Kerberos authentication failure. When combined with error code 0x18, this is, simply-put, an incorrect password. Keep an eye open for a bunch of these on a single host. This can highlight brute force attempts.

* **4765/4766 - SID History added to an account/failed to add to an account**
   
   SID History (SID=Security ID) being changed on an account should really only happen when users migrate between domains. If you see these IDs, but the system owner does not have any migrations happening, look further into these accounts being recorded. Adversaries can use these SID-HISTORY alterations to impersonate users and/or escalate privileges.

* **4794 - Attempt made to set DSRM password**
   
   DSRM is the Directory Services Restore Mode. This event ID indicates a user or admin attempted to change the DSRM administrator password. This is crutial to keep an eye out for as whomever has this password can edit the Active Directory database. You want to tune-out legitimate uses of this ID by deconflicting with the system owner and investigate any other occurrences.

* **4793/4713/4719 - Policy Changes**
   
   These IDs indicate that Domain, Kerberos, and System Audit Policies were changed. Naturally, only approved users/admins should be changing these policies, and furthermore they should be within approved times and/or change windows, so deconflict with the system owner and investigate any other occurences.

* **4735/4737/4755 - Security Enabled Groups Changed**
   
   These IDs indicate that the groups you may consider "critical" have been altered: Domain Admins, SQL Admins, finance users, security personnel, or anything else the system owner deems important. If you find these IDs on the system, ask why these IDs would be there and correlate these IDs with documentation. If there is no specific note anywhere regarding _why_ these have changed, then that can be a big red flag, even if performed by legitimate administrators.

* **4728/4732/4756 - Users being added to Security Enabled Groups**
   
   Using what you know about the Security Enabled Groups, you can guess with confidence what these IDs can indicate. Users being added to these groups should have corresponding documentation as to _why_ they are in these groups. If not, this can be that big red flag again.

* **1102 - Audit Log Cleared**

   Normally, when an attacker or insider threat is performing their actions, as long as they are trying not to get caught, they will clear their tracks. Windows logs when logs have been cleared so you can at the very least track who cleared them and begin further investigations. Though I cannot necessarily think of any legitimate reason why a system administrator would clear logs (_maybe_ for starting a new baseline??) it does not mean that finding this is definitively malicious. Always deconflict any findings.

* **4648 - Logon Attempted using Explicit Credentials**

   This ID will be logged when a user connects to a server or runs a program while providing different credentials. It will also be logged when a scheduled task is setup using different credentials. Deconfliction is key here; is the user _allowed_ to use a specific set of credentials? This may indicate that the user is hiding in plain sight as another known user, or attempting to escalate privileges or gain access to sensitive data.

* **4697 - Service Installed**

   As indicated, this ID indicates that a service has been installed on the system. Ideally, the system owner should be able to provide a list of approved and installed services and applications, as well as their approved open ports and protocols (sometimes called a Ports, Protocols, and Services list or PPS). Using this whitelist, you can bounce the discovered services using this ID query off it, highlighting any and all services not on the whitelist, and deconflict further with the system owner.

* **4688 - Process Created**

   This one is one of the hardest ones to run down as any process that runs will log this ID. However, we can use the Mitre ATT&CK Matrix and other documents from other professionals around the globe to look for specific actions. When an attacker initially gains access to a system, some of the more common commands run in a short amount of time are:
  * `tasklist`
  * `ver`
  * `ipconfig`
  * `systeminfo`
  * `net time`
  * `netstat`
  * `whoami`
  * `net start`
  * `qprocess`
  * `query`
  * `hostname`

   After running these commands to get the lay of the land, where they are, which machine they have gained access to, which user, and local time, they perform further recon on the environment:
  
  * `pwd`
  * `dir`
  * `ping`
  * `type`
  * `net view`
  * `net use`
  * `net user`
  * `net localgroup`
  * `net group`
  * `net config`
  * `net share`
  
   Upon completing more detailed and widespread recon, the attacker may attempt to gain a foothold into the network further and want to gain persistence as well:
  
  * `at`
  * `reg`
  * `wmic`
  * `wusa`
  * `netsh firewall`
  * `sc`
  * `rundll32`
  
   When looking through event logs, you may see these processes (all or some), and running them does not make you a malicious actor. These are typical of a system administrator to run if they prefer to use the command line. However, a user who, according to the system owner or available baselines, is running these commands and should not have reason to access the command line, or does not have permissions to be able to access the command line, and the commands have been run in a short amount of time, that may be a red flag as well. Normally, once a user has logged in, they know what user they are logged in as, as well as the machine they are logged into.

* **4672 - Special Privileges assigned to New Logon**
   
   This is a special ID that indicates an administrator, or a user with admin privileges, has logged on. This can be anything from a user/admin logging in to a service or scheduled task being run with the admin's credentials or explicit credentials. Once again, rule out the normal actions, highlight abnormal, deconflict, investigate.

* **4698/4702 - Scheduled Task Created/Updated**
   
   Scheduled tasks are a normal way to ensure backups are created for users, user accounts, filesystems, logs, etc. However it is also very common for attackers to create a scheduled task for persistence. This ID is logged each time a new task is created and even when an existing task is edited. These created/updated tasks should be fairly easy to find as the system owner or admins should have a list of scheduled tasks they created, and anything additional requires investigation. It is also worth noting that any suspicious-looking scheduled tasks, such as a registry key being edited, created, or deleted, or an executable running with network permissions to a remote location should be flagged.

* **4720 - A Local User Account was Created**
   
   This may not seem like a big deal, and in a normal environment, it may not be. However, when an attacker looks to bypass security measures, they may create a local account while using a machine on a domain, grant privileges to that account or escalate privileges using that account, and work on getting a local admin account somehow. This type of action may be indicated by your finding Event ID 4720 followed shortly by Event ID 4732.

* **4732 - A Member was added to a security enabled local group**
   
   This ID indicates that a user account has been added to a local system's privileged groups, such as the administrators group. This would, depending on the environment, be a huge red flag but should always be verified against the known adminstrators and privileged users list from the system owner and administrators. 4732 following a 4720 can be a pretty good indicator that an attacker is elevating privileges for themself.

