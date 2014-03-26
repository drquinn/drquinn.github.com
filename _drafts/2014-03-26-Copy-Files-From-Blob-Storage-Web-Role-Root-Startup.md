---
layout: post
title: Copy files from Azure Blob Storage into your Web Role Site Root In Startup Task
---

There are many reasons you might want to move files into your Web Role on start.  
- Faster deployment time (files can move faster in Azure than they can be published as one large package)
- Size restrictions - the cspkg file sent to Azure can only be 600MB max.  With multiple roles or large site resources this limit can be easily exceeded.
- Bandwidth costs - repeatly deploying sites at half a gig can add to data transfer cost.  Uploading files once and pulling them repeatedly in a site that uses many instances helps lower cost.

Whatever the reason, this can be accomplished using either the OnStart method of your WebRole.cs file or from within a startup task.  Simply get the siteroot using environmental variables.  You will want to add files to your siteroot, which represents your actual website as opposed to your approot, which is only a launching point for WebRole.cs.

I am doing this using a Startup Task because I want to copy to happen immediately and in the background.  
