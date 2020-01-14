# Lab 3
_Last edited 04/16/2019_

[![DNS image](https://www.elegantthemes.com/blog/wp-content/uploads/2018/03/what-is-dns.png)](https://www.elegantthemes.com/blog/wp-content/uploads/2018/03/what-is-dns.png)


## Overview
The purpose of this lab is for students to gain hands on experience with DNS. By the end of the lab you will be familiar with:

- `dig`
- DNS queries
- security faults of DNS

We highly recommend using search engines to help you find solutions.As usual, there will be multiple ways to complete this lab, and we expect you to follow proper assignment formatting ([assignment template](https://bwalchen.github.io/314-docs/assignments/)).

<br>

## Part I - website
Pick a popular website that has a lot of content on it. Ideally it should contain advertisements. You will be using it throughout this lab.

**Deliverable**: What website did you select?

<br>

## Part II - dig
Before jumping into more complicated scenarios we would like you to learn how to use the **`dig`** command as it is an incredibly helpful and simple tool.

Go to  <a href=" https://bwalchen.github.io/314-docs/trouble/dns/" target="_blank"> 314-docs/trouble/dns/</a> and scroll down to the `dig` section.

Run dig against the website you selected. Open up two tabs in your browser, grab two different IP addresses that were returned to you, and put one in each tabs' URL bar. 

**Deliverable** 

* What addresses did the dig command return? Copy dig results to your report (code block or screenshot).
* Where did the two IP addresses bring you to?
* What do these addresses represent and why type of record are they?
* What are the main types of records a nameserver holds on to?

<br>

## Part III - dev tools
---
Open up a new tab preferably in a chromium based browser (Chrome, Brave, etc...). Open up the dev tools (right click on page and click Inspect), and go to the *Network* option. 

In that very same tab paste in your selected web page into the URL bar and click enter. You should see all of the `GET` and `POST` requests that the browser made for that webpage load up in the dev tools.

Right-click the header of the Network Log table and select Domain. The domain of each resource is now shown.
[![dev tools show domain](https://developers.google.com/web/tools/chrome-devtools/network/imgs/tutorial/domain.png)](https://developers.google.com/web/tools/chrome-devtools/network/imgs/tutorial/domain.png)


Take some time to scroll through the domains, files, and file types your website was requesting.

**Deliverable**

* From a quick glance what is the most common file type that you see is requested?
* Approximately how many *domains* do you see in that list that don't matchup with the website domain you initially visited?
  * Why do you think this page is getting information from other websites?
  * If you had to guess, how many DNS requests do you think were sent in order to fully load this page?



<br>
<br>

## Part IV - wireshark
---

!!! warning "Important: clear DNS cache" 
    Before examining DNS requests in Wireshark you will want to clear your DNS cache. Instructions on what that means, what it's for, and how to do so can be found here [314-docs/trouble/dns](https://bwalchen.github.io/314-docs/trouble/dns/#clear-dns-cache).


Create a capture of you visiting your website by: 

* Open Wireshark, begin a capture
* Quickly open a new tab and visit your website in your browser
* Once it has fully loaded end your Wireshark capture.

Now **filter** for DNS packets only in the display filter.

* Identify the DNS response containing the information you needed in order to convert your website name into an IP address.
* Use the information contained within that packet for the following deliverable.


**Deliverable**

* Assuming almost all of the DNS requests you see in Wireshark right now are for the one website you visited, how many DNS requests do you see? 
  * Overall were there less or more DNS queries than you'd expect?
* How did you identify the DNS packet associated with the website you visited?
  * Provide a screenshot of the packet (specifically of the DNS information).
* List the ip addresses you received for the website from the DNS server that resolved your request.
* Which *transport layer protocol* ([think OSI model](https://bwalchen.github.io/314-docs/course-prep/osi/)) is present within the DNS headers?
* Compare this DNS response to others in the capture (generate more if needed).
  * Which port number(s) are shared in common across these DNS requests?
  


<br>
<br>

## Part V - security
Open two packets in bytes view, a dns and a http packet that is encrypted with tls.
* Double click one of the DNS packets in order to be able to see the bytes view. 
* Remove the dns display filter and replace it with tls (web traffic). 
* Open one of the tls packets in byte view too.

**Deliverable**

* Examine the bytes view of the two packets. Do you see any human
readable values in the output?
  * Looking at these two packets and others in your capture, does Wireshark provide any clues about whether or not your DNS is encrypted?
  * Does it provide any clues on whether your web traffic is encrypted?
* Attacker:
  * What information might an outside observer be able to glean about your computing activities by capturing your DNS traffic?
  * Was any discernible information revealed (as far as you can tell) through your web traffic?