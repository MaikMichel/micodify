---
title: "Arraylike Functions in PL/SQL"
date: 2019-08-01T21:11:15+02:00
lastmod: 2019-08-01T21:11:15+02:00
draft: false
description: "build arrays as arguments"


tags: ["plsql","tipps"]
categories: ["PLSQL"]
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
Some time ago, when I had to take care of Oracle Forms, I was annoyed that in Forms you always have to call several procedures for switching on and off. So you have to drop the following lines to show a text field again.

<!--more-->

```sql
set_item_property('MY_ITEM', visible, property_true);
set_item_property('MY_ITEM', enabled, property_true);
set_item_property('MY_ITEM', navigable, property_true);
```

And the whole thing is then repeated per item, of course. It would be cool here if you could operate all items at once. And all this would be the same as in other languages, for example Java or JavaScript. Oracle gives us special types for this:

**odcivarchar2list** and **odcinumberlist**

With these you can directly create a variable list of parameters and pass it to a method. This is what a procedure that simply sets all passed properties with a corresponding value would look like:

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

and the call then just like that:

```sql
set_item_properties('MY_ITEM', sys.odcinumberlist(visible, enabled, navigable), property_true);
```

Cool, right?
