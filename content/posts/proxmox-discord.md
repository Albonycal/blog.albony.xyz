---
title: "Dark Mode for Proxmox"
date: 2021-12-10T15:12:24+05:30
draft: false
tags: ["proxmox"]
author: "Albony"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "PVE Discord Theme"
canonicalURL: "https://blog.albony.xyz/posts/proxmox-discord"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "/proxmox-discord.png" 
    caption: "Proxmox" 
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/blog.albony.xyz/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link

---



#### How to make proxmox UI beautiful 
 I use proxmox daily for virtual machines and LCX containers,
 It is one of the best virtual machince platform that utilizes KVM
*And it's FOSS*
![Proxmox](/proxmox.png)
but the default UI is too bright and I prefer darker themes
#### Discord-PVE
Discord-PVE is a discord theme for the proxmox virtual environment 
it uses custom stylesheet.

To Install it: run this oneliner 
```sh
bash <(curl -s https://raw.githubusercontent.com/Weilbyte/PVEDiscordDark/master/PVEDiscordDark.sh ) install
``` 
This looks soo much better
 https://github.com/Weilbyte/PVEDiscordDark
![Proxmox](/proxmox-discord.png)

