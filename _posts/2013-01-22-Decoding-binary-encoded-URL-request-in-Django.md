---
layout: post
title: Decoding binary encoded URL request in Django App
---

How to decode a GET request which contains encoded (%xx quoted hex format) binary data.
For example, following is a GET request:

<table>
    <tr><td>http://mosms.senthadev.com/sendsms/?udh=%02q%00&amp;ud=%00%0B%0A%B0%00%01%00%00%00%00%12%00%01</td></tr>
</table>

This was one of the hurdle that I faced while implementing a Django web application. 
In this request, parameters *udh* and *ud* contains encoded binary data. 

Therefore, to retrieve the values, I used below code. But the return values were in ascii.  

    ud = request.GET.get('ud')
    udh = request.GET.get('udh')
    print('%r' % ud) # u'\x02q\x00'
    print('%r' % udh) # u'\x00\x0b\n\ufffd\x00\x01\x00\x00\x00\x00\x12\x00\x01'

For example,
I was expecting *027100* for ud, but I got *u'\x02q\x00'*.
And this caused an error when I tried to convert the value back to binary using *binascii.b2a_hex(ud)*.
Django threw the following error:

    UnicodeEncodeError: 'charmap' codec can't encode character u'\ufffd' in position 3: character maps to
This thrown for Django 1.4 

How to solve this in a very limited time frame?
And, I used following code:
 
    import urllib, binascii
    from django.http import HttpResponse

    def submit():
        #collecting the GET params into dict
        data = dict(item.rsplit('=') for item in request.META['QUERY_STRING'].rsplit('&'))
        udh = str(binascii.b2a_hex(urllib.unquote(data['udh'])))
        ud = str(binascii.b2a_hex(urllib.unquote(data['ud'])))
    
        #now I have the ud and udh as proper hex string to be converted to binary data.
        #udh : 027100
        #ud : 000b0ab0000100000000120001

