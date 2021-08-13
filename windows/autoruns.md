# Run and RunOnce Registry Keys

## What are Run and RunOnce Keys?

Run and RunOnce Keys are registry keys that run programs every time a user logs in to a system. The information stored in the key is known as a data value and is a command-line entry less than 260 characters--ANY application can be run, ANY command can be run. You can even write multiple commands in a single key data value to have multiple apps/commands run on login. The order of these commands in this data value does not correlate the order in which they run.

## Where do I find these Run and RunOnce Keys?

The first places you should look for these Run and RunOnce Keys are in the following registries:

* HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
* HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
* HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce
* HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce

There are many more places you can look, but these should be the first as they are the most widely-known and used. Some other known and used keys are:

_This key is not created by default on Windows Vista and newer. If found on Vista or newer, consider it interesting_
* HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnceEx


_The following keys can be used to set startup folder items for persistence_
* HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders
* HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
* HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
* HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders


_The following keys can control automatic startup of services during boot_
* HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce
* HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce
* HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunServices
* HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunServices


_The following keys refer to the policy settings which specify startup programs_
* HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run
* HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run


_The following key controls actions that occur on Windows 7 logons. Most actions are under the control of the OS, however this key adds custom actions_
* HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Userinit


_The following key can automatically launch programs on login for Windows 7 machines_
* HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell


_The following key causes Windows to check the file-system integrity after an abnormal shutdown. Adversaries can use it to launch their applications at boot_
* HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager

---

### Some of these keys exist and have data in them. Is this malicious?

Many times you will be assessing the security of a host and find values in these keys. While that may be concerning at first glance, it may definitely be worth noting while you continue looking through other keys and collecting data. Values existing in these keys does not automatically denote malicious activity. As you can see from each description, they have legitimate purposes as well. It is always best to verify the startup scripts and applications in these keys with the system owner and further research (in the case the owner cannot speak specifically to technical details) to ensure you are tracking something malicious or falling down a rabbit hole.


### How do I collect these keys and their values? Is there a tool that can do this for me?

You can collect the keys and their values with Powershell using the following command:
`Get-Item -Path Registry::<keyname> | Select-Object -ExpandProperty Property`

You can also use CMD with the following command:
`reg query <keyname>`

With these two commands, you can create a script to collect the data for yourself in the format you would like.

If you are in the market to get a tool, Microsoft's SysInternals Suite is an excellent place to start. Using this suite of tools, you have a command-line tool called `Autorunsc` and gui tool called `Autoruns`. This utility has the most comprehensive knowledge of startup keys and it shows you what is currently configured to run on system start or login. Furthermore, it can show you what runs when you start Windows applications such as Internet Explorer, Explorer, and media players. Additionally, Autoruns will report user shell extensions, toolbars, and browser helper objects, Winlogon notifications, scheduled tasks, auto-start services, and more, which also helps detect Potentially Unwanted Programs (PUPs).


### I found a key I believe is malicious, what should I do?

First thing first, you want to gather as much detail about it as possible. Does the system owner know what it is? What is the key executing when it is called? Is the key successfully executing? Cross-reference the key's supposed actions with network traffic and Windows event logs to validate your claims and ideas. Once you have a better understanding of the events, you can use a sandbox virtual machine to run the same malicious application to examine the outcome inside the sandbox. Verify your claims also against what your system owner knows. Make sure you trace the key back to the point of entry. If you were to remove the key, will it come back due to a different persistence mechanism? Tracing back to the point of entry will give you as the defender/responder the information necessary to prevent a re-infection. This may require checking filesystem permissions in a specific user account, folder, service, or even firewall rules and service configuration settings.

