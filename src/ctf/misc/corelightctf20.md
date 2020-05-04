# CorelightOS CTF Writeup

This was a short two hour CTF hosted by Corelight over multiple sessions. Due to an extra space character while entering a flag, I got stuck at one problem for way too long and did not complete either PCAP 1 or 2.

You could either `ssh` and use the Linux command line to grep through log files or use an SIEM like `Splunk`.

I used both, so for now I'll only write the flag for each question then maybe one day revisit with the bash-fu. 

[json_logs.tgz](assets/json_logs.tgz)

[tsv_logs.tgz](assets/tsv_logs.tgz)

## PCAP 1

### Resources

To access the dataset, use any of the following:

    Splunk (no login required)
    Elastic (credentials below)
    SSH to <REDACTED>, and look in ~/tsv_logs/pcap1

Credentials:

    Username: <REDACTED>
    Password: <REDACTED>

Once you've logged to the tool of your choice, enter the flag "FreePointsPlease" below to unlock the questions for this scenario. You can always return to this question later

`FreePointsPlease`

### Question 1
An HTTP request is made to a specific PHP page. What is the name of that page?
`whoami`

### Question 2
What is one of the IP addresses where that PHP page was hosted?
`66.228.32.31`

### Question 3
What is the IP address that mail.ventascintas.com resolved to?
`142.4.4.112`

### Question 4
What was the IP address that w01099b7.kasserver.com resolved to?
`85.13.157.226`

### Question 5
There is a fairly "generic" X.509 self-signed certificate from a company in London. Can you figure out the domain? (Format: domainname.com)
`example.com`

### Question 6
There is a unique JA3 hash associated with this "generic" certificate. What is that hash?
`35492f143de0f906215ea3aaf6ee0a74`

### Question 7
What was the most recent JA3S hash associated with the previous JA3?
`f2e1706526fe0692ee36be58110ffc83`

### Question 8s
What specific encryption algorithm was used with the aforementioned certificate?
`TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA`

### Question 9
Let's pick apart that X.509 further; there is a unique City that only shows up with this suspicious certificate. What is that City?
`London`
### Question 10
What department does that previous certificate allegedly correspond to?
`IT Department`

### Question 11
What is the SHA1 of the previously-mentioned X.509 certificate?
`answer not confirmed`

### Question 12
As part of this traffic, there were two executables downloaded from 104.168.98.206. What is the SHA1 of the most recently downloaded?
`answer not confirmed`

### Question 13
An executable was downloaded from 124.158.6.218. What was the name of that executable?
`i5pv72yr.exe`

### Question 14
There was a document downloaded in this PCAP that has some Spanish flair to it. What was that document's name?
`answer not confirmed`

### Question 15
What IP address did mail.casaroyal.cl resolve to?
`200.75.0.9`

### Question 16
There's an email address sending suspicious emails (that maybe, maybe looks like a FireEye competitor). What is that email address?
`crogstrike@gchteam.com`

### Question 17
An analyst was reading an introduction to threat hunting and came across a User-Agent string that looked familiar: "WinHTTP sender". What is the hostame of the infected host?
`SKINNER-WIN-PC`

### Question 18
Let's pivot on some metadata. There are some weird user agents in this PCAP; which "WinHTTP" one only shows up once?
`WinHTTP sender/1.0`

### Question 19
I did some JA3S hunting; there are some suspicious domains in there associated with some SMTP traffic. There's one JA3S that appears to be associated with port 80. What is the organization of the issuer, owned by GoDaddy?
`Starfield Technologies\\, Inc.`

### Question 20
We've heard from intel that another suspicious document has been found. Email subjects included the word "dossier" - what was the name of the attachment?
`INF 17844.doc`

### Question 21
Reverse engineering team came back; there was an odd executable in the PCAP. They've provided a SHA1 indicator of 026064006b987ed951ffce4f03c4394f557bf588. Can you determine what the downloaded file name was?
`i5pv72yr.exe`

## PCAP 2

### Resources

To access the dataset, use any of the following:

    Splunk (no login required)
    Elastic (credentials below)
    SSH to <REDACTED>, and look in ~/tsv_logs/pcap2

Credentials:

    Username: <REDACTED>
    Password: <REDACTED>

Once you've logged to the tool of your choice, enter the flag "FreePointsPlease" below to unlock the questions for this scenario. You can always return to this question later

`FreePointsPlease`

### Question 1
There are multiple site using Let's Encrypt - what is one of the Subject Names?
`tile.openstreetmap.org`

### Question 2
Looking at all of the traffic, what is the unique JA3 hash that was observed?
`bc6c386f480ee97b9d9e52d472b772d8`

### Question 3
Uh-oh, looks like we have some unencrypted traffic! There were some requests for /en/www/. What hostname was this to?
`afroamericanec.bit`

### Question 4
There was a particular MIME type of which only two files were observed. What is that MIME type?
`answer not confirmed`

### Question 5
What IP address was that MIME type downloaded from?
`answer not confirmed`

### Question 6
A server response appears to be using stenography to hide something in a GIF image. Zeek data can be used to identify a mismatch in the MIME type and filename to help us find a find the suspicious URI that returns this image.
What was the full URI that corresponded to the newest "GIF"?
`/pixel.gif`

### Question 7
There is one odd HTTP request that did not have a corresponding server response code. What was the server IP address of this HTTP request/response pair?
`188.165.62.40`

### Question 8
What MIME type corresponded to this odd request?
`image/png`

### Question 9
Let's go back and revisit one of those Let's Encrypt sites. What is the IP address for the JA3S hash of e35df3e00ca4ef31d42b34bebaa2f86e ?
`93.95.100.178`

### Question 10
What is the two-letter country code where this IP is located?
`RU`

### Question 11
Who is listed as one of the administrative contacts, indicated by FVV36-RIPE?
`REDACTED for privacy`
