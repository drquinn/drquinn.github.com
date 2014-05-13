---
layout: post
title: PowerShell ADDS-Domain-Controller Feature Name Not Valid
---

This error appeared after running a PowerShell script on Windows Server 2012 R2:

*The role, role service, or feature name is not valid: "ADDS-Domain-Controller". The name was not found.*

This script had been working on Windows Server 2008 R2.  Running **Get-WindowsFeature** on Windows Server 2008 R2 shows these available features, including ADDS-Domain-Controller:

![Features in Server 2008 R2](/assets/ADDS-Domain-Controller/2008R2.png "Features in Server 2008 R2")

However run the same command on Server 2012 R2:

![Features in Server 2012 R2](/assets/ADDS-Domain-Controller/2012R2.png "Features in Server 2012 R2") 

Notice **ADDS-Domain-Controller** is no longer present.  Instead, install the feature **AD-Domain-Services**, which will take care of part of the task.  **dcpromo** is now also depricated for Windows Server 2012.  Replace that command with [Install-ADDSForest](http://technet.microsoft.com/en-us/library/hh974720.aspx) or [Install-ADDSDomainController](http://technet.microsoft.com/en-us/library/hh974723.aspx).  I'm installing a brand new domain so I'll use the forest cmdlet:

	Install-ADDSForest –DomainName $domain –DomainMode Win2012 –ForestMode Win2012 -Force -SafeModeAdministratorPassword (convertto-securestring $password -asplaintext -force) –DatabasePath $locationNTDS –SYSVOLPath $locationSYSVOL –LogPath $locationNTDSLogs -verbose


