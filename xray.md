# All-on-443
XRAY-REALITY-VISION-SELF-STEAL + VLESS-CDN-WS + WEBSITE + WEBPANEL SIMULTANEOUSLY ALL ON port 443

**Goal:**
Reality config, CDN config, panel and HTML site all simultaneously on port 443, using a clean IP and domain. **Note:** If you thought you had a clean IP, pinged it and had no issues (for Reality), or thought you had a completely clean domain (for CDN) but it was actually previously flagged without your knowledge, be assured it will be blocked again if traffic goes through it. This often happens with Hetzner or Aeza subnets for IPs.

The Reality config will be on port 443 and Nginx on port 8443.

The domain used as the SNI for Reality should have its CDN proxy tick turned off. The script will get a certificate for it on our server.

The SNI field in the Reality inbound should be this domain with the tick off.

In the Reality inbound, dest should be set to 127.0.0.1:8443. Reason: Xray will send any non-Reality connection to dest, so when someone other than a Reality user's request reaches it, it will send it to Nginx without any changes.

This includes all requests from CDN inbounds, requests from people's browsers to open our domain, and your requests to open the panel which includes the domain and panel path. They reach Nginx without any changes. So this means if you have Nginx skills, you can bring up Telegram proxy on 443 with fake TLS and even SSH on 443, but I don't know how to do that yet and found the SSLH tool for this.

Why should Xray manage port 443 instead of Nginx? In the Russian-language core group, it was concluded that having the core itself be the program facing unauthorized requests (both from people and the filtering system) is better than Nginx. Otherwise, you can send Reality requests to the core with block stream and pre read, and have Nginx manage requests on 443.

Friends asked why Reality can't be put behind a CDN. The reason is the use of tcp(stream). The CDN wants to cache content but we want everything to reach the server instantly. That's why ws and grpc or split are used in CDN inbounds. These are themselves on a tcp connection, but the transfer is done in a message-oriented way.

Our problem with foreign CDNs was the detection of tls-in-tls and sometimes utls (unless you bring up a CDN config that is plain http). I myself passed 750GB through a domain with Cloudflare, then the site would come up but the VPN wouldn't connect, only a real browser connection would work. (The solution is browser dialer, 2dust said he's not interested in implementing it as it's difficult for him). Of course, I later implemented this same Reality method and it worked without issues.

The script below from GFW4FUN installs and configures the panel, Nginx and gets the domain's security certificate for us.
https://github.com/GFW4Fun/x-ui-pro

In the image below, Hoffenheim is on CDN and the other two are direct, all simultaneously on one server. (Turning off Arvan after 50GB had another reason but was good for testing. So you know you can simultaneously use Arvan Abrdarak Zas Araz and Cloudflare Gcore Fastly CDNs and all on one port 443, all TLS. The necessary and sufficient condition is support for websocket or splithttp.)

Explanation of port 80 image: If you look at the Nginx config, it also listens on port 80, which is why when the panel goes down on 443, when you remove the s from https it comes up. As a result, you can have a CDN config with port 80 without interference from the Reality inbound occupying 443. An unencrypted config. Provided that in Cloudflare, always use https and hsts are **disabled** and the encryption mode is not on Full(strict) and Strict. Note, with the logic explained in [Towards the Stars](https://github.com/Saleh-Mumtaz/Proxy-Wars/blob/main/first-impact.md) by Johann, even if you use this, Telegram or Google or any other site won't put aside their encryption, so the data seen in the channel is encrypted, but it has a **very big flaw** which is why you **should not** use this except for testing some things or in very exceptional circumstances, and that is that all Handshake stages are also seen which is not good. All five packets are seen completely. It **doesn't have** the tls-in-tls problem but well it doesn't have fingerprint either. (To solve this issue you can use Iranian CDN) **So the consequences of using it are on you**. Creating its config in the **panel** will be similar to the TLS one (just the external proxy port should be 80, there it's 443), but, in the user config unlike that security section is empty. You'll understand what I mean when you go further down.

You can use any CDN that supports websocket or if it doesn't, supports splithttp, you can use them simultaneously too.
For each CDN you should naturally enter a separate domain.

The problem with this method is limiting you to one Reality inbound, but with short id and uuid it's not an issue. If you want a connection with little disruption, use Iranian CDN but connect directly as much as possible.

## Setup

First we disable the server firewall.
```
ufw disable
```

We use this Chinese friend's script to install and configure the panel, Nginx and domain.
```
sudo su -c "$(command -v apt||echo dnf) -y install wget;bash <(wget -qO- raw.githubusercontent.com/GFW4Fun/x-ui-pro/master/x-ui-pro.sh) -panel 1 -cdn off"
```
Be aware that his script installs a lot of extra things like Tor, Psiphon, Warp, V2ray panel, sets up daily cron jobs to restart Nginx, Xray core and monthly renewal of security certificates.

**Copy the link it gives for the panel**, remove the s from https, port 80 is in Nginx's hands, and manage the panel with this link until full setup. After finishing, you can enter with the same https link without port. Before closing this http address, check in another tab if the https panel comes up or not.

His script can be run many times to install subsequent domains, but it keeps changing the code and suddenly you might see it messed up something and the server went down completely. I myself removed everything extra it has, from one version onwards its Nginx settings didn't change anymore, and I think we don't need more than that anymore.

**Important: Be sure to bring up the fake site**, in the video tutorial and previous text tutorial by myself there was no emphasis but this time I emphasize run this command and it brings up an HTML site itself.
```
bash <(wget -qO- raw.githubusercontent.com/GFW4Fun/x-ui-pro/master/x-ui-pro.sh) -RandomTemplate yes
```

## Nginx

Now we change the Nginx config for the domain:
```
sudo nano /etc/nginx/sites-available/yourdomain.com
```

Change the 443s to 8443
You can use unix domain socket for connecting Nginx and Xray, it's easy but I didn't bring it here. It greatly reduces the number of local TCP connections between Nginx and Xray, I saw 3000 connections in the panel, with implementing this method it came to 50. The reason for my suggestion is that the Xray creator and Linux wiki said it performs better with local port and shows itself very well in high number uses.

Test the new Nginx config and restart
```
ln -fs "/etc/nginx/sites-available/yourdomain.com" "/etc/nginx/sites-enabled/"
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl restart nginx
```

If you encountered an error:
```
sudo fuser -k 80/tcp
sudo fuser -k 443/tcp
sudo fuser -k 8443/tcp
sudo systemctl start nginx
```
And run the commands from the previous block again.

## Xray REALITY Inbound

## self-stealing

Port 443, VLESS protocol, then go down and set security to Reality, then click on clients to open, then set flow to **xtls-rprx-vision** for it, but be sure to set this for new users too when creating them in the small window that opens for creation there will be an option **(flow)**, do **not** select xtls-rprx-vision-udp443!
Do **not delete** short ids!
Do **not leave** spider x empty, you can put several spider xs in this field, the creator said to give different ones for each client but well I didn't see how we can set it to take a random one each time copying, we should tell Engineer Sanaei that just as he set it to give a different short id with each client, we should say for spider x too if he can put this. From that side they also say it doesn't affect improving the connection, so why it exists only rprx knows.

## self signed certificate
- Reality without domain (the goal it was originally designed for)
Install and set up Nginx yourself, don't run any of the mentioned scripts except the command to bring up the fake site, which you should do after manually installing and setting up Nginx.
Go into a directory like /root or /etc, run the following commands in order, openssl must be installed.
```
openssl ecparam -name prime256v1 -genkey -noout -out google.key
openssl req -x509 -nodes -days 365 -key google.key -out google.crt -subj "/CN=google.com"
openssl x509 -text -noout -in google.crt
```
The last command checks the status of cn certificates, these certificates are only accepted by your server, they have nothing to do with Google itself.
Now you need to configure Nginx.
```
vim /etc/nginx/sites-available/google.com
```
Copy the block below into it and save and exit, the path of my security certificates was in /root, change it for yourself.
```
server {
	server_tokens off;
	server_name google.com *.google.com;
	listen 80;
	listen 8443 ssl;
 	listen [::]:80;
        listen [::]:8443 ssl;
	index index.html index.htm index.php index.nginx-debian.html;
	root /var/www/html/;
	ssl_protocols TLSv1.3;
	ssl_ciphers HIGH:!aNULL:!eNULL:!MD5:!DES:!RC4:!ADH:!SSLv3:!EXP:!PSK:!DSS;
	ssl_certificate /root/google.crt;
	ssl_certificate_key /root/google.key;
	if ($host !~* ^(.+\.)?google.com$ ){return 444;}
	if ($scheme ~* https) {set $safe 1;}
	if ($ssl_server_name !~* ^(.+\.)?google.com$ ) {set $safe "${safe}0"; }
	if ($safe = 10){return 444;}
	if ($request_uri ~ "(\"|'|`|~|,|:|--|;|%|\$|&&|\?\?|0x00|0X00|\||\|\{|\}|\[|\]|<|>|\.\.\.|\.\.\/|\/\/\/)"){set $hack 1;}
	error_page 400 402 403 500 501 502 503 504 =404 /404;
	proxy_intercept_errors on;
}
```
Run these commands to recognize the new file
```
ln -fs "/etc/nginx/sites-available/google.com" "/etc/nginx/sites-enabled/"
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl restart nginx
```
In sni you should put www.google.com, and in dest, put 127.0.0.1:8443. Flow and short id and spid etc all like the previous inbound, self-steal.
Unlike the previous one, I didn't have heavy traffic on this method, I don't know how long it will last if you follow the conditions stated and have a really clean IP.
> [!CAUTION]
> This method is not recommended, I captured Client Hello and Server Hello packets in this state with Wireshark. OCSP stapling and Certificate chain and validity can be checked by the firewall here.

## Xray CDN Inbound

- **How to create CDN inbound from Chinese friend's page:**
Note that here use a domain whose CDN proxy tick is on. According to the tutorial video.
**Important** be sure to set utls to chrome, random which selects one of these but randomized creates a characteristic each time and in talking with Chinese friends they said it's better to be chrome for everyone.
- The tls-in-tls problem for CDN still remains, because xtls-rprx-vision doesn't exist for websocket. It depends on your domain how long it takes to get blocked. Emphasis is on using direct, if it's not possible at all use Iranian CDN.

In my text tutorial I meant for CDN you should first turn off the CDN proxy tick then run the gfw4fun script for it, after it's set go change the ports inside that domain's config like the Reality domain. Then go back and turn on the CDN tick, so the site comes up for them too. I don't know how Nginx responds to incoming requests for which a site hasn't been set up, most likely the site won't come up or at least won't be https, because it doesn't have that domain's security certificates.

For ufw

Re-enabling the server firewall, port 2789 is for my SSH change it to yours, also if possible don't leave the panel port open and when necessary update your firewall rules to allow the panel port to make changes or fix it then return to the previous state. The whole goal is to have few ports open on the server, and the server's response to invalid requests e.g. filtering system requests gives the reaction of a real site.
```
ufw status
ufw reset
ufw default deny incoming
ufw default allow outgoing
ufw allow 80,443,2789/tcp
ufw allow 80,443,2789/udp
```

Activation at the end
```
ufw enable
```

# Xray core's own reverse tunnel
All of this, or even that SSLH-EV tutorial can be run on an Iran server, then create a client in the Reality inbound, export the config to enter into the foreign panel's outbounds. Then for the core's own reverse first set up foreign, then come set up Iran. If a problem occurred or you don't want your foreign IP to connect to Iran, you can do all this on the Iran server but, run Cloudflare Argo Tunnel on this same Iran server, create a config with Argo, now this time enter this Argo config into the foreign panel's outbounds and act like before, so that this time it connects through Cloudflare and Iran sees Cloudflare IPs. The user config is still this Reality.

# Why return to Reality?
The core creator rprx and one of his friends fangliding said in a GitHub conversation that in their opinion this is not Reality detection and the problem is from our misconfiguration. Or that the firewall is closing all port forwarding and until it closes this method doesn't work.
https://github.com/XTLS/Xray-core/discussions/3269
In the same discussion rprx himself said I've made several new more powerful methods too and I'm keeping them for a rainy day for when they block Reality in China, until then I won't release them. fangliding also explained that in dest and sni when you put a site like Google or Zula, or any famous or non-famous site whose IP is not on your subnet, the firewall by giving a request that reaches dest through Xray and pinging sni realizes that your IP doesn't match the IP of that sni and detects you exactly like a port forwarder.
Iranian friends in this field still believe no, our filtering has reached the power to detect Reality. Anyway, Chinese friends have used sni shunting to solve this issue, its translation is taking sni out of the main path, when the firewall's request reaches the core and it gives it to Nginx, Nginx shows Google or Zula for example but the IP that reaches the request sender's hand is our server's IP. (This was what I guessed it would be, but it's different) So you can test shunting too, because anyway in SNI it will be Google or any other whitelisted site and well this is better, unless with a simple change the Iran firewall itself converts SNI of suspicious sites to IP and compares with the destination IP. When you're using Hetzner and SNI is Google it's obvious that Hetzner and Google have neither IP nor ASN related to each other, so maybe, maybe it works for non-famous sites, which then we come to this conclusion why not buy a domain and do it properly.
(I think the way to detect that our server has no connection to that used sni can't be done by it coming to ping and see if we're in that ASN or not, this is impossible, first of all Google Google LLC has [eight million](https://ipinfo.io/AS15169) IPs, how does it want to figure out which IP to check which connection to Google? Well let's assume it resolved IPs didn't match, Google has very many IPs. Does it want to close everything that didn't match? The solution to this problem for it may be to come manipulate the DNS inside the country for these domains, define a series of specific IPs, check outside this range, the next problem is that is Google just one site? It has very diverse services, all are Google IPs too)
https://github.com/chika0801/Xray-examples/tree/main/VLESS-Vision-REALITY/nginx_sni_shunting

After reading this discussion I became sure with the characteristics of the system ruling the country that there is nothing we have that China and Russia don't have, especially in the IT field. To validate a Reality config I distributed one in Mahsa Ng. After one terabyte from all operators across the country finally the IP was hit. (The domain is completely healthy and I brought up Reality on it again, it's passed 500GB so far. Some friends recently in groups were saying with Reality we passed up to 5 terabytes in 3 months before the IP was hit.

The setup was wrong, Reality port 8443 and site 443. Also the possibility of IP reporters' interference with the help of software like Wireshark but there was also for Android. Or even running Mahsa Ng on Bluestacks and using that Wireshark. Why am I saying this? The reason is that on that same server above (the server that I then implemented the site and Reality mode on one port on, not the Mahsa Ng config server that was filtered after one tera) I implemented the wrong setup, 150 gigabytes passed through it, all on Hamrah Aval, without the IP being closed.

Hamrah Aval users consumed 560 gigabytes, Irancell users 260 gigabytes.

**A point** I saw from the Kafka panel creator that's not bad to say here too: The goal of making a VPN based on tls is for our users' and our server's traffic to be completely similar to visiting a normal https site (all users connecting only to 443).

With the help of REALITY as inbound security, the characteristics of a tls connection are simulated flawlessly (tls1.3 x25519)

utls builds the characteristics of a real browser (chrome) (Note: The utls library is open and the filter can easily examine it and finally obtain the characteristics it needs from it to detect it (hasn't done or been able to yet)) (Fingerprint)

And xtls-rprx-vision solves the problem of detecting tls-in-tls. (Deep Packet Inspection DPI)

The fallback that has become destination in the Reality inbound shows the reaction of a real site to invalid requests. (ACTIVE PROBING)

# Translation of a few comments
RPRX [Comment](https://github.com/net4people/bbs/issues/327#issuecomment-1935672503) Main developer of Xray core

"The core of REALITY is to use the real TLS handshake of the target site to complete the handshake with the client, and then use the shared secret to encrypt the subsequent data. This process is completely indistinguishable from a real TLS connection to the target site.

The only difference is that after the handshake is completed, the subsequent data is not sent to the target site, but is decrypted by the server and processed.

Therefore, as long as the target site is not blocked, REALITY cannot be blocked. Unless the censor can decrypt TLS, which is impossible.

Of course, if the censor finds that there are too many connections to a certain IP that are disconnected after the TLS handshake is completed, they may become suspicious. But this is a general problem for all proxy protocols, not specific to REALITY.

In addition, REALITY's XTLS Vision flow control can make the traffic characteristics more similar to normal HTTPS, making it more difficult to detect.

In summary, REALITY is currently the most secure and undetectable proxy protocol. Of course, we will continue to improve and innovate to deal with possible future challenges."
