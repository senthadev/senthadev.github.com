---
layout: post
title: Replacing a SMS which is already been sent
---

Let me explain this topic with the below given scenario.

Company "VirtualA" has implemented a two step authentication using sms to secure their web application.
How does it work? basically, when the user enter his/her login credential, application will generate a single use token and sent it over to user via sms.
And the user has to enter the received token to continue. If the token is mismatched or expired then the login will rejected.

Users are happy with this security measure until those piled up sms starts to cause problem to the user.
How can we stop this sms pile up. Once option is to use by sending replaceable sms.
This means, at any time, user's mobile will contain only one sms from "VirtualA".

According the 3Gpp TS 23.040 specification (which is a technical realization of a SMS ), 
we can set the TP-PID field bits to achieve the sms replacement. We must note that this feature is optional.
Which means Mobile OS/(U)SIM has to implement this feature as mentioned in the 3gpp specification.
Well the mobiles(windows phone/android/iphone/nokia) which I tested supports it.

>TP-PID (TP-Protocol Identifier) says,
>When bit 7 = 0, bit 6 = 1 and rest of the bits 5..0 are used as defined below:
>
>01000000 (0x40)- Short message type 0 (This is the normal text sms)
>>01000001 (0x41) - Replace short message type - 1
>>01000010 (0x42) - Replace short message type - 2
>>01000011 (0x43) - Replace short message type - 3
>>01000100 (0x44) - Replace short message type - 4
>>01000101 (0x45) - Replace short message type - 5
>>01000110 (0x46) - Replace short message type - 6
>>01000111 (0x47) - Replace short message type - 7

This means we could have 7 replaceable sms, for example sending a long sms (more than 160 characters).

How will the mobile identifies the previous sms which need to be replace when a new sms is received?
It will use the originating address + pid value combination to find and replace.

"VirtualA" uses 27000 as an originating address to send the token sms's.
And use Kannel SMS gateway to submit sms to smsc. Below given url is a sample url to send sms with pid values.

>http://token.viruala.io:12000/cgi-bin/sendsms?username=xx&password=xx&from=27000&to=47xxxxxxx&smsc=toekn&coding=0&pid=65&text=124323

parameter ‘pid’ represents the TP-PID, and its value is 65(in decimal, which is Replace short message type - 1).
User will receive a sms text = 124323

And, when the user logins again, application will send another token sms,

>http://token.viruala.io:12000/cgi-bin/sendsms?username=xx&password=xx&from=27000&to=47xxxxxxx&smsc=toekn&coding=0&pid=65&text=tgr435

Now the mobile will replace the previous sms text "124323" with "tgr435".
And mobile will contain only one sms with the latest token.

Hope the SMS providers provide this options to their customers.
