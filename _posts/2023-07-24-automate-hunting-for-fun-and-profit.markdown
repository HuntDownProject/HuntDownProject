---
layout: post
title:  "Automate hunting for fun and profit"
date:   2023-07-24 17:52:00 -0300
categories: threathunting
---

Automate your threat hunting and get more time for a coffee while hunting malicious website.

The goal here is to automate the discover of new malicious website from an already know domain, eg: sometimes a threat actor removes their phishing site, but uses the same provider, then a new domain arrives but using the same infrastructure. Thinking about this hypothesis that this POC was written.

# Get the arsenal

To get this job we need some tools that are:

| toolkit | description |
| ------- | ----------- |
| [anew](https://github.com/tomnomnom/anew) | this will be usefull to get new data and verify if we already analyzed.
| [dnsx](https://github.com/projectdiscovery/dnsx) | with this tool all suspected domains can have their dns information explored.
| [httpx](https://github.com/projectdiscovery/httpx) | a powerfull httpx toolkit.
| [katana](https://github.com/projectdiscovery/katana) | a powerfull crawler toolkit.
| [notify](https://github.com/projectdiscovery/notify) | a "super util" toolkit to manage notifications.
| [proxify](https://github.com/projectdiscovery/proxify) | a superb proxy. 

Install all the tools following script bellow:

```bash
go install -v github.com/tomnomnom/anew@latest
go install -v github.com/projectdiscovery/dnsx/cmd/dnsx@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/projectdiscovery/katana/cmd/katana@latest
go install -v github.com/projectdiscovery/notify/cmd/notify@latest
go install -v github.com/projectdiscovery/proxify/cmd/proxify@latest
```

# Scripting

The first thing here is running a discovery using a known bad domain, and here imagine the following domains:

```bash
cat discovery.txt
resgateitaushop.info
componentecliente.com
```

What we need is their IP Address and here the `dnsx` help us:

```bash
cat discovery.txt | anew discovery_new.txt | notify | dnsx -resp-only -silent -retry 3
172.61.144.2
104.21.8.223
14.21.55.18
172.64.158.18
```

The `anew` will get only new domains and put in the file `discovery_new.txt`, if the domain is already know (already in discovery_new.txt), `anew` don't send nothing to `notify`, if is a new domain then this goes to `notify` following to `dnsx` that resolves each domain.

The next step is to discover the network range that each IP address belongs, and to get exactly the website [https://bgp.he.net](https://bgp.he.net) is used. To crawler this, `katana` comes to help, but first needs to be configured.

Edit their config file for field config `($HOME/.config/katana/field-config.yaml)` and append the config bellow:


```yaml
- name: getdomains
  type: regex
  part: body
  regex:
  - <a href=\"\/dns\/([^"]+)\"
  group: 1

- name: getroute
  type: regex
  part: body
  regex:
  - <a href=\"\/net\/[^"]+\">(.*)</a>
  group: 1
```

Running `katana` as in the example, the `getroute` field configured run the regex `<a href=\"\/net\/[^"]+\">(.*)</a>` and extract all network ranged finded in the web page.

```shell
katana -u https://bgp.he.net/ip/172.61.144.2 -f getroute -ct 20 -rl 1 -silent | anew network_ranges.txt
```

Again the `anew` is used to include only new data, now the file `network_ranges.txt` and created/populated.

And finally to get the domains registered for each in the network range, the katana come to get things done.

```shell
 katana -u https://bgp.he.net/net/172.67.144.0/20 -f getdomains -ct 20 -rl 1 -silent | anew domains_temp.txt | httpx -silent -title -td | notify >> final_report.txt
```

Now `katana` is using the regex/field config `getdomains` that extract the domains from the webpage, and it's not alone, again `anew` to populate domains_temp.txt to work only in new domains, that are send to `httpx` that get the title from HTML page and also the technologies used in the website.

All the examples above, also save the work in .txt files that can be used before and also uses the `notify` that are used to send notifications to discord (in my case), I will not cover here the notify configurations because it`s to easy, just follow the examples in oficial github repository.

Here for katana I also used the parameters `-ct` to set up the crawler duration as the `-rl` for the rate limit to not overload the website services.

# Putting it all together

I said automation right ? so let`s putting it all together to have fun:

```shell
#!/bin/sh

crawler_timeout=20

echo "Running Discovery for suspected domains" | notify
network_range=`cat discovery.txt | anew discovery_new.txt | notify | dnsx -resp-only -silent -retry 3`
for ip in $network_range;
do
    echo "Discovering range for IPv4 $ip"
    katana -u https://bgp.he.net/ip/$ip -f getroute -ct $crawler_timeout -rl 1 -silent | anew network_ranges.txt
done

ranges=`cat network_ranges.txt | notify`
for range in $ranges;
do
    echo "Discovering domains for range $range" | notify
    katana -u https://bgp.he.net/net/$range/ -f getdomains -ct $crawler_timeout -rl 1 -silent | anew domains_temp.txt | httpx -silent -title -td | notify >> final_report.txt
done
```

# Are you still there ?

Brave, I expected that you are questioning about the `proxily` mentioned in the arsenal list. Maybe in some cases you need a proxy services and here I combined `proxily` with tor service that helps a lot, you just need to run `proxily` as in the documentation

```shell
proxify -socks5-proxy 127.0.0.1:9050
```

Don't forget the tor, grab your prefered from docker image for it, and the final script altered to use the proxy:

```shell
#!/bin/sh

crawler_timeout=20
proxy="http://localhost:8888"

echo "Running Discovery for suspected domains" | notify
network_range=`cat discovery.txt | anew discovery_new.txt | notify | dnsx -resp-only -silent -retry 3`
for ip in $network_range;
do
    echo "Discovering range for IPv4 $ip"
    katana -u https://bgp.he.net/ip/$ip -f getroute -ct $crawler_timeout -rl 1 -proxy $proxy -silent | anew network_ranges.txt
done

ranges=`cat network_ranges.txt | notify`
for range in $ranges;
do
    echo "Discovering domains for range $range" | notify
    katana -u https://bgp.he.net/net/$range/ -f getdomains -ct $crawler_timeout -rl 1 -proxy $proxy -silent | anew domains_temp.txt | httpx -silent -title -td -http-proxy $proxy | notify >> final_report.txt
done
```