---
title: "APT ipv6 connection issue: over ipv6 network"
date: 2022-05-16T15:12:24+05:30
draft: false
tags: ["apt", "ubuntu" , "ipv6" ]
author: "Albony"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "APT trying to use IPv6 to connect to a mirror even though the network only supports IPv4"
canonicalURL: "https://blog.albony.xyz/posts/apt-ipv6"
disableHLJS: false # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "/apt_ipv6.png" 
    caption: "APT trying to connect over IPv6" 
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/blog.albony.xyz/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link

---
## The Problem 
#### Today is patch monday, I update my docker containers and packages today :D but when I did `sudo apt update && sudo apt upgrade -y` I noticed that it was trying to connect to the ubuntu mirror over IPv6 (and failing)  
#### My ISP hasn't enabled IPv6 for me, so I can't connect over IPv6. and I don't have any IPv6 DNS servers. But still `apt` was trying to connect over IPv6... weird... 
![APT trying to connect over IPv6](/apt_ipv6.png)

## The Fix
#### You can create a file in the `/etc/apt/apt.conf.d` directory to force `apt` to use IPv4  
``` sh
 echo 'Acquire::ForceIPv4 "true";' | sudo tee /etc/apt/apt.conf.d/99force-ipv4

```
I tried updating after this and can confirm that it works :D 



![apt fix](/apt_ipv4.png)

