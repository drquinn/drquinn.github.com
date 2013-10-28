---
layout: post
title: Troubleshooting HTML5 Video in SharePoint 2010
---

As expected, Internet Explorer presents problems rendering the HTML5 `<Video>` tag in SharePoint 2010. 

	<video width="320" height="240" controls>
		<source src="mov_bbb.mp4" type="video/mp4">
		Your browser does not support the video tag.
	</video>

I threw this sample into a Content Editor Web Part and had the video rendering perfectly Chrome and Firefox right away. But I took a look in IE, and:

![SharePoint Invalid Video](/assets/SharePointVideo/invalid-video-1.PNG "SharePoint Invalid Video")


Invalid Source. After digging around, many articles suggest that HTML5 tags, like Video, need the HTML5 DOCTYPE to render.


###Update the Master Page to Include the HTML5 DOCTYPE

Here’s a few blog articles about it:
https://www.nothingbutsharepoint.com/sites/eusp/Pages/HTML5-and-SharePoint-2010-Part-2-An-Overview-Continued.aspx
http://blog.drisgill.com/2010/09/html5-and-sharepoint-2010-and-ie9-beta.html

The overall point, is that you must replace 
	<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

with

	<!DOCTYPE html>

Which must be done in the master page. First thing I tried was taking a copy of v4.master, uploading it to the master page gallery, and pointing the site to use it.  Well the page failed with:

"The base type [some class] is not allowed for this page. The type is not registered as safe." blah blah error.

Will adding this class to your web.config as a safe type help? Maybe, but it did not for me. The problem was that the page was unghosted. If you upload a master page to the gallery like I did, its ghosted status is set to none. I tried changing this with PowerShell, but apparently you cannot. The solution?  Deploy the Master Page as a Solution.  So I did that, which removed the error! But the video still doesn't play.  


###Add MIME to IIS

If your site is having a problem recognizing the .mp4 file format, you might need to update IIS.  Even though the video plays fine in Chrome, it seems IE depends on this setting play in IIS to play .mp4 files.

![MIME Types](/assets/SharePointVideo/MIMETypes.PNG "MIME Types")


Add the .mp4 exension by clicking add.  Enter .mp4 as the File name extension and video/mp4 as the MIME type.

![Add MIME Type](/assets/SharePointVideo/MIMETypes2.PNG "Add MIME Type")


###Enable the Desktop Experience

This was the final step I needed to make things work. Like many SharePoint developers, I am working on a Virtual Machine running Windows Server 2008 R2 which does not have Windows Media Player installed by default. There are a few ways to install Windows Media Player (such as through the Microsoft website), but I decided to do it by enabling the Desktop Experience Feature. Enabling that feature installs Windows Media Player which installs the codecs needed to play .mp4 files.

![Desktop Experience](/assets/SharePointVideo/desktopExperience.PNG "Desktop Experience")


Reboot after installing and hopefully at this point, your video is playing in IE, Firefox, and Chrome.
