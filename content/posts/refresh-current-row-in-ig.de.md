---
title: "Interactive Grid – Nur die aktuelle Zeile aktualisieren"
date: 2018-01-27T22:02:37+02:00
lastmod: 2018-01-27T22:02:37+02:00
draft: false
author: ""
authorLink: ""
description: ""
license: ""

tags: []
categories: []
hiddenFromHomePage: false

featuredImage: "/images/refresh-current-row-in-ig/Snap1.gif"
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
In meinem aktuellen Projekt benutze ich auf einer Seite ein Interactive Grid. Wenn man auf eine Zeile klickt, sprich diese selektiert, wird unterhalb des Grids ein Report angezeigt, der diesen Datensatz darstellt. Das Bearbeiten des Datensatzes wird dabei in einem modalen Dialog realisiert. Wenn dieser geschlossen wird, soll sich natürlich der Report inklusive der gerade selektierten Zeile im Grid aktualisieren.
<!--more-->
Leider bleibt die Selektion nach einem Refresh des Grids nicht erhalten. Wenn man via JavaScript das Grid aktualisiert, sich vorher den ausgewählten Datensatz merkt, und anschließend diesen wieder markieren möchte, kommt man zu dem Problem, dass das Aktualisieren des Grids asynchron von statten geht. Und der auszuwählende Datensatz noch gar nicht da ist.

Da ich aber nur den einen Datensatz bearbeiten möchte, reicht es eigentlich aus, nur diese Zeile zu aktualisieren. Dazu bietet das IG eine entsprechende Methode per API an.

```javascript
var myGrid = apex.region( "TEST_CASES_GRID" )
 .widget()
 .interactiveGrid("getViews")
 .grid;
myGrid.model.fetchRecords( myGrid.getSelectedRecords());
````

Das ganze klappt nur, wenn das IG über eine RowID verfügt.

!["Aktualisieren der ausgewählten Zeile"](/images/refresh-current-row-in-ig/apex_ig_refresh_single_record.gif "Aktualisieren der ausgewählten Zeile")
