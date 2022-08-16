---
title: "New Termux and Armbian mirrors live now! "
date: 2022-08-16T07:19:31Z
tags: ["termux" , "armbian" , "mirror"]
author: "Albony"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: " "
canonicalURL: "https://blog.albony.xyz/posts/termux-armbian"
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "/mirror.png"
    caption: " "
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/blog.albony.xyz/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link

---
## Termux

#### mirror.albony.xyz/termux
Its added to the termux mirrorlist so you should get it automatically soon, if you want to add it manually: 

`termux-change-repo` (part of termux-tools package) can be used to modify sources. 
Another way of doing it is by using `apt edit-sources` and adding the following lines:

``` 
# main
deb https://mirror.albony.xyz/termux/termux-main stable main

# root
deb https://mirror.albony.xyz/termux/termux-root root stable

# X11
deb https://mirror.albony.xyz/termux/termux-x11 x11 main

```

## Armbian

#### mirror.albony.xyz/armbian

This is also added to the armbian mirrorlist but you can also add it manually by editing the sources. 

The mirrors are hosted at Nagpur, India and sync aprox. every 53 minutes from the upstream.


