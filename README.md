# Community VPN for Slippi

Many Slippi players have consistent problems connecting to other players.
They are often caused by routing or traversal issues outside of the players' control, often at the ISP level.
In these cases, tunneling through a VPN can solve connection issues and even lower ping.
Retail VPNs can work for individual players but don't scale to the population of a whole region.
For example, many ISPs in the Philippines use CGNAT which makes it difficult to establish domestic P2P connections.

To that end, I have created a VPN specifically for the Melee community in the Philippines using cloud compute.
With the caveat that I'm a professional software developer, I can say it was not terribly difficult to get up and running, even starting from minimal experience with VPN software, server administration, and even cloud providers.
It is low maintenance and cost around $6.50/month at peak usage.
In case anyone is interested in doing the same for another region, I will elaborate on the setup:

## Cloud Provider
There are many metrics by which you can evaluate cloud providers, but in this case the most important is latency.
Since this VPN is for the Philippines, I chose Alibaba Cloud which has a datacenter in Manila.
The downsides to this included: no relevant free trial and immediate payment processing issues.

## Protocol/Server: Wireguard
Some rudimentary research suggested performance advantages over other VPN protocols/servers.
As VPNs are relatively overkill for our use case (just proxying), I looked extensively into using a SOCKS5 proxy via Dante instead, but Wireguard won out due to the "Virtual Network" part of VPN (every Wireguard peer gets a static IP) being useful for bandwidth limiting.
The SOCKS5 protocol has lower overhead (in terms of bandwidth) and doesn't require unique credentials per user so if you have another way to limit bandwidth, it could be preferrable.
In my experiments Dante seemed to counterintuitively consume *more* CPU on the server than Wireguard.
That may be due to kernel-mode (Wireguard) vs user-mode (Dante), UDP not being supported, but not a first-class citizen in SOCKS5, or some other unknown factors.

## Client: WireSock + WireSockUI
Together, these provide a simple UI and can be entirely configured (including credentials) through .conf files which we prepare in advance and distribute to users on-demand.
Importantly, it has split-tunneling capability (to route only Slippi traffic) which prevents accidental misuse (a relatively common problem with sharing accounts on retail VPNs).
Make sure you turn "Virtual Network Adapter mode" on. This may fix connection loops in the scenario where both players are using the VPN.
<img width="604" height="537" alt="image" src="https://github.com/user-attachments/assets/929a266c-8600-40b7-b19a-246c9d66acda" />

## Bandwidth Limiting
Unfortunately, the split-tunneling is configured client-side, so we have to guard against *intentional* misuse.
Since Wireguard requires assigning each peer on the virtual network a static IP, we are able to use the utility "tc" to do simple bandwidth limiting.
Using Wireguard, Slippi singles seems to very consistently use ~200kbps, so I have the limit set at 800kbps, leaving headroom for doubles (assumed to use ~600kbps).
This is one part where I'm convinced there must be some better way.
Ideally, we would simply limit outgoing traffic to *any single IP+port combination* to 800kbps, but I couldn't find (in a reasonable timeframe) a way to do this.
If you have any suggestions, please get in touch!

## Results
No users have reported still being unable to connect to other players while using the VPN.
Additionally some players report a dramatic decrease in ping, in one case down from 60-70ms to 10ms when connecting to other players in the Philippines.
The server can have at least 254 users configured and should be able to support dozens of simultaneous clients.
Again, it cost $6.50 per month at peak usage.

## Config Samples
I consulted various guides on setting up Wireguard that you can find by Googling.
Important to note, I configured the firewall at the cloud provider level (not using "ufw" on the server).
Also make sure to check your server's network device.
On Alibaba Cloud it is eth0 like most guides say, but on Google Cloud it was ens4.

wireguard server: https://pastebin.com/raw/vxDrSh6S

wireguard client: https://pastebin.com/raw/pinjZC1q

Additionally I followed https://www.procustodibus.com/blog/2022/12/limit-wireguard-bandwidth/ to set up bandwidth limiting via "tc"
https://pastebin.com/raw/3vZkyV0k
