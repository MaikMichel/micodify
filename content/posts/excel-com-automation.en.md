---
title: "Excel SaveAs via COM/OLE - Automation exception when using W2008"
date: 2015-06-17T21:14:10+02:00
lastmod: 2015-06-17T21:14:10+02:00
draft: false
description: "Was isn das"

tags: ["Forms", "Oracle", "Microsoft", "Excel", COM", "Bug"]
categories: ["Oracle Forms"]
hiddenFromHomePage: false

featuredImage: "/images/excel-com-automation/excel_splash.jpg"
featuredImagePreview: ""

toc: false
autoCollapseToc: true
math: false
lightgallery: true
linkToMarkdown: true
share:
  enable: true
comment: true
---

It’s desperate. I moved the trace export for a customer from the client to the server via forms. No problem on my Windows7 computer. The logic is actually quite simple. Create Excel document on the server, delete transfer to the client, delete it on the server and then display it on the client.

<!--more-->

Unfortunately this fails on the server (Windows2008 R2 64bit). The COM command SaveAs for Excel has no meaning here. And without a saved file on the server there is no file on the client.

After much debugging and googling I found the following link: <http://per.lausten.dk/blog/2011/04/excel-automation-on-windows-server-2008-x64.html>

**The solution?:** Simply put an empty folder called Desktop in the directory: ```C:\Windows\SysWOW64\config\systemprofile```.

I could have come up with it myself…... :stuck_out_tongue:
