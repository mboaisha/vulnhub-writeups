# Mr Robot

**SPOILERS AHEAD. YOU HAVE BEEN WARNED**

**Difficulty**: Easy
**Link**: https://www.vulnhub.com/entry/mr-robot-1,151/

## Writeup

It's a good practice to run vulnerable virtual machines in an isolated network. Away from production and/or sensitive servers and workstations.

As such, it is best to host the VMs in "Host-Only" Mode in the same subnet.

![Check your settings](img1.PNG)

Once we launch the mr-robot VM. We are greeted with a login screen.

![Interesting](img2.PNG)

Obviously, we do not know the credentials to the box. There must be a way to enter externally (i.e. from network-facing services.)

For a quick network discovery I used *NetDiscover*, which is readily available in Kali Linux.

![Hosts found](img3.PNG)

A default *nmap* scan shows this result for 192.168.56.103:

```
Nmap scan report for 192.168.56.103
Host is up (0.0013s latency).
Not shown: 997 filtered ports
PORT    STATE  SERVICE
22/tcp  closed ssh
80/tcp  open   http
443/tcp open   https
MAC Address: 08:00:27:BA:66:66 (Oracle VirtualBox virtual NIC)
```

Seems about right, Browsing to the IP via *Firefox* gives me a faux boot sequence and a menu:

![whoa](img4.PNG)


There isn't anything obviously exploitable at first glance just cool Mr Robot stuff from the show.

Viewing the page source, there is a cool ascii art but that's about it.

```
<!doctype html>
<!--
\   //~~\ |   |    /\  |~~\|~~  |\  | /~~\~~|~~    /\  |  /~~\ |\  ||~~
 \ /|    ||   |   /__\ |__/|--  | \ ||    | |     /__\ | |    || \ ||--
  |  \__/  \_/   /    \|  \|__  |  \| \__/  |    /    \|__\__/ |  \||__
-->
<html class="no-js" lang="">
  <head>
    

    <link rel="stylesheet" href="css/main-600a9791.css">

    <script src="js/vendor/vendor-48ca455c.js"></script>

    <script>var USER_IP='208.185.115.6';var BASE_URL='index.html';var RETURN_URL='index.html';var REDIRECT=false;window.log=function(){log.history=log.history||[];log.history.push(arguments);if(this.console){console.log(Array.prototype.slice.call(arguments));}};</script>

  </head>
  <body>
    <!--[if lt IE 9]>
      <p class="browserupgrade">You are using an <strong>outdated</strong> browser. Please <a href="http://browsehappy.com/">upgrade your browser</a> to improve your experience.</p>
    

    <!-- Google Plus confirmation -->
    <div id="app"></div>

    
    <script src="js/s_code.js"></script>
    <script src="js/main-acba06a5.js"></script>
</body>
</html>
```

Since we confirmed that this VM is mostly a LAMP stack, I decided to bring out one of my favorite tools. Enter *Nikto*

Luckily, we don't have to fiddle with the tool alot in this case:

```
nikto -h http://192.168.56.103/
```

which gave us plenty of results to go through.

![niktoScan](img5.PNG)

The results tell us two things:

* There is a [robots.txt](https://www.robotstxt.org/robotstxt.html) file
* There is very possibly a Wordpress instance hosted

I decide to check for both before proceeding.

Turns out there is a /robots.txt file. Hold on, there is more to it.

The content of the text file is:

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

*wget* the files and examine them. Turns out the text file does contain the key:
```
073403c8a58a1f80d943455fb30724b9
```

Neat, first key is down. But what is fsocity.dic?

using *cat* shows a **HUGE** of what appears to be a wordlist. I noticed some duplicates so I decided to clean up the wordlist:

```
uniq fsocity.dic > fsocity1.txt
```

Now we look into the possible Wordpress instance, and what do you know, there is one!

![wpfound](img6.PNG)
