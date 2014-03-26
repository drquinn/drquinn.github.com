---
layout: post
title: Windows Azure Web Role Startup Task Repeating in Loop
---

Startup Tasks in an Azure Web Role can get into a repeating loop for a variety of reasons.  However in my most recent case, the Startup Task itself was not the issue.  It was running to completion, then being called again and again and again and… well the role would never leave the "Running Application Startup Tasks..." phase.

###The reason: 
An error from code running in the OnStart method of WebRole.cs.  The error was hidden and would cause the whole startup process to restart.

###Solution: 
Be sure everything in OnStart is wrapped in a try/catch.  And be sure anything you catch is being logged.  In the sample below, I use Azure Trace Diagnostics to send logging information to Azure Tables.  Then use Azure Storage Explorer to open the table and view the log details.  This will help you identify whatever error is lost in OnStart and avoid the role from restarting everything.

	namespace WebRole1
	{
	    public class WebRole : RoleEntryPoint
	    {
	    	public override bool OnStart()
	        {
	            
	            // Set up tracing diagnostics.
	            Trace.Listeners.Add((TraceListener)new DiagnosticMonitorTraceListener());
	            Trace.WriteLine("Entering OnStart...");

	            try
	            {
	            	// Any code to run on start.
	            }

	            catch (Exception ex)
	            {
	                Trace.TraceError(ex.ToString());
	            }
	        }
	    }
	}

###Enable Diagnostics
You also need to be sure to have diagnostics enbaled for your web role, and point it to a storage account in Azure.  Set this by right clicking on your Web Role and clicking properties.  Check the Enable Diagnostics box and click the ellipsis to load in your Azure subscription where you can select your storage account.  For development, it is helpful to log all information.

![Web Role Diagnostics Configuration](/assets/AzureLoop/diagnosticsAzure.PNG "Web Role Diagnostics Configuration")
