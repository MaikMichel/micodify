---
title: "Arraylike Functions in PL/SQL"
date: 2019-08-01T21:11:15+02:00
lastmod: 2019-08-01T21:11:15+02:00
draft: false
author: ""
authorLink: ""
description: ""
license: ""

tags: []
categories: []
hiddenFromHomePage: false

featuredImage: "/images/array-like-functions/array-like-functions.jpg"
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
Vor geraumer Zeit, zur Zeiten als ich mich noch um Oracle Forms kümmern musste, habe ich mich geärgert, das man in Forms für das Ein- und Ausschalten immer mehrere Prozeduren aufrufen muss. So muss man für das Wiedereinblenden eines Textfeldes folgende Zeilen absetzen.

<!--more-->

```sql
set_item_property('MY_ITEM', visible, property_true);
set_item_property('MY_ITEM', enabled, property_true);
set_item_property('MY_ITEM', navigable, property_true);
```

Und das Ganze wiederholt sich dann natürlich pro Item. Cool wäre doch hier, wenn man alle Items auf einmal bedienen könnte. Und das ganze so wie in anderen Sprachen, zum Beispiel Java oder JavaScript. Oracle gibt uns hierfür spezielle Typen:

**odcivarchar2list** und **odcinumberlist**

Mit diesen kann man direkt eine variable Liste an Parametern erstellen und an eine Methode übergeben. So würde eine Prozedur, die einfach alle übergegeben Eigenschaften mit einem entsprechenden Wert setzt, so aussehen:

```sql
procedure set_item_properties(p_item        in varchar2,
                              p_properties  in sys.odcinumberlist,
                              p_property    in number) is
begin
  for i in (select m.column_value m_value
              from table(p_properties) m)
  loop
    set_item_property(p_item, i.m_value, p_property);
  end loop;
end;
```

und der Aufruf dann einfach so:

```sql
set_item_properties('MY_ITEM', sys.odcinumberlist(visible, enabled, navigable), property_true);
```

Cool, oder ?
