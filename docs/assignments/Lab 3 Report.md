# Lab 3 Report - Analyze DNS & HTTP in Wireshark

INFO 314
DATE: 
NAME: 

## Overview  
The purpose of this lab is for students to gain hands on experience with DNS. By the end of the lab you will be familiar with:

- `dig`
- DNS queries
- security faults of DNS

We highly recommend using search engines to help you find solutions.

## Part I - Website

1. **What website did you select?**  
   
   
   
## Part II - dig

2. **What addresses did the dig command return? Copy dig results to your report (code block or screenshot).**  
   
   
   
3. **Where did the two IP addresses bring you to?**  
   
   
   
4. **What do these addresses represent and why type of record are they?**  
   
   
   
5. **What are the main types of records a nameserver holds on to?**  
   
   
   
## Part III - Dev Tools

6. **From a quick glance what is the most common file type that you see is requested?**    
   
7. **Approximately how many domains do you see in that list that don't matchup with the website domain you initially visited?**  
   
   
   
8. **Why do you think this page is getting information from other websites?**  
   
   
   
9. **If you had to guess, how many DNS requests do you think were sent in order to fully load this page?**  
   
   
   
## Part IV - Wireshark

10. **Assuming almost all of the DNS requests you see in Wireshark right now are for the one website you visited, how many DNS requests do you see?**  
   
   
   
11. **Overall were there less or more DNS queries than you'd expect?**  
   
   
   
12. **How did you identify the DNS packet associated with the website you visited?**  
   
   
   
13. **Provide a screenshot of the packet (specifically of the DNS information).**  
   
   
   
14. **List the IP addresses you received for the website from the DNS server that resolved your request.**  
   
   
   
15. **Which transport layer protocol (think OSI model) is present within the DNS headers?**  
   
   
   
16. **Compare this DNS response to others in the capture (generate more if needed).**  
   
   
   
17. **Which port number(s) are shared in common across these DNS requests?**  
   
   
   
## Part V - Security

18. **Examine the bytes view of the two packets. Do you see any human readable values in the output?**  
   
   
   
19. **Looking at these two packets and others in your capture, does Wireshark provide any clues about whether or not your DNS is encrypted?**  
   
   
   
20. **Does it provide any clues on whether your web traffic is encrypted?**  
   
   
   

**Attacker:**  

21. **What information might an outside observer be able to glean about your computing activities by capturing your DNS traffic?**  
   
   
   
22. **Was any discernible information revealed (as far as you can tell) through your web traffic?**  
   
   
   
## Deliverables  
**For your deliverable, you will need to submit one file Canvas under the Assignment "Lab 3 - Analyze DNS & HTTP in Wireshark".**  

- **This completed lab report (preferably in PDF format)**