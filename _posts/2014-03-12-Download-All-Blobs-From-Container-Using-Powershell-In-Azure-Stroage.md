---
layout: post
title: Download all Blobs from a Container using Powershell in Azure Storage
---

Many common functions in Azure with PowerShell are difficult to locate across the internet. I'll be posting more here as I work through them.  This should have been simple but was suprising hard to get working (like much of Azure).  Save this script and run it with your variables filled in to download all blobs from a container in Azure to a folder on your local hard drive.  Here, I'm downloading everything from the 'packageitems' contianer and placing it into a folder called pstest on my C drive.  Be sure to also replace your account name and account key below in the connection string.

	$container_name = 'packageitems'
	$destination_path = 'C:\pstest'
	$connection_string = 'DefaultEndpointsProtocol=https;AccountName=[REPLACEWITHACCOUNTNAME];AccountKey=[REPLACEWITHACCOUNTKEY]'

	$storage_account = New-AzureStorageContext -ConnectionString $connection_string

	$blobs = Get-AzureStorageBlob -Container $container_name -Context $storage_account

	foreach ($blob in $blobs)
	    {
			New-Item -ItemType Directory -Force -Path $destination_path
	  
	        Get-AzureStorageBlobContent `
	        -Container $container_name -Blob $blob.Name -Destination $destination_path `
			-Context $storage_account
	      
	    }
