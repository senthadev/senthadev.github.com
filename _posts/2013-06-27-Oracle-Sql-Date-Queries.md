---
layout: post
title: Oracle Sql Date Queries
---

How to create a date from string value?

    select to_date('2013-06-27', 'yyyy-mm-dd') from dual;

For available formats : [http://psoug.org/reference/date_func.html](http://psoug.org/reference/date_func.html)

- - -

How to get the first day of the month from a random date?

    select trunc(sysdate, 'mon') from dual;
    select trunc(to_date('2013-06-27', 'yyyy-mm-dd'), 'mon') from dual;

this will return 2013-06-01

- - -

How to get all the dates, in a particular month from a random date, into a table of rows?

    select (trunc(sysdate, 'mon') + (level-1)) all_dates
    from dual
    connect by level <= to_number(to_char(LAST_DAY(sysdate), 'dd'));

For example, if the sysdate returns 2013-06-27, then result would be 30 records starting from 2013-06-01..2013-06-30

- - -

How to get start and end dates of all the weeks, starting from monday, of a particular month for a random date?

    select 
        weeks.cdate monday,
        weeks.cdate + 6 sunday
    from (
        select trunc(trunc(sysdate, 'mon') + ((level-1)*7), 'Day') + 1 as cdate
        from dual
        connect by level <= to_number(to_char(last_day(sysdate), 'W'))
    ) weeks
    where to_char(weeks.cdate, 'MM') = to_char(sysdate, 'MM')

For example, if the sysdate returns 2013-06-17, then the result:

<table>
    <tr><td>monday</td><td>sunday</td></tr>
    <tr><td>2013-06-03</td><td>2013-06-03</td></tr>
    <tr><td>2013-06-10</td><td>2013-06-16</td></tr>
    <tr><td>2013-06-17</td><td>2013-06-23</td></tr>
    <tr><td>2013-06-24</td><td>2013-06-30</td></tr>
</table>

trunc(sysdate, 'Day') will return the 'sunday' of that particular week. In order to get monday, we are adding 1 to the received date.


