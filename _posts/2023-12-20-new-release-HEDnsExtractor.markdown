---
layout: post
title:  "HEDnsExtractor, a suite for hunting suspicious targets, expose domains and phishing discovery"
date:   2023-12-20 14:47:00 -0300
categories: hunting
---
<br>
We announce the new version of HEDnsExtractor with new features and automation mechanisms for discovering new phishings using yaml recips and finally an introduction to the use of regex directly in HEDnsExtractor:

* Implementing workflows with yaml 🔥
* Adds support to work with multiples domains as target 🔥
* Regex support 🥷
* Adds support to work with IPv6 filters 🔥
* Change to Golang 1.21 🆙
  
# Workflows

With this new feature it is possible to map multiple domains and use regex to identify domains based on their regex, see example below:

$ HEDnsExtractor -workflow WellsFargo_Detection.yaml

```yaml
domains:
  - 104.237.252.65
  - cancelfrgoref3eb0d.com
  
regex: (well|frgo|fargo)
```
<br>

![image](https://github.com/HuntDownProject/HuntDownProject.github.io/blob/main/assets/img/form_output_2.jpeg?raw=true)

> **ProTip:** You can insert the list of authorized targets into yaml and run the query periodically to see if new domains have been identified.

# Regex

In the yaml recipe you can perform your filters without having to use another tool to perform the action:

**Example 1:** regex: (well|frgo|fargo)<br>
**Example 2:** regex: (*gov\d+)<br>
**Example 3:** regex: (cancel\d{3})

![image](https://github.com/HuntDownProject/HuntDownProject.github.io/blob/main/assets/img/form_output_3.jpeg?raw=true)

# IPv6 filters

We have added the possibility of querying via IPv6:

$ hednsextractor -target 2001:db8:85a3::8a2e:370:7334
