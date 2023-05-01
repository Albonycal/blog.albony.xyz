---
title: "Another Case Of Bad Routing: Vi and Airtel"
date: "2023-05-01T09:53:15Z"
tags: ["routing" , "internet" , "vowifi"]
author: "Shrirang Kahale"
# author: ["Me", "You"] # multiple authors
showToc: false
TocOpen: false
hidemeta: false
comments: true
description: "Abismal routing between Airtel and Vi"
canonicalURL: "https://blog.albony.xyz/posts/airtel-vi-bad-routing"
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "/vi-bgptools.png"
    caption: " "
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/blog.albony.xyz/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link

---


## Even though Bharti Airtel (AS9498) is an upstream of Vodafone Idea (Vi) the routing between the two ISPs is consistently bad. While this heavily affects P2P traffic, it also has other impacts. 

![](/vi-bgptools.png)

### ➜  VoWiFi (Voice Over WiFi) is very useful for places which have poor cellular coverage, But due to the horrible routing between the two ISPs the VoWiFi experience is bad. When I use VoWiFi (Vi SIM) on Airtel Broadband connection, there are constant call drops and stuttering. 

### ➜ 3GPP (Third Generation Partnership Project) has defined a number of network domains that are used to describe different parts of the cellular network architecture.

#### The domains used for VoWiFi look like this: `epdg.epc.mnc022.mcc404.pub.3gppnetwork.org`



#### MCC stands for Mobile Country Code, and it is a three-digit code used to identify the country where a mobile network operator is based. The MCC is assigned by the International Telecommunication Union (ITU) and is used in combination with the Mobile Network Code (MNC) to uniquely identify a mobile network operator.

### Now let's take a look at this traceroute: 
```c
fawks on phoenix ~ took 1s 
➜ mtr -wr epdg.epc.mnc022.mcc404.pub.3gppnetwork.org 

Start: 2023-05-01T14:42:16+0530
HOST: phoenix                Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 2401:4900:1c02:526c::   0.0%    10    0.4   0.4   0.4   0.5   0.0
  2.|-- 2401:4900:1c02:8fff::1  0.0%    10   13.8  20.1  13.6  48.0  11.3
  3.|-- 2404:a800:2a00::2a5     0.0%    10   14.3  15.0  14.0  18.3   1.4
  4.|-- 2404:a800::204         50.0%    10   44.6  44.9  44.1  45.8   0.7
  5.|-- 2404:a800:1a00:803::7a  0.0%    10  103.6 103.1 102.2 105.9   1.1
  6.|-- 2402:6800:760:7::72     0.0%    10   87.5  88.8  87.5  92.5   1.5
  7.|-- 2400:5200:1400:72::1   20.0%    10  104.9 103.2 102.3 104.9   1.1
  8.|-- ???                    100.0    10    0.0   0.0   0.0   0.0   0.0
  9.|-- 2402:3a80:1060::3       0.0%    10  103.8 105.0 103.7 108.1   1.7
```

### Look at the 6th hop, `2402:6800:760:7::72` belongs to AS55429 Limelight Networks India, which is one of the downstreams of Bharti Airtel (AS9498) why is it routing through that? 

![](/limenetworks.png)

### If are a user using Vi internet, the routing is even worse. 

```c
➜ mtr -wr 2402:8100:3149:9ee8:2859:60ff:fedf:1ac6      
Start: 2023-05-01T15:08:48+0530
HOST: phoenix                                              Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 2401:4900:1c02:526c::                                 0.0%    10    0.6   0.5   0.3   0.6   0.1
  2.|-- 2401:4900:1c02:8fff::1                                0.0%    10   14.8  19.1  13.4  35.1   8.4
  3.|-- 2404:a800:2a00::2a1                                   0.0%    10   33.8  24.8  14.1  43.0   9.7
  4.|-- 2404:a800::147                                        0.0%    10  125.4 126.0 123.3 133.8   3.8
  5.|-- ???                                                  100.0    10    0.0   0.0   0.0   0.0   0.0
  6.|-- port-channel12.core2.fra1.he.net                      0.0%    10  138.5 138.7 138.0 140.0   0.7
  7.|-- level3-as3356.10gigabitethernet3-7.core1.fra1.he.net  0.0%    10  139.8 140.5 139.8 141.8   0.7
  8.|-- lo-0-v6.ear2.London1.Level3.net                       0.0%    10  152.9 152.8 152.1 153.9   0.6
  9.|-- 2001:1900:5:2:2::7e3e                                20.0%    10  241.3 241.8 241.2 242.9   0.6
 10.|-- 2400:c700:0:370::f2                                  10.0%    10  231.3 231.6 231.0 232.6   0.5
 11.|-- 2402:8100:4000::7bb                                   0.0%    10  230.6 231.2 230.6 232.7   0.8
 12.|-- 2402:8100:4000::613                                   0.0%    10  252.7 250.0 248.8 252.7   1.1
 13.|-- 2402:8100:4000::7e4                                   0.0%    10  230.9 230.8 230.0 231.6   0.6
 14.|-- 2402:8100:4000::5fd                                   0.0%    10  247.1 247.9 247.0 250.5   1.1
 15.|-- ???                                                  100.0    10    0.0   0.0   0.0   0.0   0.0
 16.|-- 2402:8100:4000::7a8                                   0.0%    10  229.6 229.3 228.7 230.5   0.8
 17.|-- fd00:0:17:17::72                                      0.0%    10  247.6 247.1 246.5 248.2   0.5
 18.|-- ???                                                  100.0    10    0.0   0.0   0.0   0.0   0.0

```

### It goes from: 

### India (Airtel) ➜  Germany (HE.net) ➜  UK (Level3) ➜  India (Vi) with RTT of more than 200ms

#### I tried connecting from my Vi LTE through a VPN to my server at home, it's literally unusable: very high latency along with packetloss. They need to do something about this.
