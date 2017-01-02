---
title: 'remote desktop connection to localhost: a regression in Windows 7?'
id: 101
date: 2010-01-27 22:57:16
tags:
- putty
- rdc
- remote desktop connection
- ssh
- ssh tunnel
- tcpip
- win7
- windows
- winxp
---

I maintain a Windows server. It is web-facing, and lives in a DMZ on the other side of the world from me. I have to install new programs every now and then. Windows being Windows, it's easiest to do this with a desktop session. Remote Desktop Connection is the key tool for doing this. Since the version of Remote Desktop Protocol (RDP) I'm connecting to [isn't secure](http://en.wikipedia.org/wiki/Remote_Desktop_Protocol) over the public Internet I use an [ssh tunnel](http://en.wikipedia.org/wiki/Ssh_tunnel#SSH_tunneling) to connect. This is easy to [set-up in Putty](http://oldsite.precedence.co.uk/nc/putty.html).

![01_initial_settings.png](http://intwoplacesatonce.com/wp-content/uploads/2010/01/01_initial_settings.png)

An ssh tunnel works by accepting packets on one side of the ssh connection, and putting them back in to the TCP/IP stack on the other side of the tunnel -- as if the packets originated from the "far" computer. This can be done in either direction. In the screenshot above I've configured a tunnel accepting packets on my local machine. They will be re-injected on the remote machines stack addressed to "localhost:3389". In other words a program connecting to my computer's port 3390 will actually connect to the remote computer's port 3389\. Port 3389 is Remote Desktop Protocol, so if I point RDC at localhost:3390, I'll connect to the remote computer's RDP server.

![02_RDC_connection_localhost.png](http://intwoplacesatonce.com/wp-content/uploads/2010/01/02_RDC_connection_localhost.png)

I recently started using Windows 7 and this set up broke. It seems in Windows 7, Remote Desktop Connection prevents connections to localhost. Trying to work around the limit using 127.0.0.1 or your public IP address or computer name does not work either. RDC still recognises that you are, apparently, connecting to the computer you are already connected to. This is an awkward limitation when using an ssh tunnel or some other connection forwarding.

Luckily there is a workaround.

Apparently Windows XP before service pack 2 had [this same limitation](http://www.bitvise.com/remote-desktop). People worked around it by pointing RDC at 127.0.0.2\. It's not used that often, but the whole range of addresses starting with 127 are all routed back to the local machine. In other words you always [have a /8 network](http://en.wikipedia.org/wiki/Localhost#IPv4) running on your own machine. To make this work, I had to check the "Local ports accept connections from other hosts" option for putty. Without the option putty will only listen for connections to address 127.0.0.1\. With the option it accepts connections on any address. Now I can point RDC at 127.0.0.2:3390 and get connected to the remote desktop, securely.

![03_revised_settings.png](http://intwoplacesatonce.com/wp-content/uploads/2010/01/03_revised_settings.png)

It seems a strange limitation for RDC to refuse to connect to localhost. I can understand the initial idea; having this limit would prevent remoting to a computer you are already remoted to. That's an easy enough mistake to make if you are managing several servers, and it's a nice save. The strange bit is that someone repealed the limit in XP SP2, but now it is back again. How does that happen? Was SP2 on a branch, and they forgot to merge it back? Was the limit in the original spec, and the spec didn't get updated when the limitation was removed? Did they just decide the limit feature was back in? As someone stung by the reintroduction of the feature, it feels like an accidental regression.

![04_RDC_connection_127_0_0_2.png](http://intwoplacesatonce.com/wp-content/uploads/2010/01/04_RDC_connection_127_0_0_21.png)
