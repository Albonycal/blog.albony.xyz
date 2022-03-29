---
title: "Setting DNS for containers in a docker environment"
date: 2021-12-10T15:12:24+05:30
draft: false
tags: ["docker" "dns" ]
author: "Albony"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "specifying DNS for docker containers"
canonicalURL: "https://blog.albony.xyz/posts/DNS-DOCKER"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "/docker.png" 
    caption: "Docker" 
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/blog.albony.xyz/"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link

---
## Setting DNS for all containers in a docker environment 
#### I have noticed that all my containers default to `8.8.8.8` google dns because they are unable to connect to the DNS servers configured for my network.
#### I have been using a dirty hack to set my own DNS servers. I knew this wasn't the correct way to do it. i.e making a `resolv.conf` in the docker directory and mounting it at `/etc/resolv.conf` in the container :D
#### It works but isn't the correct and proper way to do it. 
##  Configuring `/etc/docker/daemon.json`
#### You can set the DNS for the docker environment in the `/etc/docker/daemon.json` file.  
``` sh
 sudoedit /etc/docker/daemon.json

```
If asked chose a text editor of choice. 
and add this to the file. 
```json
{
    "dns": ["<DNS_IP>", "<DNS2_IP>"]
}
```
Mine looks like this, 
![[/daemon.json.png]]

#### Now restart the docker daemon: 
```sh
sudo systemctl restart docker
```

#### The containers should use the provided DNS servers now.
