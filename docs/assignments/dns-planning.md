Overview
The objective of this exercise is to use basic knowledge of the Domain Name System (DNS) to define public and private domains for use in upcoming exercises. Follow the attached instructions to determine the organization of your domains according to the given specifications.

Before you start, review this week's readings and DNS-related resources to understand the structure and constraints of legal domain names as well as the distinction between a hostname, a sub-domain, and a domain. Give consideration as well to DNS resource naming schemes.

Objective
For this exercise, you'll select two DNS domains and provide names for resources that would be associated with these domains. 

The first domain that you define will be a public-facing domain. Your public domain is the domain that customers (or in this case, classmates) would use to connect to resources you put on the Internet. 


The second domain we’ll define is for private use only. Our private domain name will be a sub-domain of our public DNS with the restriction that it will only be available on the LAN.
Private domains are incredibly valuable in business networks, where they are  servers, printers, and other resources without having to keep track of IP addresses or to rely on less secure mechanisms like MDNS.

Specifications
All domains created for this assignment must use the .pi TLD, e.g., domain.pi.
Choose a unique name for your public-facing domain name, making sure to satisfy the appropriate constraints for naming a domain.

Choose a private-facing sub-domain within your public-facing domain to use for private DNS (DNS within your own intranet). The name of your subdomain does not have to be unique, since it’ll be under the hierarchy of your unique public-facing domain, e.g., corp.domain.pi or lan.domain.pi.
Define hostnames for important resources within your public and private-facing DNS.
Each of your two domains will have its own name server. 
Internally (private-facing), you may refer to your name server with the same hostname that you’ve already assigned your Pi, e.g., titan.corp.domain.pi.
Externally (public-facing), we typically follow some sort of standardized naming convention for important server roles like our name server, e.g., ns.domain.pi or dns01.domain.pi.
On your public-facing domain, define additional hostnames for the following resources:
An SMTP server (mail server) that will receive mail from other domains. Mail servers also tend to follow a standard naming convention, e.g., mail01.domain.pi.
A webmail server that your users can use to access their inboxes. Pick a user-friendly name that will be easy for users to remember.
A hostname (web server) that customers would use to find your website. Pick a user-friendly name that will be easy for users to remember.
Deliverables
Fill in this provided template  Download this provided templateas guided above. Then with your Linux Networking  GitHub repository you created in the Checkpoints:

Create a new branch for this assignment using 'git checkout -b dns-planning'
Add your DNS Planning Excercise Report.md' to your repository folder. Keep it in Markdown (.md) format, don't export it to PDF.
After adding the files to your repository folder, use 'git add .' to add the files to your commit you will submit
Commit your added files using 'git commit -m <commit message>'
Push your new commit using 'git push origin dns-planning'
Create a new Pull Request for this branch by going to your GitHub repository webpage, selecting the "Pull Requests", selecting "New Pull Request", and selecting your new commit. DO NOT MERGE THE COMMIT INTO MASTER IF ASKED. The pull requests are for us to check over your work before merging it into master.
Submit the link for the Pull Request via Canvas