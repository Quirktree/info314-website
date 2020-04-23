# Lab 4 - Analyze DNS & HTTP in Wireshark
[![DNS image](https://www.elegantthemes.com/blog/wp-content/uploads/2018/03/what-is-dns.png)](https://www.elegantthemes.com/blog/wp-content/uploads/2018/03/what-is-dns.png)

[Lab 4 Assignment page on Canvas](https://canvas.uw.edu/courses/1373089/assignments/5369618)

## Overview

The purpose of this lab is for students to gain hands on experience with DNS. By the end of the lab you will be familiar with:

- **`dig`** (or PowerShell's **`Resolve-DnsName`** command)
- DNS queries
- security faults of DNS

We highly recommend using search engines to help you find solutions. As usual, there will be multiple ways to complete this lab. Please use the template provided on Canvas under the Assignment "Lab 3 - Analyze DNS & HTTP in Wireshark" to complete your lab report.



## Part I - website
Pick a popular website that has a lot of content on it. Ideally it should contain advertisements. You will be using it throughout this lab.

**Report**: 

1. What website did you select?

## Part II - dig
Before jumping into more complicated scenarios we would like you to learn how to use the **`dig`** command as it is an incredibly helpful and simple tool.

We've provided some resources at <a href="/resources/dns-clients/#perform-dns-lookups-manually" target="_blank">Perform DNS Loookups Manually</a> to help you get started.

Use dig to lookup the domain name of the website you selected. Open up a new tab in your browser and enter (one of) the IP address(es) returned by dig to discover where it will take you. If dig returned more than one address, open up a second tab and enter it now.

!!! Note 
    Don't panic if you receive an error instead of a live site. Some servers host more than one website and need the name to help route you to the right place. Try another site, e.g., uw.edu, and see if you get different results.

**Report**: 

2. What addresses did the dig command return? Copy dig results to your report (code block or screenshot).

1. What did you observe when you browsed to the IP addresses directly?

1. Aside from **A** records containing IP addresses, did you discover any other DNS records from your query? List at least 3 other record types that DNS manages (use Google to look up other kinds of records if your query returned less than 3 types!)

1. Look closely at the output of the `dig` command. How can you be sure that the query completed successfully? Identify the IP address of the server used to handle your query. If you used PowerShell's `Resolve-DnsName` then you won't see the server IP that answered your request. Instead use `ipconfig /all` and find the IP under `DNS Servers` under your network interface.


## Part III - dev tools
Open up a new tab preferably in a chromium based browser (Chrome, Brave, etc...). Open up the dev tools (right click on page and click Inspect), and go to the *Network* option. 

In that very same tab paste in your selected web page into the URL bar and click enter. You should see all of the **GET** and **POST** requests that the browser made for that webpage load up in the dev tools.

Right-click the header of the Network Log table and select Domain. The domain of each resource is now shown.

[![dev tools show domain](https://developers.google.com/web/tools/chrome-devtools/network/imgs/tutorial/domain.png)](https://developers.google.com/web/tools/chrome-devtools/network/imgs/tutorial/domain.png)


Take some time to scroll through the domains, files, and file types your website was requesting.

**Report**: 

6. From a quick glance what is the most common file type requested?

1. Approximately how many *domains* do you see in that list that don't matchup with the website domain you initially visited?

1. Why do you think this page is getting information from other websites?

1. If you had to guess, how many DNS requests do you think were sent in order to fully load this page?



## Part IV - wireshark
Before examining DNS requests in Wireshark you will want to clear your DNS cache. Instructions on what that means, what it's for, and how to do so can be found here [314-docs/trouble/dns](https://bwalchen.github.io/314-docs/trouble/dns/#clear-dns-cache).


Create a capture of you visiting your website by: 

* Open Wireshark, begin a capture
* Quickly open a new tab and visit your website in your browser
* Once it has fully loaded end your Wireshark capture.

Now **filter** for DNS packets only in the display filter.

* Identify the DNS response containing the information you needed in order to convert your website name into an IP address.
* Use the information contained within that packet for the following deliverable.


!!! Hint
    Learn how to use the search tool to find string content or research display filters that allow you to specify the domain name.

**Report**: 

10. Assuming almost all of the DNS requests you see in Wireshark right now are for the one website you visited, how many DNS requests do you see? 

1. Overall were there less or more DNS queries than you'd expect?

1. How did you identify the DNS packet(s) associated with the website you visited?

1. Provide screenshots of the packet(s) (specifically of the DNS information in Packet Details).

1. List the ip addresses you received for the website from the DNS server that resolved your request.

1. Which *transport layer protocol* ([think OSI model](https://bwalchen.github.io/314-docs/course-prep/osi/)) is used to carry the DNS packet?

1. Compare this DNS response to others in the capture (generate more if needed).

1. Which port number(s) are shared in common across these DNS requests?



## Part V - security
Open two packets in bytes view, a dns and a http packet that is encrypted with tls.

* Double click one of the DNS packets in order to be able to see the bytes view. 
* Remove the dns display filter and replace it with tls (web traffic). 
* Open one of the tls packets in byte view too.

**Report**: 

18. Examine the bytes view of the two packets. Do you see any human
    readable values in the output?

1. Looking at these two packets and others in your capture, does Wireshark provide any clues about whether or not your DNS is encrypted?

1. Does it provide any clues on whether your web traffic is encrypted?

Attacker:

21. What information might an outside observer be able to glean about your computing activities by capturing your DNS traffic?

1. Was any discernible information revealed (as far as you can tell) through your web traffic?