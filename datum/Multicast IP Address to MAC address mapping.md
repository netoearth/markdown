Multicast IP address live in the 224.0.0.0 – 239.255.255.255 range but what about MAC addresses and Ethernet frames? What do we do on layer 2 to make multicast work? Let me show you an example of a MAC address:

![mac broadcast multicast bit](https://cdn.networklessons.com/wp-content/uploads/2013/03/mac-broadcast-multicast-bit.png)

Above you see an example of a MAC address. In the first octet, bit 0 has been reserved for broadcast or multicast traffic. When we have unicast traffic this bit will be set to 0. For broadcast or multicast traffic this bit will be set to 1.

On layer 3 IANA has reserved the class D range (224.0.0.0 – 239.255.255.255) for multicast IP addresses. What about layer 2? What MAC addresses do we use for multicast traffic?

For layer 2 we also have a reserved prefix to use for multicast traffic. The **24-bit MAC address prefix 01-00-5E** is reserved for layer 2 multicast. Unfortunately only half of the MAC addresses in this 24-bit prefix can be used for multicast, this means we only have 23 bits of MAC address space to use for multicast. Here’s an illustration:

![multicast mac address 23 bit](https://cdn.networklessons.com/wp-content/uploads/2013/03/multicast-mac-address-23-bit.png)

As you can see the first 3 octets are 01-00-5E. This is the reserved range. This means there are 8+8+8 = 24 bits left for us to use. I just told you that only half of this 24-bit space is available to us which means that only 23 bits can be used. Why can we only use 23 bits?

There’s a funny story why we only have 23 bits left…back in the days (1990 something) Steve Deering was working on his research on IP multicast and he wanted the IEEE to assign 16 OUIs (Organizational Unique Identifiers) to IP multicast MAC addresses. each OUI has 24 bits of address space, so 16 x 24 bits would supply enough MAC addresses to create a 1:1 relation between multicast IP address and multicast MAC address.

Each OUI costed $1000 and Steve’s manager didn’t want to pay 16 x $1000 = $16.000 just for MAC address space. As a result Steve’s manager bought a single OUI (24 bit) and gave half of the space (23 bit) to Steve to use for his multicast research. Why does this matter? Let me show you:

![multicast 28 unique bits](https://cdn.networklessons.com/wp-content/uploads/2013/03/multicast-28-unique-bits.png)

Above you see an IP address which has 32 bits. A multicast IP address also has 32 bits but the first 4 bits are always the same (1110) because we use the 224.0.0.0 – 239.255.255.255 range. This means that each multicast IP address has **28 unique bits**.

Now if we want to map our 28 bit multicast IP address to our 23 bit MAC address we have a problem…we miss **5 bits of mapping information:**

![multicast ip to mac mapping](https://cdn.networklessons.com/wp-content/uploads/2013/03/multicast-ip-to-mac-mapping.png)

This means that we have to map multiple Multicast IP addresses to the same Multicast MAC address. We don’t have enough MAC addresses to give each multicast IP address its own MAC address.

We miss 5 bits of mapping information: 25 = 32. This means we will map **32 multicast IP addresses to 1 multicast MAC address**. Here’s an example:

-   224.1.1.1
-   224.129.1.1
-   225.1.1.1
-   …
-   …
-   …
-   238.1.1.1
-   238.129.1.1
-   239.1.1.1

The multicast IP addresses above all map to the same multicast MAC address (01-00-5E-01-01-01). This can cause some problems in our networks. For example, a host that listens to the 239.1.1.1 multicast IP address will configure its network card to listen to MAC address 01-00-5E-01-01-01. If someone else is streaming to the 224.1.1.1 multicast IP address it will also end up at our host because the MAC address is the same. The host will have to look at the IP address of the received frame to see if it’s for 239.1.1.1 and discard frames that are meant for 224.1.1.1.

Now the big question remains…what multicast IP addresses map to which multicast MAC address and how do we calculate this? You can use a calculator of course but if you are studying for a Cisco exam you don’t have this luxury. Let’s take a look at how to do this!

First we’ll figure out which multicast MAC address maps to which 32 multicast IP addresses. You can use the following table  to calculate between decimal, hexadecimal and binary:

<table><tbody><tr><td>Decimal</td><td>0</td><td>1</td><td>2</td><td>3</td><td>4</td><td>5</td><td>6</td><td>7</td><td>8</td><td>9</td><td>10</td><td>11</td><td>12</td><td>13</td><td>14</td><td>15</td></tr><tr><td>Hexadecimal</td><td>0</td><td>1</td><td>2</td><td>3</td><td>4</td><td>5</td><td>6</td><td>7</td><td>8</td><td>9</td><td>A</td><td>B</td><td>C</td><td>D</td><td>E</td><td>F</td></tr><tr><td>Binary</td><td>0000</td><td>0001</td><td>0010</td><td>0011</td><td>0100</td><td>0101</td><td>0110</td><td>0111</td><td>1000</td><td>1001</td><td>1010</td><td>1011</td><td>1100</td><td>1101</td><td>1110</td><td>1111</td></tr></tbody></table>

We will take the following multicast MAC address and calculate what 32 multicast IP addresses map to it:

**01:00:5e:0b:01:02**

First we have to translate this MAC address from hexadecimal to binary:

<table><tbody><tr><td>0</td><td>1</td><td>0</td><td>0</td><td>5</td><td>e</td><td>0</td><td>b</td><td>0</td><td>1</td><td>0</td><td>2</td></tr><tr><td>0000</td><td>0001</td><td>0000</td><td>0000</td><td>0101</td><td>1110</td><td>0000</td><td>1011</td><td>0000</td><td>0001</td><td>0000</td><td>0010</td></tr></tbody></table>

Above you can see how I translated the hexadecimal address into binary, this is the full MAC address:

<table><tbody><tr><td>0000 0001</td><td>0000 0000</td><td>0101 1110</td><td>0000 1011</td><td>0000 0001</td><td>0000 0010</td></tr></tbody></table>

Now we will take the lowest 23 bits of this MAC address:

<table><tbody><tr><td>0000 0001</td><td>0000 0000</td><td>0101&nbsp;1110</td><td>0<span>000 1011</span></td><td><span>0000 0001</span></td><td><span>0000 0010</span></td></tr></tbody></table>

The bits that I highlighted in red are the lowest 23 bits of the MAC address.

Now we will take the class D multicast IP address range in binary:

<table><tbody><tr><td><span>1110</span> <span>0000</span></td><td><span><span>0</span><span>000</span> <span>0000</span><br></span></td><td><span>0000 0000</span></td><td><span>0000 0000</span></td></tr></tbody></table>

The digits in blue (1110) are the class D IP address in binary (224 in decimal). The green digits are the 5 bits that we lose because we have to map a 28 bit unique multicast IP address to a 23 bit multicast MAC address. We will take the blue and green digits and put the red digits behind them:

<table><tbody><tr><td><span>1110</span> <span>0000</span></td><td><span>0</span><span>000 1011</span></td><td><span>0000</span> <span>0001</span></td><td><span>0000</span> <span>0010</span></td></tr></tbody></table>

Let’s convert this binary address into a decimal IP address:

<table><tbody><tr><td>224</td><td>11</td><td>1</td><td>2</td></tr><tr><td><span>1110</span> <span>0000</span></td><td><span>0</span><span>000 1011</span></td><td><span>0000 0001</span></td><td><span>0000</span><span> 0010</span></td></tr></tbody></table>

So the complete multicast IP address is 224.11.1.2. Now we can play with the green digits to see what other multicast IP addresses map to the same MAC address:

<table><tbody><tr><td><strong>Binary Multicast IP Address</strong></td><td><strong>Decimal Multicast IP Address</strong></td></tr><tr><td><span>1110</span> <span><span>0000</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>224.11.1.2</td></tr><tr><td><span>1110</span> <span><span>0001</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>225.11.1.2</td></tr><tr><td><span>1110</span> <span><span>0010</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>226.11.1.2</td></tr><tr><td><span>1110</span> <span><span>0011</span> 0</span><span>000 1011 0000 0001 0000 0010</span></td><td>227.11.1.2</td></tr><tr><td><span>1110</span> <span><span>0100</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>228.11.1.2</td></tr><tr><td><span>1110</span> <span><span>0101</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>229.11.1.2</td></tr><tr><td><span>1110</span> <span><span>0110</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>230.11.1.2</td></tr><tr><td><span>1110</span> <span><span>0111</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>231.11.1.2</td></tr><tr><td><span>1110</span> <span><span>1000</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>232.11.1.2</td></tr><tr><td><span>1110</span> <span>1</span><span><span>001</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>233.11.1.2</td></tr><tr><td><span>1110</span> <span><span>1010</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>234.11.1.2</td></tr><tr><td><span>1110</span> <span><span>1011</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>235.11.1.2</td></tr><tr><td><span>1110</span> <span><span>1100</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>236.11.1.2</td></tr><tr><td><span>1110</span> <span><span>1101</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>237.11.1.2</td></tr><tr><td><span>1110 <span><span>1110</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></span></td><td>238.11.1.2</td></tr><tr><td><span>1110</span> <span><span>1111</span> <span>0</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>239.11.1.2</td></tr><tr><td><span>1110</span> <span><span>0000</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>224.139.1.2</td></tr><tr><td><span>1110</span> <span><span>0001</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>225.139.1.2</td></tr><tr><td><span>1110</span> <span><span>0010</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>226.139.1.2</td></tr><tr><td><span>1110</span> <span><span>0011</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>227.139.1.2</td></tr><tr><td><span>1110</span> <span><span>0100</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>228.139.1.2</td></tr><tr><td><span>1110</span> <span><span>0101</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>229.139.1.2</td></tr><tr><td><span>1110</span> <span><span>0110</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>230.139.1.2</td></tr><tr><td><span>1110</span> <span><span>0111</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>231.139.1.2</td></tr><tr><td><span>1110</span> <span>1</span><span><span>000</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>232.139.1.2</td></tr><tr><td><span>1110</span> <span><span>1001</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>233.139.1.2</td></tr><tr><td><span>1110</span> <span><span>1010</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>234.139.1.2</td></tr><tr><td><span>1110</span> <span><span>1011</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>235.139.1.2</td></tr><tr><td><span>1110</span> <span><span>1100</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>236.139.1.2</td></tr><tr><td><span>1110</span> <span><span>1101</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>237.139.1.2</td></tr><tr><td><span>1110</span> <span><span>1110</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></td><td>238.139.1.2</td></tr><tr><td><span>1110 <span><span>1111</span> <span>1</span></span><span>000 1011 0000 0001 0000 0010</span></span></td><td>239.139.1.2</td></tr></tbody></table>

There you have it, all the multicast IP addresses that map to multicast MAC address 01:00:5e:0b:01:02. Now you know how it is done in binary, you can learn a faster method to calculate these IP addresses.