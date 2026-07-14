# Common Network Security Threats: A Research Report

## Introduction

Modern organizations depend on interconnected networks for nearly every business function, from processing payments to storing patient records to running industrial control systems. This dependence has made networks an attractive target: a single successful attack can disrupt operations, expose sensitive data, damage brand reputation, and trigger significant financial and regulatory consequences. The threats covered in this report — denial-of-service attacks, man-in-the-middle attacks, IP spoofing, and DNS poisoning — are not new, but they remain persistently effective because they exploit fundamental design assumptions in internet protocols (such as the trust placed in source IP addresses, unauthenticated routing announcements, and unencrypted name resolution) that are difficult to fix at internet scale. Understanding how these attacks work, the damage they have caused in practice, and the mitigations that actually reduce risk is foundational knowledge for any network administrator or security practitioner.

## 1. Denial-of-Service (DoS) and Distributed Denial-of-Service (DDoS) Attacks

### How It Works
A DoS attack attempts to make a system, service, or network resource unavailable to legitimate users by overwhelming it with traffic or exploiting a resource-exhaustion flaw. A DDoS attack is the same concept carried out from many distributed sources simultaneously — typically a botnet of compromised devices — making the traffic harder to filter and the source harder to trace. According to CISA, common techniques include SYN floods, in which an attacker repeatedly sends TCP connection requests but never completes the handshake, tying up server ports until none are left for legitimate users, and Smurf attacks, which spoof a victim's IP address in broadcast ICMP requests so that many hosts flood the victim with replies. A related and increasingly common category is the reflection/amplification attack, in which an attacker sends a small, spoofed request to a misconfigured public service (such as DNS, NTP, or memcached) that is instructed to send its much larger reply to the victim rather than the attacker, multiplying the attack traffic by orders of magnitude.

### Real-World Example
On October 21, 2016, Dyn — a major DNS provider — was hit by a massive DDoS attack built on the Mirai botnet, which had compromised hundreds of thousands of internet-connected devices such as DVRs, routers, and IP cameras, largely by exploiting default factory credentials. <cite index="9-1">The attackers flooded Dyn's DNS infrastructure with a huge volume of DNS lookup requests originating from tens of millions of IP addresses.</cite> Because so many major websites relied on Dyn to resolve their domain names, the attack caused <cite index="2-1">over 50 major internet platforms — including Twitter, Netflix, PayPal, Reddit, Amazon, Spotify, and Sony — to become temporarily inaccessible across the northeastern United States and parts of Europe.</cite> A separate but related incident illustrates the amplification variant: in February 2018, GitHub was hit with a <cite index="27-1">1.35 Tbps DDoS attack that abused publicly exposed memcached servers, which were tricked via spoofed source IP addresses into sending oversized responses toward GitHub rather than the attacker</cite>, with an amplification factor of up to roughly 51,000x the size of the original request.

### Impact
- Extended service outages affecting not just the direct target but every downstream customer relying on the same infrastructure (as seen with Dyn's customers).
- Direct financial losses from business interruption, incident response, and reputational damage; Dyn faced substantial recovery costs and reputational harm following the 2016 attack.
- Erosion of customer trust and potential SLA or contractual penalties for affected service providers.
- Exposure of secondary vulnerabilities — the Dyn/Mirai incident highlighted how poorly secured consumer IoT devices can be weaponized at internet scale.

### Mitigation Strategies
1. **Rate limiting and traffic filtering at the network edge** — deploying upstream scrubbing services, load balancers, and firewalls that can absorb or filter volumetric traffic before it reaches core infrastructure (a core recommendation in CISA's DDoS Quick Guide).
2. **Redundant, geographically distributed DNS and infrastructure** — using multiple DNS providers and anycast routing so that an attack on one provider or region does not take down the entire service, a direct lesson learned from the Dyn incident.
3. **Closing amplification vectors** — disabling or firewalling unnecessary UDP-based services (DNS, NTP, memcached, SSDP) from responding to the public internet, and enabling ingress/egress filtering (BCP 38) to prevent spoofed source addresses from leaving a network in the first place.

## 2. Man-in-the-Middle (MITM) Attacks

### How It Works
In a MITM attack, an adversary secretly positions themselves between two communicating parties, intercepting and potentially altering traffic while both sides believe they are communicating directly with each other. <cite index="17-1">Common techniques include ARP spoofing, which sends fake Address Resolution Protocol messages to associate the attacker's MAC address with a target IP address on the local network; SSL/TLS stripping, which downgrades an HTTPS connection to unencrypted HTTP; and rogue Wi-Fi access points (evil twins), which trick users into connecting to an attacker-controlled hotspot.</cite> A more targeted variant involves compromising or forging digital certificates so that a fraudulent site or proxy appears to present a trusted, valid certificate to the victim's browser.

### Real-World Example
In 2011, attackers breached Dutch certificate authority DigiNotar and used the compromise to <cite index="12-1">issue more than 500 fraudulent security certificates for major websites including Google, Yahoo, and Microsoft, which were then used to conduct MITM attacks against users, primarily in Iran; the breach was severe enough that DigiNotar was removed as a trusted certificate authority and subsequently declared bankruptcy.</cite> A more consumer-facing example occurred in 2015, when it was discovered that Lenovo had preinstalled adware called Superfish on consumer laptops; <cite index="15-1">the software installed its own root certificate authority in Windows so it could transparently intercept encrypted HTTPS traffic and inject advertisements, effectively breaking the trust model that HTTPS relies on for every user of the affected laptops.</cite>

### Impact
- Theft of credentials, session tokens, and financial data transmitted over the intercepted connection.
- Complete breakdown of trust in the encryption chain — once a rogue certificate authority is trusted by a device, every "secure" connection made through it can be silently intercepted.
- In enterprise contexts, MITM-enabled business email compromise has been used to alter invoice payment details, redirecting large wire transfers to attacker-controlled accounts.
- Reputational fallout severe enough to end a company's business, as happened to DigiNotar.

### Mitigation Strategies
1. **Enforce HTTPS/TLS everywhere with HSTS** — ensuring servers reject downgraded HTTP connections and browsers refuse to load insecure fallback versions of a site, which directly closes the SSL-stripping vector.
2. **Certificate transparency and certificate pinning** — monitoring public Certificate Transparency logs for unauthorized certificates issued for an organization's domains, and pinning certificates in mobile/critical applications so a fraudulent CA-issued certificate is rejected even if it is technically "valid."
3. **Network-layer protections against local interception** — using dynamic ARP inspection and port security on switches to prevent ARP spoofing, and requiring VPN use on untrusted networks such as public Wi-Fi to encrypt traffic end-to-end regardless of the local network's trustworthiness.

## 3. IP Spoofing

### How It Works
IP spoofing is the practice of crafting network packets with a forged source IP address so that the receiving system believes the traffic originated from a different, often trusted, host. This is possible largely because <cite index="21-1">the core design of the internet makes it difficult for a receiving network to verify whether a packet's source address is legitimate, in part due to multihoming, where a network may have more than one internet provider and there is no reliable way to confirm which path is authentic.</cite> Spoofing underlies several attack types, most notably reflection/amplification DDoS attacks, where an attacker forges the victim's IP address in requests sent to third-party servers, causing those servers to unknowingly flood the real victim with responses.

### Real-World Example
The February 2018 attack on GitHub is a clear illustration: <cite index="28-1">an attacker spoofed their IP address to appear as the victim's address, then sent forged requests to vulnerable, publicly exposed memcached servers; because the servers had no way to verify the request's origin, they sent their (much larger) responses directly to the spoofed victim address</cite> rather than back to the attacker. This produced a record-breaking 1.35 Tbps flood against GitHub using no botnet at all — just spoofed packets aimed at misconfigured infrastructure. The same underlying technique was used against several other organizations that same week, with peak traffic reported as high as 1.7 Tbps.

### Impact
- Enables massively amplified DDoS attacks (as seen with GitHub) with a small investment of attacker bandwidth.
- Can be used to bypass IP-based access controls or authentication schemes that trust source addresses (for example, "trusted host" firewall rules).
- Complicates attribution and incident response, since logs point to spoofed, often innocent, third-party IP addresses rather than the true attacker.
- Implicates innocent third parties — networks running the misused reflector service (e.g., exposed memcached or DNS servers) can be blamed or blocklisted for traffic they did not intentionally send.

### Mitigation Strategies
1. **Ingress/egress filtering (BCP 38 / RFC 2827)** — configuring network edge routers to drop outbound packets whose source address does not belong to the originating network's assigned address block, preventing spoofed packets from ever leaving the attacker's network.
2. **Securing and firewalling reflector-capable services** — ensuring services like DNS, NTP, and memcached are not needlessly exposed to the public internet on UDP, and disabling protocol features (like memcached's UDP support) that are not required for legitimate use.
3. **Deploying anti-spoofing and reputation-based filtering upstream** — using ISP- or CDN-level scrubbing (e.g., Cloudflare, Akamai) that can detect and drop anomalous, spoofed-looking traffic patterns before they reach the target network, and participating in community efforts such as CAIDA's Spoofer project to identify and close spoofing-permissive networks.

## 4. DNS Poisoning / Spoofing (Bonus Threat)

### How It Works
DNS poisoning (also called DNS cache poisoning or DNS spoofing) occurs when an attacker corrupts the records held by a DNS resolver or otherwise manipulates the DNS resolution process so that a domain name resolves to an attacker-controlled IP address instead of the legitimate one. <cite index="37-1">This is a cyberattack in which a victim's DNS server is tricked into associating a hostile server with a valid domain name, exposing users who type in the genuine domain name to phishing, malware, and other fraud.</cite> One powerful way to achieve this at scale is by combining it with a Border Gateway Protocol (BGP) hijack: an attacker announces false routing information claiming ownership of IP address ranges that belong to a legitimate DNS provider, causing internet traffic destined for that provider's real DNS servers to be misdirected to the attacker instead.

### Real-World Example
On April 24, 2018, attackers targeted MyEtherWallet, a web-based cryptocurrency wallet. <cite index="34-1">Using a compromised upstream internet service provider, the attackers hijacked roughly 1,300 Amazon-owned IP address ranges via a BGP route announcement, redirecting traffic intended for Amazon's Route 53 DNS service.</cite> <cite index="31-1">Attackers then pointed DNS resolutions for the MyEtherWallet.com domain to a server in Russia hosting a fake copy of the site that logged victims' private keys.</cite> The fake site used a self-signed TLS certificate, which triggered browser warnings, but <cite index="31-1">not all users heeded the warning and proceeded to log in anyway, resulting in stolen funds.</cite> Notably, neither AWS nor MyEtherWallet's own systems were compromised — the attack exploited trust gaps in the BGP and DNS protocols themselves, external to either company's infrastructure.

### Impact
- Direct financial theft — the MyEtherWallet incident resulted in roughly $150,000–160,000 in stolen cryptocurrency in about two hours.
- Large-scale phishing enabled without any need to compromise the victim organization's own servers, making the attack far harder for the target to detect or prevent unilaterally.
- Erosion of trust in DNS resolution generally, since even users who type the correct URL can be routed to a malicious server.
- Potential for follow-on attacks such as malware distribution, credential harvesting, or man-in-the-middle interception once traffic has been redirected.

### Mitigation Strategies
1. **Deploy DNSSEC** — cryptographically signing DNS records so resolvers can verify that a response has not been tampered with, preventing a poisoned or spoofed record from being accepted as authentic.
2. **Use DNS resolvers and monitoring services that support RPKI/BGP route validation** — Resource Public Key Infrastructure (RPKI) lets networks cryptographically validate that a BGP route announcement actually comes from the legitimate owner of that IP block, directly closing the vector used in the MyEtherWallet hijack.
3. **Enforce HSTS and certificate pinning for critical domains, and monitor for anomalous DNS/BGP activity** — so that even if traffic is redirected, browsers and users are warned by certificate mismatches (as happened with MyEtherWallet's self-signed certificate), and organizations can detect route hijacks quickly using services like BGP monitoring feeds.

## Comparison Table

| Threat | Attack Vector | Who Is at Risk | Difficulty to Execute | Ease of Mitigation |
|---|---|---|---|---|
| DoS / DDoS | Traffic flooding, resource exhaustion, reflection/amplification via spoofed requests to third-party services | Any internet-facing service; especially high-value or high-traffic sites and their downstream customers | Low–Medium (botnets and amplification tools are widely available; low technical skill needed) | Medium (requires upstream scrubbing, redundancy, and provider cooperation; hard to fully prevent, easier to absorb) |
| Man-in-the-Middle (MITM) | ARP spoofing, rogue Wi-Fi, SSL stripping, compromised/fraudulent certificates | Users on shared/public networks, mobile app users, organizations relying on weak certificate validation | Medium (local network attacks are easy; CA compromise or large-scale interception is harder) | Medium–High (TLS/HSTS, certificate pinning, and switch-level ARP protection are well-established and effective) |
| IP Spoofing | Forging the source IP address in packets, often combined with reflection services | Networks hosting exposed UDP services (DNS, NTP, memcached); ultimately any DDoS target | Low–Medium (crafting spoofed packets is straightforward without egress filtering in place) | Medium (BCP 38 ingress/egress filtering is effective but requires broad ISP adoption to fully eliminate) |
| DNS Poisoning / Spoofing | Cache poisoning, BGP route hijacking to redirect DNS resolution | Any organization/user relying on unauthenticated DNS; especially high-value targets like financial and crypto platforms | Medium–High (cache poisoning requires precise timing; BGP hijacking requires compromising or controlling network infrastructure) | Medium (DNSSEC and RPKI are effective but adoption across the internet remains incomplete) |

## Conclusion

Three key takeaways for a network administrator emerge from this research. First, many of the most damaging attacks — DDoS amplification, IP spoofing, and BGP-enabled DNS hijacking — succeed not because of a flaw unique to the victim, but because foundational internet protocols (IP, BGP, DNS) were designed without built-in authentication, meaning defense often depends on protections deployed by other networks (egress filtering, RPKI, DNSSEC) as much as your own. Second, redundancy and defense-in-depth consistently reduce blast radius: organizations that relied on a single DNS provider (Dyn's customers) or a single layer of trust (unencrypted DNS, unpinned certificates) suffered outsized impact compared to those with layered protections. Third, user-facing warning signals — such as certificate errors — are a critical last line of defense, but cannot be relied upon alone, since the MyEtherWallet and Equifax incidents both show that users will often click through security warnings under time pressure; technical controls (DNSSEC, certificate pinning, HSTS) must be enforced automatically rather than left to user judgment.

## References

1. Cybersecurity and Infrastructure Security Agency (CISA). *Understanding Denial-of-Service Attacks.* https://www.cisa.gov/news-events/news/understanding-denial-service-attacks

2. Cybersecurity and Infrastructure Security Agency (CISA) & National Security Agency (NSA). *Understanding and Responding to Distributed Denial-of-Service Attacks.* https://www.cisa.gov/sites/default/files/2024-03/understanding-and-responding-to-distributed-denial-of-service-attacks_508c.pdf

3. National Institute of Standards and Technology (NIST). *SP 800-189: Resilient Interdomain Traffic Exchange: BGP Security and DDoS Mitigation.* https://csrc.nist.gov/pubs/sp/800/189/final

4. National Institute of Standards and Technology (NIST). *SP 800-61 Revision 3: Incident Response Recommendations and Considerations for Cybersecurity Risk Management.* https://csrc.nist.gov/pubs/sp/800/61/r3/final

5. MITRE ATT&CK. *Network Denial of Service (T1498) and Adversary-in-the-Middle (T1557).* https://attack.mitre.org/

6. Wikipedia. *DDoS attacks on Dyn.* https://en.wikipedia.org/wiki/DDoS_attacks_on_Dyn

7. GitHub Engineering Blog. *February 28th DDoS Incident Report.* https://github.blog/news-insights/company-news/ddos-incident-report/

8. IBM. *What Is a Man-in-the-Middle (MITM) Attack?* https://www.ibm.com/think/topics/man-in-the-middle

9. Wikipedia. *Man-in-the-middle attack.* https://en.wikipedia.org/wiki/Man-in-the-middle_attack

10. Cloudflare Blog. *The Real Cause of Large DDoS: IP Spoofing.* https://blog.cloudflare.com/the-root-cause-of-large-ddos-ip-spoofing/

11. BleepingComputer. *Hacker Hijacks DNS Server of MyEtherWallet to Steal $160,000.* https://www.bleepingcomputer.com/news/security/hacker-hijacks-dns-server-of-myetherwallet-to-steal-160-000/

12. The Register. *AWS DNS Network Hijack Turns MyEtherWallet into ThievesEtherWallet.* https://www.theregister.com/2018/04/24/myetherwallet_dns_hijack/
