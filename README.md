# CSE 123 Project 2 - Router Implementation
---

Actual files are not public according to Academic Integrity. Inquiry for code: shzhangyihan@gmail.com

Yihan Zhang  

---
## Overview

> This simple router is designed based on a given static network topology and routing table on Mininet. It can process and forward raw Ethernet frames to its desired next hop. 

---
## Supporting Functionalities
>* Handle ARP requests and replies.  
>* Handle traceroutes through it and to it.  
>* Respond ICMP echo requests.  
>* Handle TCP/UDP packets sent to one of its interfaces. In this case the router responds with an ICMP port unreachable. 
>* Maintain an ARP cache whose entries are invalidated after a timeout period.  
>* Queue all packets waiting for outstanding ARP replies. If a host does not respond to 5 ARP requests, the queued packet is dropped and an ICMP host unreachable message is sent back to the source of the queued packet.
>### Extra Functionality: Designed for George Varghese Espresso prize (CSE 123)  
> * **IP fragmentation** functions when the frame size is over 66 bytes. If the frame is specified as DF(don't fragment), still forward the original sized frame.  
> * **Firewall** prevents the external access from Internet to private network (hardcoded as interface "eth3"). All connections must start from the hosts in private network, and expire if no package is going through the connection for 5 seconds.  
> * **NApT** *under construction, almost done but out of time :(*

----
## Usage
> IP fragmentation and Firewall are turned off by default.  

>* Enable **IP fragmentation**  

    ./sr -f
>* Enable **Firewall**  

    ./sr -F

>* Enable **NApT**  

    ./sr -N


----
## Design Aspects
>### **IP fragmentation size**  
> The MTU (Maximum Transmission Unit) is set to be 66 bytes, so as the maximum payload size of an IP packet can be a multiple of 8. Since the offset bit in IP header only record the offset as being divided by 8, having a maximum payload size other than some multiples of 8 would create blank spaces between fragments. Theoretically, MTU can be any size calculated as following:
14 bytes (length of Ethernet header) + 20 bytes (length of IP header) + k * 8 bytes (payload size, while k can be any integer > 0). In this case, MTU is selected as 66 bytes to exhibit fragmentations when doing traceroutes. MTU is defined in line 32 of file ./sr_router.h:

    #define MAX_PACKET_SIZE 66

>### **Don't fragment packets**  
> Theoretically, the DF packets with a size larger than MTU should be dropped. However, considering the tininess of this testing MTU, all DF packets would be forwarded. 

>### **Private network definition**  
> Since this design depends on a given static network topology, the private network is hardcoded as interface "eth3", with IP address 10.0.1.1. If this router would be designed as a general purpose one, more functions to load in and initiate the private network setting should be added. 

>### **Firewall cache data structure**  
> The Firewall caches the source IP and destination IP for all outgoing frames from the private network to Internet through the following data structure:

    struct sr_fwentry {
        uint32_t ip_from;
        uint32_t ip_to;
        time_t sent; 
        struct sr_fwentry *next;
    };
>### **Firewall expiration time**  
> The hosts outside of the private network can only send packets to the private hosts when the IP address pair can be found in the cache. Each time a frame is forwarded through this IP pair, the timer would be updated. If no packet is transmitting between this IP pair for 5 seconds, the router would assume the connection is over and destroy the cache entry. Expiration time is hardcoded in 
line 199 of file ./sr_router.c:

    if(difftime(now, fwentry->sent) > 5.0) {

----
## Special notes
> * When Firewall is turned on, all connection attempts (ping or traceroute) from server side to the client side would be dropped, unless the client have send packets to the server within the recent 5 seconds. 
> * IP Fragmentation can not be used along with NApT, since router is not able to reassembly fragments in order to read port number.



----
## changelog
* 17-Mar-2017

----
## thanks
* Prof. Alex C. Snoeren
* [mininet](http://mininet.org)
