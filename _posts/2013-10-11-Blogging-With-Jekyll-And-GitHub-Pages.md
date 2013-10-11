---
layout: post
title: Setting Up This Blog with Jekyll and GitHub
---

{{ page.title }}
================

I have been stuck for months deciding on a blogging platform to choose.  I really should have just picked something and started but I couldn't.  It needed to feel right.  This does.  The number one goal was having control of the files I publish. Word Press, Tumbler, Medium, SharePoint, or anything platform based was out.  I just want to push static pages (html/css/js) without anything sever side.  But I wanted a bit more ease than having to type each line of HTML myself.  No need to insert `<h1>` and `<p>` tags if something can take care of simple styles for me.  Which is exactly what Jekyll can do.  Jekyll is a Ruby script that takes in text files and spits out all of the complete html needed to upload directly to your favorite host.  However I was not looking to pay for hosting.  Luckily GitHub Pages are free and are coincidently built on Jekyll as well.  So how do you use Jekyll?  And how do you post your Jekyll generated site for free to GitHub?

### How to Use Jekyll ###

Use this guide!

[http://www.andrewmunsell.com/tutorials/jekyll-by-example/index.html](http://www.andrewmunsell.com/tutorials/jekyll-by-example/index.html)

After looking through a few Jekyll/GitHub Pages guides, I finally stumbled on this magnificant tutorial.  It explains Jekyll with a goal of hosting your site on Amazon, but everything up until the Amazon piece can be followed exactly.  The high level overview:

*	Set up Jekyll locally.  Do this so that you won't need to upload every tiny change to GitHub.  Edit and build locally, then push everything at once to GitHub. This isn't actually needed.  You can upload your source directly to GitHub and it will run those files through Jekyll and spit out your site.  But there is a slight delay from upload to render and I want to see my changes immediately.
*	Build out the folder structure that is expected and read by Jekyll.
*	Add some content pages.
*	Style your blog with Bootstrap (or just do it yourself if you have better CSS and design skills than I do).

### How to Post a Jekyll Generated Site to GitHub for Free ###

Very simple.  Create a new repository in GitHub called *yourusername*.github.com.

Add the files and folders you created by following Andrew Munsell's wonderful tutorial.  Wait a couple of minutes.  Go to that URL.  The source files are automatically ran through Jekyll by GitHub and your site will magically appear.

### Point You Custom Domain ### 

Just a few steps - follow this guide directly from GitHub:

[https://help.github.com/articles/setting-up-a-custom-domain-with-pages](https://help.github.com/articles/setting-up-a-custom-domain-with-pages)

You will add a CNAME file to the root of your GitHub repository. Then go to your domain provider and add an A Record to your DNS settings that points your custom domain name to the GitHub server IP.