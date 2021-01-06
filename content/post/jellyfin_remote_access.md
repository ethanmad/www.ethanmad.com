---
title: 'Jellyfin Remote Access with Tailscale'
Description: 'Set-up guide for Jellyfin or other local services over Tailscale'
date: '2021-01-07'
blurb: 'This is a guide on setting up Jellyfin for remote access by yourself & friends using Tailscale with a reverse proxy to get easy-to-rembemer URs like https://jellyfin.ethanmad.com or http://jellyfin. Though this is written about Jellyfin, you can probably follow along for any other local service.'
emoji: '🖥️'
Tags: ['networking', 'self-host']
---

![Jellyfin on my phone over LTE, through Tailscale](jellyfin_over_lte_800x.jpg "Look ma, no LAN! - Connecting to my Jellyfin server over LTE, through Tailscale")

A few months ago,  I set up a [Jellyfin](https://jellyfin.org) media server on my desktop so that I could stream content from my library to my phone in order to watch shows in bed.
That was pretty cool, but what if I wasn't home?
I live with some housemates and don't have access to port forwarding settings on our router and have a dynamic IP address.
With exposing the service to the Internet not an easy option, using a VPN was my next thought.

I had previously heard about [Tailscale](https://tailscale.com), a mesh VPN network using Wireguard. Since it handles NAT-traversal, is free to use, and [BSD-licensed](https://github.com/tailscale/tailscale/blob/main/LICENSE "The network service is non-free, of course. Use plain Wireguard or Nebula if you want control over the network."), this seemed like a perfect solution.[^Tailscale alternatives]
I didn't see any guides about setting up remote access to Jellyfin using Tailscale or similar, so here's mine!
I'm on Arch Linux, but most steps will be similar regardless of operating system.

[^Tailscale alternatives]: There are some alternatives to Tailscale you might consider as I did, namely plain [Wireguard](https://www.wireguard.com/), [ZeroTier](https://https://www.zerotier.com/), and [Nebula](https://github.com/slackhq/nebula). I didn't want to configure Wireguard on each device I wanted to share access with, so that was out. ZeroTier's website is broken by my adblockers, so I passed on it. Nebula has a great [set-up guide by Ars Technica](https://arstechnica.com/gadgets/2019/12/how-to-set-up-your-own-nebula-mesh-vpn-step-by-step/), but it's slower than Wireguard and not as polished. That's how I decided on Tailscale, and I'm happy with my choice so far.

*This post written with some feedback by the Tailscale team after I participated in a survey, but it is not sponsored by Tailscale.*

# Jellyfin

1. If you haven't installed Jellyfin, follow the [Quick Start](https://jellyfin.org/docs/general/quick-start.html) guide to get going.
    * Don't worry about step 5 (secure the server); we'll get to that.
    * No, you don't need to use the Networking tab in Jellyfin's settings.

# Tailscale & DNS

Tailscale is a mesh VPN network, which means you can treat remote devices as if they're on your local network. Tailscale assigns each device an [IP address](https://tailscale.com/kb/1033/ip-and-dns-addresses) in the `100.x.y.z` range. Only you (or those you give access) can access your device with the given IP address.

1. [Register for Tailscale](https://login.tailscale.com/start). A free account will work fine.
1. Install the Tailscale app on your server and any clients.
Enable the VPN with `$ tailscale up` on Linux, or by clicking the toggle on other operating systems.
1. [Test your connection](https://tailscale.com/kb/1030/next-steps) as described or pinging one of the IP addresses assigned to you. `$ tailscale status` will show IP addresses of all of your connected devices.
1. If you want to access your server via a subdomain like `jellyfin.ethanmad.com`, create an `A` record on your domain with name `jellyfin` and the IP address assigned to your server by Tailscale (e.g., `100.109.161.29`).
1. If you'd rather use Tailscale's Magic DNS to access your devices by their hostname (e.g., `ethanmad-desktop`), then enable "[Magic DNS](https://tailscale.com/kb/1081/magic-dns)"[^Magic DNS] in your [Tailscale admin panel](https://login.tailscale.com/admin/dns).
    * You'll need to add a nameserver to make this work; if you don't already have one, you can [pick a provider](https://www.privacytools.io/providers/dns/).
    (I use [NextDNS](https://nextdns.io) so that connecting to Tailscale blocks ads for me. If you don't know what you're looking for, you can use Cloudflare's `1.1.1.1`.)
1. Try accessing your Jellyfin server by entering either `http://jellyfin.ethanmad.com:8096` or `http://ethanmad-desktop:8096`, using your domain or hostname instead of mine. If you can connect, you're set! In the next step, we'll add TLS and remove the need for the port number.
1. If you want to share your device with friends, [share access with them](https://tailscale.com/kb/1084/sharing)[^sharing]. *Your friends will have access to any port unless you set up an [access control list (ACL)](https://tailscale.com/kb/1018/acls#acls) restricting their access!*

[^Magic DNS]: At the time of writing, Magic DNS is a public beta feature. I don't think you will be able to use HTTPS just yet, but I think a new Tailscale feature will address this in the near future. Device hostnames will also soon be renamable, in case you'd prefer to access your server another way.
[^sharing]: At the time of writing, sharing nodes is an opt-in beta feature. I heard from Ross at Tailscale that it's receiving better control features soon.

# Reverse Proxy and HTTPS

We will use Caddy[^reverse proxy] to reverse proxy port requests on ports 80 (HTTP) and 443 (HTTPS) to 8096 (Jellyfin) *and* to set up TLS & HTTPS.

Jellyfin provides a [guide](https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148) for using Caddy as a reverse proxy, but it will not enable HTTPS.
Since Tailscale's underlying protocol, Wireguard, encrypts traffic, TLS doesn't add much value other than removing the browser nag;
you can safely skip TLS use that guide and skip setting up TLS if you're short on time.

1. [Install Caddy](https://caddyserver.com/docs/install) after reading the rest of this section to determine which version you need.
1. If you're using your domain name (`jellyfin.ethanmad.com`) to access Jellyfin, you can use the [DNS challenge](https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148) to set up TLS. Pick whichever method is more convenient for you of the two.
1. If you're using Tailscale's Magic DNS, I don't think you can get a publicly-trusted TLS certificate at the time of writing. This may change in the future, since Tailscale is considering adding a built-in reverse proxy to make this easier.
1. Create `/var/lib/caddy/Caddyfile` and, using mine (below) as a reference, add in your configuration.
    * If you only want access via one of subdomain or Magic DNS, then take add just the relevant section to your Caddyfile. My Caddyfile shows both.
    * The listed Cloudflare API key is an example; it is not really mine. You would use your API key for your DNS provider instead. Do not share API keys with others.
    * Note that the Magic DNS configuration requires specifying port 80 since Caddy tries to automatically set up HTTPS.
1. If you don't already have one, set up a [systemd service file for Caddy](https://caddyserver.com/docs/install#linux-service) and enable it so it's always running: `$ systemctl enable --now caddy`
1. Try accessing your Jellyfin server, e.g., by entering either `https://jellyfin.ethanmad.com` or `http://ethanmad-desktop`, using your domain or hostname instead of mine. If you can connect, you're done.

``` Caddyfile
# /var/lib/caddy/Caddyfile
{
    email ethan@ethanmad.com
}
jellyfin.ethanmad.com {
    tls {
        dns cloudflare 1484053787dJQB8vP1q0yc5ZEBnH6JGS4d3mBmvIeMrnnxFi3WtJdF # not my real key
    }

    reverse_proxy :8096
}
ethanmad-desktop:80 {
    reverse_proxy :8096
}
```


[^reverse proxy]: There are other equally viable reverse-proxy options, like Apache, Nginx, and Traefik. I like Caddy: I use it elsewhere, set-up is easy, and it handles TLS itself. Ross told me Tailscale is adding a built-in reverse proxy, which will eliminate the need for running one locally.

# Conclusion

Share your services with your friends and family. All they have to do is sign up for Tailscale using the node sharing link you send them and connect. I've been using it to share access to Jellyfin with friends and family across the US without problems. If you need help, see the [Tailscale forums](https://forum.tailscale.com/).

Here's the message I sent to my dad when sharing with him. You could use something similar:
> To access my media library, you need to use A VPN to connect. So first download Tailscale (https://tailscale.com/download) and log in with your Google account. I'll send you an link which you'll need to open to gain access to my server.
>
> Then install the Jellyfin app (https://jellyfin.org/clients/) if you want to watch on your phone.
>
> Once both are downloaded, turn on Tailscale then open Jellyfin and enter https://jellyfin.ethanmad.com as the server address. Then you can browse and watch whatever you want! Use AirPlay or Chromecast to get it on the TV.


# Bonus ideas
- Add [jfa-go (jellyfin-accounts go)](https://github.com/hrfee/jfa-go) so you don't have to deal with account creation or password resets for friends.
- Integrate [Ombi](https://github.com/tidusjar/Ombi) so friends can request content without messaging you.
- [Limit access to your server with `ufw`](https://tailscale.com/kb/1077/secure-server-ubuntu-18-04#step-3-allow-access-over-tailscale).
    - The linked guide's rules are pretty restrictive, so use your judgement when deciding which rules to use on your system.
    - Since we are using a reverse proxy, Jellyfin is already accessed over ports 80 and 443; there's no need to add a special rule for it.
    - Don't forget to add rules for any other services (e.g., mosh, syncthing, etc.) you need access to.
