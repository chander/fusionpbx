# fusionpbx
Setup for FusionPBX

## General Setup Instructions for FusionPBX/FreeSwitch with FlowRoute

My setup involves FreeSwitch sitting on a private (10.0.1.0/24) network behind a NAT firewall (pfsense), it took me some time
and effort to get things setup properly, so I put together this document in case I have to do it in the future.

## Setup of Firewall/PFSense

1. Connect to the PFSense Firewall, and process to System->Advanced->Firewall & NAT
  1. Change the "Firewall Optimization Options" to "Conservative"
    * In theory, this might not be required, since FreeSwitch will send periodic INVITEs to ensure NAT connections are kept alive; but I did it anyways.
  1.  Go to Firewall -> Aliases and add am alias, called "Flowroute" with the following IP addresses/networks:
  
    * 216.115.69.144/32, 147.75.65.192/28, 34.226.36.32/28, 34.210.91.112/28, 147.75.60.160/28
    * Check with Flowroute for their latest IP address list.
  1.  Go to Firewall -> NAT and add a new TCP/UDP rule that allows inbound packets from alias "Flowroute" on the WAN to the
      IP of the FusionPBX host (note that the host will need a static IP Address.)
      
      * Interface: WAN
      * Protocol: TCP/UDP
      * Source: Single host or alias -> Flowroute
      * Source Port Range: Any
      * Destination: Any
      * Destination Port Range: SIP
      * Redirect Target IP: Fusionpbx address
      * Redirect Target Port: SIP
      * Description: Flowroute SIP
      * Ensure the rule (when created) also creates the Firewall Pass-through rule on the WAN.
      
  1.  Add another NAT rule, this one should allow UDP traffic from and port on any source to a destination of "any" with a destination port range of 16384-32768 to route to your Redirect target IP of your FusionPBX server, with a Redirect target port of 16384 (call the rule "Flowroute Direct Audio")
  
      * Interface: WAN
      * Protocol: UDP
      * Source: Any
      * Source Port Range: Any
      * Destination: Any
      * Destination Port Range: 16384-32768
      * Redirect Target IP: Fusionpbx address
      * Redirect Target Port: 16384
      * Description: Flowroute Direct Audio
  
  1.  Go to Firewall -> NAT -> Outbound and switch the mode to "Hybrid"
  1.  On the same page, add a new Mapping with an interface of WAN, a Source of your FusionPBX IP Address and a port of 5060 for the 
      source and the destination.
      
## Setup of FreeSwitch

Freeswitch will need to be configured to have "Allow" records for the domain you are using - thereby allowing connections from Flowroute directly without authentication:
  1.  Go to Advanced -> Access Controls then click on "domains"
    1.  Press the "+" button to add a new rule, and proceed to enter in all the CIDR network data (as allow rules) as you added for the PFSense "Flowroute" alias.
    1.  Alternately, you might be able to set the default for the domain to "allow" (since only the hosts you specified can really access Flowroute through the firewall.)
  1.  Go to Advanced -> Variables 
  
      * Find "externa_sip_ip" and "internal_sip_ip" and change them to auto-nat
      * I'm not certain this needs to be done for me, since I don't have any external sip clients, just a gateway, but in theory this
        should make flowroute realize that you are behind nat and do natty stuff.
 
## The rest
 
Now that the network stuff is setup, you can go to Accounts->Gateways and add in a gateway to flowroute; check the interconnection screen on the flowroute site - you should see a single registration (always); if you see multiple, then something might be wrong with your nat setup, causing the nat connection to timeout an have to reconnect.  That's bad.
 
If inbound calls get routed to your Flowroute configured "failover" number, it's likely that you might have a domain issue - where the access controls are not setup right.

The best way to troubleshoot these issues is to log into your fusionpbx server, start a fs_cli session and type in "sofia loglevel all 9" which will give you a stream of log output that's very verbose.


### Legal stuff

Copyright (c) 2019 Chander Ganesan 

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
