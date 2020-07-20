---
title: "Datepicker mit mehreren markierten Tagen"
date: 2015-11-15T22:00:52+02:00
lastmod: 2015-11-15T22:00:52+02:00
draft: false
author: ""
authorLink: ""
description: ""
license: ""

tags: []
categories: ["APEX"]
hiddenFromHomePage: false

featuredImage: "/images/mark-multiple-days-in-datepicker/estee-janssens-zni0zgb3bkQ-unsplash.jpg"
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

In einem meiner aktuellen APEX-Projekte muss ich einen Minikalender zeigen und dort bestimmte Tage markieren. Leider gibt es in APEX nur die Option genau ein Datum zu markieren. Nach einer kleinen Google-Recherche stieß ich auf folgenden Beitrag: <http://www.spiceforms.com/blog/highlight-particular-dates-jquery-ui-datepicker>

<!--more-->

Was hier also gemacht wird ist folgendes:

- Array mit den Date-Werten füllen (Index als Datum und Wert = Tooltip)

  ```javascript
  // An array of dates
  var eventDates = {};
  eventDates[ new Date( '04/12/2015' )] = new Date( '04/12/2015' ); eventDates[ new Date( '06/12/2015' )] = new Date( '06/12/2015' ); eventDates[ new Date( '20/12/2015' )] = new Date( '20/12/2015' ); eventDates[ new Date( '25/12/2015' )] = new Date( '25/12/2015' );
  ```

- dem Datepicker eine Funktion mitgeben, die vor dem Anzeigen des Pickers selbst ausgeführt wird

  ```javascript
  // datepicker
  jQuery('#calendar').datepicker(
    {
      beforeShowDay: function(date) {
        var highlight = eventDates[date];
        if (highlight) {
          return [true, "event", highlight];
        } else {
          return [true, '', ''];
        }
      }
    });
  ```

- CSS-Klasse bereitstellen und den zu markierenden Tagen zuweisen

  ```css
  .event a {
    background-color: #42B373 !important;
    background-image :none !important;
    color: #ffffff !important;
  }
  ```

Um das ganze via APEX umzusetzen habe ich einen Pageprocess in APEX angenommen, der vor dem Rendern der Seite läuft:

```sql
BEGIN
  htp.p('<script language="Javascript">');
  htp.p(' var eventDates = {};');
  for cur in (select trunc(oms_time_start) oms_date, count(1) cnt_free_slots
                from otv_meeting_slots, otv_os_op_assignment
               where oms_op_id = oooa_op_id
                 and oooa_os_name = 'Beratung'
                 and oms_ok_mail is null
                 and trunc(oms_time_start) >= trunc(sysdate)
               group by trunc(oms_time_start))
  loop
    htp.p(' eventDates["'||to_char(cur.oms_date, 'YYYYMMDD')||'"] = "'||cur.cnt_free_slots||' freie Slots";');
  end loop;
  htp.p('</script>');
END;
```

Nun habe ich mein Array. Weiter gings mit der Javascript-Methode und das anlegen der CSS-Klasse. Läuft...

![JQuery Datepicker](/images/mark-multiple-days-in-datepicker/datepicker.png "JQuery Datepicker")

Doch dann kam das Problem. Ich konnte nun kein Datum mehr auswählen. Der Picker hat bei der Auswahl eines Tages nichts gemacht. Dementsprechend konnte ich keine abhängigen Events feuern. Nach etwas debuggen und ausprobieren habe ich es nun doch hinbekommen. Zusätzlich zur beforeShowDate-Methode muss die onSelect-Methode miterzeugt werden. Wahrscheinlich hat mein Aufruf diese überschrieben, so das kein onSelect-Event gefeuert wurde.

```javascript
$('#P3_DATE_PICKER_INLINE').datepicker( {
  beforeShowDay: function(date) {
    var dateStr = padStr(date.getFullYear()) +
                  padStr(1 + date.getMonth()) +
                  padStr(date.getDate());

    var highlight = eventDates[dateStr];

    if (highlight) {
      return [true, "event", highlight];
    } else {
      return [false, '', ''];
    }
  },
  onSelect: function(date) {
    alert(date);
  }
});
```
