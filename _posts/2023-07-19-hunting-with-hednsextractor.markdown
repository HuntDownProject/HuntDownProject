---
layout: post
title:  "Hunting with HEDnsExtractor"
date:   2023-07-19 14:47:00 -0300
categories: HEDnsExtractor
---
HEDnsExtractor is a python script used to extract domains from the website [https://bgp.he.net/](https://bgp.he.net/).

Imagine that you want to extract all registered domains using the network range `104.21.0.0/19`, manually you entered the website and search for, but you want more, as threat hunter you need some more metadata, so to keep the things simple you can combine the HEDnsExtractor with another tools like [httpx](https://github.com/projectdiscovery/httpx) and [nuclei](https://github.com/projectdiscovery/nuclei).

# Installation

## First Step

Install HEDnsExtractor cloning the project from [github repository](https://github.com/teixeira0xfffff/HEDnsExtractor).

## Second Step

After that install the dependencies:

```shell
$ pip install selenium beautifulsoup4
```

## Third Step

The chromium web driver also is needed, go to the [website project](https://chromedriver.chromium.org/downloads) and download the version for your operating system, install it in a directory visible to your $PATH environment variable, then the HEDnsExtractor could use the chromedriver.

# Usage

The command line below will bring all domains and print in the stdout:

```bash
python hednsextractor.py "https://bgp.he.net/net/104.21.0.0/19#_dns"
```

Combine the output with the [httpx](https://github.com/projectdiscovery/httpx) to get the website title and to detect the tecnologies used:

```bash
python hednsextractor.py "https://bgp.he.net/net/104.21.0.0/19#_dns" | httpx -title -tech-detect -status-code
```