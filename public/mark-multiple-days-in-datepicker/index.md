# Datepicker with multiple tagged days


in one of my current Apex projects I have to show a mini calendar and mark certain days there. Unfortunately there is only the option to mark exactly one date in APEX. After a little Google research I came across the following article:
<http://www.spiceforms.com/blog/highlight-particular-dates-jquery-ui-datepicker>

<!--more-->

So what is done here is the following:

- Fill array with date values (index as date and value = tooltip)

  ```javascript
  // An array of dates
  var eventDates = {};
  eventDates[ new Date( '04/12/2015' )] = new Date( '04/12/2015' ); eventDates[ new Date( '06/12/2015' )] = new Date( '06/12/2015' ); eventDates[ new Date( '20/12/2015' )] = new Date( '20/12/2015' ); eventDates[ new Date( '25/12/2015' )] = new Date( '25/12/2015' );
  ```

- Provide the data picker with a function that is executed before the picker itself is displayed.

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

- Provide CSS class and assign it to the days to be marked

  ```css
  .event a {
    background-color: #42B373 !important;
    background-image :none !important;
    color: #ffffff !important;
  }
  ```

To do this via APEX, I have adopted a pageprocess in Apex that runs before rendering the page:

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

Now I have my array. The next step was to use the Javascript method and create the CSS class. Runs…

![JQuery Datepicker](/images/mark-multiple-days-in-datepicker/datepicker.png "JQuery Datepicker")

But then came the problem. I could no longer select a date. The picker didn’t do anything about selecting a day. Accordingly, I couldn’t fire dependent events. After some debugging and testing, I finally managed it. In addition to the beforeShowDate method, you must also create the onSelect method. Probably my call overwrote them, so no onSelect event was fired.

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

