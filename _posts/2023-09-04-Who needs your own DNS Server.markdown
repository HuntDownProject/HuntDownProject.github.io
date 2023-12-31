---
layout: post
title: "Who needs your own DNS Server ?"
date: 2023-09-01 2:27:00 -0300
categories: hunting
---

I believe readers of this site know what a DNS server is, but did you heard about the project [Pi-hole](https://pi-hole.net/) ? This is a very nice project, because settting up your own DNS server with Pi-hole you will have:

* Network-wide protection
* Block in-app advertisements
* Improve network performance
* Monitor statistics

I will not extend here about the project, because what we want is hunting the things (Can you scream hack the planet ?)

# Hunting

Lets make use of [shodan.io](https://www.shodan.io/) to search for some exposed Pi-hole hosts, it's simple you can see bellow:

```bash
[brlaw@huntdown.local:~$]  shodan search pi-hole --separator , | cut -d, -f 1 | wc -l
101
[brlaw@huntdown.local:~$]
```

The command output return `101` hosts (*to get the full output just run `shodan search pi-hole`*), but will it be the same ? To check each IP we wrote a [nuclei](https://github.com/projectdiscovery/nuclei) template, that right now while I am writing is on [queue](https://github.com/projectdiscovery/nuclei-templates/pull/8149) waiting to be merged.

The screenshot bellow shows the nuclei output:

![Shodan Search Pi-hole]({{ site.url }}/assets/img/shodan_search_pihole.png)

While the template don't get into the official distribution you can check out from your [repo](https://github.com/neriberto/nuclei-templates/blob/feature/pihole/http/exposed-panels/pihole-login.yaml).

```yaml
id: pihole

info:
  name: PI-hole login panel - Detect
  author: neriberto
  severity: info
  description: PI-hole login panel was detected.
  reference:
    - https://pi-hole.net/
  classification:
    cvss-metrics: CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:N
    cvss-score: 0.0
    cwe-id: CWE-200
  tags: panel,login
  metadata:
    max-request: 1

http:
  - method: GET
    path:
      - '{{BaseURL}}/admin/login.php'
      - '{{BaseURL}}/admin/index.php?login'

    matchers-condition: or
    matchers:
      - type: word
        words:
          - '<title>Pi-hole - '
          - 'Pi-hole: Your black hole for Internet advertisements'
          - 'Pi-hole: A black hole for Internet advertisements'
          - 'https://pi-hole.net'
          - '<pre>sudo pihole -a -p</pre>'
        condition: or
```
