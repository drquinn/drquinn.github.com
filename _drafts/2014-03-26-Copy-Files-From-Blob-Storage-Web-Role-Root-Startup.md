---
layout: post
title: Copy Files From Azure Blob Storage Into Web Role Site Root Using Startup Tasks
---

There are many reasons you might want to move files into your Web Role on start.  

-	Faster deployment time (files can move faster in Azure than they can be published as one large package)
-	Size restrictions - the cspkg file sent to Azure can only be 600MB max.  With multiple roles or large site resources this limit can be easily exceeded.
-	Bandwidth costs - repeatedly deploying sites at half a gig can add to data transfer cost.  Uploading files once and pulling them repeatedly in a site that uses many instances helps lower cost.

Whatever the reason, this can be accomplished using either the OnStart method of your WebRole.cs file or from within a startup task.  You will just need to get the changing siteroot drive using environmental variables.  You will want to add files to your siteroot, which represents your actual website as opposed to your approot, which is only a launching point for WebRole.cs.

I am doing this using a Startup Task because I want to copy to happen immediately and in the background.  The files will be present on the server before sites are provisioned.  I can then use those files in provisioning the sites.

###Copy using PowerShell

Assuming you already have a Cloud Service Project created in Visual Studio, open your ServiceDefinition.csdef, located in the Cloud Service Project.  You won't be able to call a PowerShell script directly from ServiceDefinition.csdef, but you can call one from a startup batch file.

Under the <webRole> node, add a startup task.

	  <WebRole name="WebRole1" vmsize="Large">
	    <Startup priority="-2">
	      <Task commandLine="initialize.bat" executionContext="elevated" taskType="simple" />
	    </Startup>
	  </WebRole>

Create a file named initialize.bat and place it in your Web Role's bin folder.

The only job for initialize.bat is to call a PowerShell script, so add the following code.

	PowerShell -Version 2.0 -ExecutionPolicy Unrestricted .\startup.ps1 >> "%TEMP%\StartupLog.txt" 2>&1

This batch file will first set the ExecutionPolicy to allow for our unsigned script to run, then call it.  It will then log any ouput to a text file in the Web Role temp drive.

