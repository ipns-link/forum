# Static Manifest Gateways

## Issues to Solve

1. DNSLink lacks wildcard domains, so no `kid.ipns.static-gateway.tld`.
2. `ipfs.io/ipns/gateway-cid/ipns/kid`, whether accessed directly or through a DNSLink like `gateway.tld/ipns/kid`, will fail with 404.

However, what might work is:

1. `gateway-cid.ipns.ipfs.io`
2. `ipfs.io/ipns/gateway-cid`

## Solution

Instead of a Gateway which redirects the Browser to a subdomain, we can have the Manifest act as a Static Gateway for just it's Origin, this removes the need for redirects and subdomains. This new type of Manifest is known as a **Static Manifest Gateway**.

`Origin <-IPFS-> WAN <-IPFS-> Browser`

1. Browser uses js-ipfs to connect to Origin over WSS.
2. Origin and Browser communicate over libp2p-stream.
3. Browser converts libp2p-stream data into a website within an `<iframe>`.
4. User can access the Origin through the Browser.

## Static Manifest Gateway

Manifests are added inline with IPNS-PubSub, however, they ideally need to be as small as possible. To ensure Manifest's stay small, and to remove redundancy across Manifests, the `js-ipfs` code is stored in a separate `js-ipfs.js` file which can be hosted on IPFS or other sources.

#### index.html
```
<src="CID of js-ipfs script/other link to js-ipfs script">...

<input>Please enter the ciphertext password...</input>

<iframe>Origin's website here...</iframe>

<!--IPNS-Link--
ciphertext in multibase (base64, inline)
--IPNS-Link-->
```

#### js-ipfs.js
```
<script>

  js-ipfs code here...

</script>
```

Instead of encrypting the JSON with GPG and the public keys of gateways, you instead encrypt it with a user-defined password to be entered in the Manifest, this prevents random people from navigating to a Manifest and getting instant access to the Origin tied to it.

To allow the Manifest to contact the Listener, a js-ipfs node is run within the Manifest and will decrypt the ciphertext to find the Listener. Once connected, PubSub messages will be sent between them.

The Browser then takes what the Origin sends and displays it within an `<iframe>`, this works for both static and dynamic applications running on the Origin, the user can now interact with the service exposed through the Origin.

# Summary

## Advantages

- Static Manifest Gateways can be fetched by public IPFS Gateways or locally run IPFS Gateways.
- IPFS Gateway's can't read what's sent between the Origin and Browser. The content of the `<iframe>` is also client-side only.
- IPFS Gateway's won't suffer the bandwidth from the connection since the IPFS connection is peer-to-peer.
- They are extremely difficult to block or censor.
- The main functionality doesn't require any modifications or extensions for the Browser.
- Manifest's can be kept minimal by sourcing the Javascript files from elsewhere (Ideally from IPFS when possible).
- Manifest's can be themed per the user's liking to create a sort of pre-home page.
- Query paramaters, only read by the JS code and not sent to the Origin, allow extra flexibility and can recover functionality such as by doing `http://gateway-cid.ipns.dweb.link/?url=/images/image1` to get resource URLs back.

## Disadvantages

- Malicious Public IPFS Gateways know what Manifest you are accessing and when unless extra precautions like VPNs are taken.

## Possible Improvements

- Create custom javascript code to render the libp2p-stream data into a frame without the need for `<iframe>`.
- Create a Static Gateway for the Exposer where the user can choose which Origin they want to access from a list of multiple `ID:` sections in the JSON. Note, we would need a way to prevent associating services to each other on top of this.
- Make the Listener a Delegated Router for the js-ipfs node in the Browser to use, just as an extra option for the operator.

## Resources for HTTP websites over WSS

*This was my research into if websites can be displayed over a WebSocket connection, it didn't really lead me anywhere, but, I've included it anyways...*

https://testdriven.io/blog/html-over-websockets/

https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html

https://github.com/tanrax/demo-HTML-over-WebSockets-in-Django

https://github.com/tani/hyper-tunnel

https://github.com/arthurkushman/php-wss

