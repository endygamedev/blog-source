---
layout: post
title:  "💻 Build your own home server"
date:   2021-11-19 23:03:00 +0300
categories: jekyll update
tags: cs
---

<h2><a id="content">Table of contents </a></h2>
<ul style="list-style-type: none; margin: 0 0 4% 4%;">
    <a href="#chapter1"><li>[0] Introduction</li></a>
    <a href="#chapter2"><li>[1] Operating System</li></a>
    <a href="#chapter3"><li>[2] Webdmin</li></a>
    <ul style="list-style-type: none; margin: 0 0 0 3%;">
        <a href="#chapter3.1"><li>[2.1] Installing Webdmin</li></a>
    </ul>
    <a href="#chapter4"><li>[3] DNS Records</li></a>
    <a href="#chapter5"><li>[4] Port Routing</li></a>
    <a href="#chapter6"><li>[5] Nginx</li></a>
    <ul style="list-style-type: none; margin: 0 0 0 3%;">
        <a href="#chapter6.1"><li>[5.1] Installing Nginx</li></a>
        <a href="#chapter6.2"><li>[5.2] Adjusting the Firewall</li></a>
        <a href="#chapter6.3"><li>[5.3] Checking Web Server</li></a>
        <a href="#chapter6.4"><li>[5.4] Setting Up Server Blocks</li></a>
        <a href="#chapter6.5"><li>[5.5] Certification</li></a>
    </ul>
    <a href="#chapter7"><li>[6] Conclusion</li></a>
    <a href="#chapter8"><li>[7] References</li></a>
</ul>

<h2><a id="chapter1" href="#content">[0] Introduction</a></h2>

<p align="justify">I had a fairly old laptop for a long time and no one cared about it. I was sorry that the `piece of iron` was idle. And I got a great idea that it would be cool to build my own server, and I will learn how to do it. So this article was born.</p>

<p align="justify">I decided to use it as cloud drive and hosting for my website <sub>(this website)</sub>.</p>

<p align="center"><img src="/assets/server/laptop.jpg" width="50%"></p>

<p align="justify">And yeah, that's my boo <sub>a.k.a. War Machine 4001</sub> — <i>ASUS Eee PC 1005PE</i>, for '05 it was very powerful and mobile laptop, but now its a 🗑️.</p>


<h2><a id="chapter2" href="#content">[1] Operating System</a></h2>

<p align="justify">It was decided to install <i><a href="https://ubuntu.com/download/server" target="blank_">Ubuntu Server 20.04.3 LTS</a></i> because it's very stable OS for servers. You can check full setup in this video:</p>

<p align="center"><iframe width="70%" height="315px" src="https://www.youtube.com/embed/lXcfKTNObOo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>

<p align="justify">If you prefer to use a wireless connection, then after OS installation you should install <code>NetworkManager</code>. In my case it's very easy:</p>

~~~bash
λ sudo apt-get update -y                      # update repository
λ sudo apt-get upgrade -y                     # upgrade utilities
λ sudo apt-get install NetworkManager -y      # install NetworkManager
λ sudo systemctl start NetworkManager         # start service
λ sudo systemctl enable NetworkManager        # setup autostart
~~~

<p align="justify">If you want to connect public WiFi you should type this:</p>

~~~bash
λ nmcli device wifi connect "connection_name"
~~~

<p align="justify">where <u>connection_name</u> is name of WiFi network.</p>

<p align="justify">And if you want to connect private WiFi you should type this:</p>

~~~bash
λ nmcli device wifi connect "connection_name" password "password"
~~~

<p align="justify">where <u>connection_name</u> is name of WiFi network and <u>password</u> is password for WiFi.</p>

<p align="justify">And if we type <code>ip a</code> we will see IP address of our server. To test local connection you can type <code>ssh username@ip_address</code>, where <u>username</u> is your server's username and <u>ip_address</u> is server's ip address. After that, you can remotely control your server by connecting locally through the terminal.</p>

<p align="justify">The last step is to setup UFW:</p>

~~~bash
λ sudo systemctl start ufw      # start service
λ sudo systemctl enable ufw     # setup autostart
λ sudo systemctl status ufw     # check service status
~~~

<h2><a id="chapter3" href="#content">[2] Webdmin</a></h2>

<p align="justify">Webmin is a convenient web interface for remote computer management.</p>

<h3><a id="chapter3.1" href="#content">[2.1] Installing Webdmin</a></h3>
~~~bash
λ sudo apt update -y
λ sudo apt install software-properties-common apt-transport-https wget
λ wget -q http://www.webmin.com/jcameron-key.asc -O- | sudo apt-key add -
λ sudo add-apt-repository "deb [arch=amd64] http://download.webmin.com/download/repository sarge contrib"
λ sudo apt install webmin
λ sudo ufw allow 10000
~~~

<p align="justify">After that, it will be possible to control the server from any device in the local network by clicking on the link <code>http://server_ip_address:10000</code> in the browser.</p>

<p align="center"><img src="/assets/server/webdmin.png" width="100%"></p>

<h2><a id="chapter4" href="#content">[3] DNS Records</a></h2>

<p align="justify">It may so happen that your provider company didn't give you a public external IP address. If this happens, then you must ask him to be connected. I faced such a situation.</p>

<p align="justify">Ok, after you got the external IP address you should go to the website where you bought the domain and make some records.</p>

~~~bash
A   *      🠒 your_public_IP_address     # `A` DNS record for all others subdomains
A   @      🠒 your_public_IP_address     # `A` DNS record for `@`
A   www    🠒 your_public_IP_address     # `A` DNS record for `www`
~~~

<h2><a id="chapter5" href="#content">[4] Port Routing</a></h2>

<p align="justify">It is worth noting that we have a public external IP that match to all local devices connected to our network. In order to link to our server, where Nginx will work for us, you need to register <i>Port Forwarding</i>.</p>

<p align="justify">I have a <i>Netis WF2780</i> router. To configure it, I have to open in the browser router's IP address, for example: <code>http://192.168.1.1</code>. After thar I go to <code>`Advanced Settings`</code>. In order to give a permanent IP address to the server, I have to do the following steps: <code>`Network` 🠒 `LAN` 🠒 `DHCP Client List` 🠒 *select server* 🠒 `Operations` 🠒 `Reserve`</code>. Now, even if our server is disconnected for a long time, the router will remember its MAC address and we will not have to change the settings in the future.</p>

<p align="justify">Let's move on to <i>Port Forwarding</i> by following these steps: <code>`Forwarding` 🠒 `Virtual Servers`</code> and you need to add 3 rows. As an example, let's take that the local IP address of our server is <code>192.168.1.8</code>.</p>

<table>
  <tr>
    <th>Description</th>
    <th>IP Address</th>
    <th>Protocol</th>
    <th>External Port</th>
  </tr>
  <tr>
    <td>ssh</td>
    <td>192.168.1.8</td>
    <td>TCP</td>
    <td>22</td>
  </tr>
  <tr>
    <td>website</td>
    <td>192.168.1.8</td>
    <td>TCP</td>
    <td>80</td>
  </tr>
  <tr>
    <td>website</td>
    <td>192.168.1.8</td>
    <td>TCP</td>
    <td>443</td>
  </tr>
</table>

<p align="justify">Now we can move on to the next stage.</p>

<h2><a id="chapter6" href="#content">[5] Nginx</a></h2>

<p align="justify">The next step is to host our website and for this I used Nginx. Soooo, let's go ahead to the setup.</p>

<h3><a id="chapter6.1" href="#content">[5.1] Installing Nginx</a></h3>

~~~bash
λ sudo apt update
λ sudo apt install nginx
~~~

<h3><a id="chapter6.2" href="#content">[5.2] Adjusting the Firewall</a></h3>

~~~bash
λ sudo ufw allow "Nginx Full"
λ sudo ufw status               # check UFW status
~~~

<h3><a id="chapter6.3" href="#content">[5.3] Checking Web Server</a></h3>

~~~bash
λ sudo systemctl status nginx   # check Nginx status
λ curl -4 icanhazip.com         # get your public IP address
~~~

<p align="justify">When you have your server’s IP address, enter it into your browser’s address bar: <code>http://your_server_ip</code> and you should receive the default Nginx landing page. If you are on this page, your server is running correctly and is ready to be managed.</p>

<h3><a id="chapter6.4" href="#content">[5.4] Setting Up Server Blocks</a></h3>

<p align="justify">We load the sources of our website:</p>

~~~bash
λ cd --
λ git clone https://github.com/endygamedev/endygamedev.github.io.git
~~~

<p align="justify">Create the directory for <code>ebronnikov.xyz</code> <sub>(if you are creating your own website, you should replace `ebronnikov.xyz` with your domain name)</sub> as follows, using the <code>-p</code> flag to create any necessary parent directories:</p>

~~~bash
λ sudo mkdir -p /var/www/ebronnikov.xyz/html
~~~

<p align="justify">Next, assign ownership of the directory with the <code>$USER</code> environment variable:</p>

~~~bash
λ sudo chown -R $USER:$USER /var/www/ebronnikov.xyz/html
~~~

<p align="justify">The permissions of your web roots should be correct if you haven’t modified your umask value, which sets default file permissions. To ensure that your permissions are correct and allow the owner to read, write, and execute the files while granting only read and execute permissions to groups and others, you can input the following command:</p>

~~~bash
λ sudo chmod -R 755 /var/www/ebronnikov.xyz
~~~

<p align="justify">Next, copy all your website sources into <code>html</code> directory:</p>
~~~bash
λ sudo cp -r ~/endygamedev.github.io/* /var/www/ebronnikov.xyz/html/
~~~

<p align="justify">To avoid a possible hash bucket memory problem that can arise from adding additional server names, it is necessary to adjust a single value in the <code>/etc/nginx/nginx.conf</code> file. Open the file:</p>

~~~bash
λ sudo vim /etc/nginx/nginx.conf
~~~

<p align="justify">Find the <code>server_names_hash_bucket_size</code> directive and remove the <code>#</code> symbol to uncomment the line:</p>

~~~bash
/* Filename: /etc/nginx/nginx.conf */

...
http {
    ...
    server_names_hash_bucket_size 64;
    ...
}
...
~~~

<p align="justify">Setup default configuration:</p>

~~~bash
λ sudo vim /etc/nginx/sites-available/default
~~~

<p align="justify">And edit such row:</p>

~~~bash
/* Filename: /etc/nginx/sites-available/default */

server {
    ...
    # listen 80 default_server;
    # listen [::]:80 default_server;
    listen 80;
    server_name ebronnikov.xyz www.ebronnikov.xyz;
    ...
    # root /var/www/html;
    root /var/www/ebronnikov.xyz/html;
    ...
}
~~~

<p align="justify">Restart Nginx service:</p>

~~~bash
λ sudo service nginx restart
~~~

<p align="justify">After that you can type: <code>http://ebronnikov.xyz</code> into your browser’s address bar and get your webpage. <u>BUT</u> we ended up with an uncertified website, so we need to move on to the certification stage.</p>

<h3><a id="chapter6.5" href="#content">[5.5] Certification</a></h3>

<p align="justify">We will use the utility <i><a href="https://github.com/certbot/certbot" target="blank_">Certbot</a></i>.</p>

<p align="justify">Installing Certbot:</p>

~~~bash
λ curl -o- https://raw.githubusercontent.com/vinyll/certbot-install/master/install.sh | bash
~~~

<p align="justify">Setup certification:</p>

~~~bash
λ sudo certbot --nginx -d ebronnikov.xyz -d www.ebronnikov.xyz
λ sudo certbot renew --dry-run      # Only valid for 90 days
~~~

<p align="justify">After that, you can go to the website <code>ebronnikov.xyz</code> and everything should work correctly.</p>


<h2><a id="chapter7" href="#content">[6] Conclusion</a></h2>

<p align="justify">As a result, we got a working server on which you can store some files and host various websites, you could still install a database, but this is another time.</p>

<h2><a id="chapter8" href="#content">[7] References</a></h2>
<ul style="font-size: 18px; list-style-type: none; margin: 0 0 0 3%;">
    <a href="https://youtu.be/lXcfKTNObOo" target="blank_"><li>[1] Turning an OLD PC/Laptop into a Media Server!</li></a>
    <a href="https://youtu.be/1cKF_pJ8b_o" target="blank_"><li>[2] Learn how to setup a domain with Nginx in less than 4 minutes!</li></a>
    <a href="https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04" target="blank_"><li>[3] How To Install Nginx on Ubuntu 20.04</li></a>
    <a href="https://gist.github.com/bradtraversy/cd90d1ed3c462fe3bddd11bf8953a896" target="blank_"><li>[4] Node.js Deployment</li></a>
    <a href="https://github.com/vinyll/certbot-install#how-to-install" target="blank_"><li>[5] Certbot installer</li></a>
</ul>
<p align="center">and a lot of Google ...</p>
<hr>
<h2 align="center">✌️ i didn’t think of what to write ...</h2>