---
layout: post
title: Kannel HTTP Admin
---

For a project, I need to send and recieve SMS's by accessing Mobile Operators SMSC account.
SMSC(SMS Center) is a server which resides on Operators infrastructure and acts as a gateway between internet and Operators network.
This server understands an unique protocol called SMPP (Short Message Peer to Peer).
For a project, I have to use version 3.4 of the SMPP protocol.

Therefore I used Kannel as a gateway between Operators SMSC and our Web Application.
Kannel(www.kannel.org) is an open source WAP/SMS gateway. Once the Kannel gateway is configured properly, it is accessed via HTTP inorder to send/receive SMS and administration purpose.

This article covers how to admin the kannel via HTTP. This covers for release 1.4.3

<table>
    <tr><td>To check the status of the kannel</td></tr>
    <tr><td>http://kannelhost:port/status.txt
        To get the response in xml then use http://kannelhost:port/status.xml</td></tr>
    <tr><td>To restart entire SMSC connections</td></tr>
    <tr><td>http://kannelhost:port/restart?password=xxx , must use the admin password.
        It will re-read the configuration file and start all the SMSC connetions</td></tr>
    <tr><td>To change the log level of the bearerbox in runtime</td></tr>
    <tr><td>http://kannelhost:port/log-level?level=1&amp;password=xxx ,  
        available levels are : (0 = is for 'debug', 1 = 'info', 2 = 'warning, 3 = 'error' and 4 ='panic').</td></tr>
    <tr><td>To stop a certain SMSC</td></tr>
    <tr><td>http://kannelhost:port/stop-smsc?smsc=smsc_id&amp;password=xxx , 
        smsc_id is the link name which needs to be stop. Once invoked, it will kill the SMSC connection.</td></tr>
    <tr><td>To start a dead SMSC connection</td></tr>
    <tr><td>http://kannelhost:port/start-smsc?smsc=smsc_id&amp;password=xxx , 
        it doesn't re-read the configuration from the file and use the data in the memory to start the SMSC connection.</td></tr>
</table>

There are more http commands available, but normally I don't used them. 
They are : store-status, flush-dlr, reload-lists