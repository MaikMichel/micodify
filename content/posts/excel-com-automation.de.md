---
title: "Excel SaveAs via COM/OLE - Automation schlägt unter W2008 fehl"
date: 2015-06-17T21:14:10+02:00
lastmod: 2015-06-17T21:14:10+02:00
draft: false
description: "Was isn das"

tags: ["Forms", "Oracle", "Microsoft", "Excel", COM"]
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

Es ist zum Verzweifeln. Ich habe für einen Kunden den Excelexport via Forms vom Client auf den Server verschoben. Auf meinem Windows7-Rechner auch kein Problem. Die Logik ist ja eigentlich auch ganz einfach. Exceldokument auf dem Server erzeugen, Transfer zum Client, auf dem Server löschen und anschließend auf dem Client anzeigen.

<!--more-->

Leider schlägt das auf dem Server (Windows2008 R2 64bit) fehl. Hier hat der COM-Befehl SaveAs für Excel keine Bedeutung. Und ohne gespeicherte Datei auf dem Server auch keine Datei auf dem Client.

Nach vielem Debugen und Googlen bin ich auf folgenden Link gestoßen: <http://per.lausten.dk/blog/2011/04/excel-automation-on-windows-server-2008-x64.html>

**Die Lösung?:** Einfach einen leeren Ordern mit dem Namen Desktop in das Verzeichnis: ```C:\Windows\SysWOW64\config\systemprofile``` packen.

Hätte ich auch selbst drauf kommen können... :stuck_out_tongue:
