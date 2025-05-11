---
title: "Security And Encryption In Authoritative DNS"
date: "2025-05-11T09:43:19Z"
tags: ["DNS" , "Encryption" , "QUIC" ]
author: "Shrirang Kahale"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: " "
canonicalURL: "https://blog.albony.xyz/posts/encryption-in-authoritative-dns"
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "/"
    caption: " "
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/blog.albony.xyz/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link

---


##### Internet as we know today, emerged from ARPANET, which was a research network made by the US DoD. 

Technologies like TCP/IP formed the basis of ARPANET, being the the first network to use IP based communication. 

ARPANET was eventually dissolved, but new networks were formed and with growing ideas and technology, The Internet was born.

People to this date are reminiscent about the days of the [dot-com bubble](https://en.wikipedia.org/wiki/Dot-com_bubble) which was a period of rapid internet growth. 


As the network grew, So did the number of servers and IP addresses, and the concept of using easy to remember domain names was called for. 

Before DNS a giant [text file](https://rscott.org/OldInternetFiles/) "HOSTS.TXT" was maintained which had the records. But quickly it was realised that this is not feasible. That's when DNS was born. A giant, hierarchial and scalable database. 
(RFC 882 and RFC 883 were standardised by IETF in 1983)

Those were the days of plain text communication and encryption wasn't considered, when DNS was standardised, it was merely a system which could scale. 

While mechanisms like DoT and DoH exist for encrypted DNS between recursive and stub-resolvers (clients) and are very popular in today's date, progress on the authoritative side has been slow.

## ADoT (Authoritative DNS over TLS)



A few days ago I unearthed evidence of Google's Public DNS (8.8.8.8) probing authoritative nameservers over DoT (port 853) by looking at traffic to my authoritative servers.

![Image showing output of dig tool](/image_2025-05-11T09-39-18Z.png)

I found that Google lists the IP ranges used for authoritative DNS queries and sure enough, the IPs I saw on my end matched with their Mumbai range used for Google DNS. 

After a bit more digging, I came across this [blogpost](https://security.googleblog.com/2024/03/) which mentions Google DNS adding support for ADoT (Authoritative DNS over TLS) 

>  We’ve deployed DNS-over-TLS to authoritative nameservers (ADoT), following procedures described in RFC 9539 (Unilateral Opportunistic Deployment of Encrypted Recursive-to-Authoritative DNS). 
 Real world measurements show that ADoT has a higher success rate and comparable latency to UDP. And ADoT is in use for around 6% of egress traffic. At the cost of some CPU and memory, we get both security and privacy for nameserver queries without DNS compliance issues.

PowerDNS (which I am using) currently **does not support ADoT**. Although experimental support has been added in BIND and some other implementations, interestingly PowerDNS Recursor does support probing for DoT as-per the above-mentioned RFC.

As of now, Google Public DNS remains the only one to support ADoT. According to Google stats ADoT support is less than 0.1% of their total query volume. 
Which makes me question the economics of this.

Others have good reasons to not implement it, ADoT can be extremely resource intensive, not only due to the added CPU cycles for encryption but also due to the active probing mechanism outlined in RFC 9539, which requires them to active probe to see if authoritative nameservers support ADoT. Google due to their large scale and bigger pockets can justify this, while others might not be able to. 

It is interesting to note that [Root-B server has also added experimental support for DoT. ](https://b.root-servers.org/news/2023/02/28/tls.html)


## DoQ (DNS over QUIC)

DoQ is a relatively new technology compared to the others - it has minimal support as it only became a Proposed Standard in May 2022. 

One of its key features is mandatory encryption - the QUIC initial handshake can only be performed with TLS encryption (RFC9000, section 7.1). 

Thanks to its optimizations - most importantly, built-in support for 0-RTT handshakes, it is a far more efficient transport for DNS compared to DoT or DoH.


QUIC is a lot **more** than just something that is "like TCP" but over UDP. Unlike UDP, QUIC has support for congestion control algorithms like `BBR` and `CUBIC`, QUIC is also multipath friendly. And should offer better delivery than plain old UDP. 

Overall, I am still sceptical about the technology, it will do fine for DNS but when personally testing HTTP/3 for [mirror.albony.in](https://mirror.albony.in/) which is a popular free software mirror in India, I found it consistently slower than HTTP/2 (TCP). TCP has gone through decades of optimisations, a lot of work has been put into making it better and it shows. I also found this research [paper](https://arxiv.org/pdf/2310.09423) pointing out the very same thing.

I do think QUIC is the modern counterpart and *is* the future, with all the investments put into it. It is also censorship-proof as it encrypts the SNI (Server Name Indicated) by default. 

For HTTP/2 TLS ECH is proposed to solve the problem of exposed plain-text SNI and it's already in use quite a bit.



Anyway, we are getting off-topic! Back to DNS. 
<hr> </hr>

The problem of probing exists in ADoQ as well. After-all these are proposed standards and support is very rare. 

I could not find any evidence of ADoQ being implemented.

#### I would also like to emphasise that this probing mechanism is opportunistic at-best. Which means that it'll fall back to unencrypted DNS if encrypted communication fails.  (For both DoQ and DoT)

For those who are familiar with it, this mechanism is very similar to how the [Happy eyeballs](https://en.wikipedia.org/wiki/Happy_Eyeballs) algorithm operates to check for IPv6 connectivity. 

The "unilateral" in  RFC 9539 means that authoritative server does not need to do anything to tell the recursive servers that they support DoT/DoQ, this thus requires the active probing mechanism on the recursor side as discussed above.  
And this probing, similar to Happy eyeballs must happen from time to time, because network conditions change. 
Overall this adds massive operational expenditure on recursive side. 

There's no public data on this part yet, but I have tried contacting a few organisations and still waiting for their reply. 

## XoT (XFR over TLS)

As we know, it is recommended for redundancy reasons to have multiple nameservers hosting your zone. 
Now, the question arises on how to keep them in sync? Changes to the zone file should also propagate quickly to all nameservers in-order to maintain consistency and avoid problems. 

Enter, XFR. 

RFC 5936 (AXFR) and RFC 1995 (IXFR) defines the zone transfer mechanisms currently used to achieve this purpose, which is done using clear-text communication using DNS. 
There's also a `NOTIFY` mechanism to notify all servers to retrieve latest copy of zone file. If they are authorized to do so, the authoritative servers can also manually fetch the zonefile using the `RETRIEVE` mechanism. 

RFC 9103 (XoT) is a proposed standard to make this zone transfer encrypted. Although one does not need XoT for this communication to be encrypted if one is willing to implement a custom solution, because in this case you can control the servers end-to-end.

This means that you can use encrypted wireguard or any-other tunnels to do XFR, many people do this. 

Another method is to use database replication, after-all the zone files in the end are stored in databases. This would avoid XFR. 


## A-DNS Security

RFC 3833 [Threat Analysis of the DNS] describes threats to serving correct DNS data to clients. 

DNS Response spoofing is by-far the most popular one. Some of the counter-measures for this are mentioned in RFC 5452.

ADoT/ADoQ is not one of them, but it is the most effective one. 

### DNS Cookies

The current work-arounds include using "DNS cookies" which is an Extended DNS (EDNS) option. 

Currently defined by RFC 9018 DNS cookies are "a lightweight DNS transaction security mechanism that provide limited protection to DNS servers and clients against a variety of denial-of-service amplification, forgery, or cache-poisoning attacks by off-path attackers. 


#### In short words, they are random strings included in the response from the authoritative server pre-exchanged during intial query. This cookie is then to be included in every subsequent communication.

When supported by both recursor and authoritative server, it allows recursor to detect spoofed responses and server to determine that a client's address is not spoofed.

It is to be noted that this method is **not full-proof** because the server needs to exchange cookie first and the recursor has to retain it for a fixed amount of time to verify subsiequent requests.
But it is still a good counter-measure if deployed widely.

BIND9 and PowerDNS both support DNS Cookies.

<hr> </hr>

## Post Quantum Cryptography (PQC)

As encryption is mentioned in my blog post, I felt it was essential to also include post quantum cryptography. Quantum Computers in future maybe fast enough to break the AES encryption that's holding our world. Post quantum cryptographic algorithms need to be implemented to circumvent this threat. These algorithms are preparing for Y2Q, the year when Quantum computers will have enough processing power to crack modern AES. 

Popular messaging protocol Signal has already implemented post-quantum cryptography with their [X3DH](https://signal.org/docs/specifications/x3dh/) protocol. Fortunately for internet protocols other than changing the algorithms used, no major change is required in-order to accommodate for PQC.

The field itself is very complicated hence I won't comment anymore. 


<hr> </hr>

Thank you for reading! 


If you have any questions, please don't hesitate to leave a comment or contact me.

Shrirang




