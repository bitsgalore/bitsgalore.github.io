---
layout: post
title: Moving my Internet domains
tags: [internet, DNS, GitHub-Pages]
comment_id: 84
---

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2023/02/donkey-cart.png" alt="Icon of a donkey that is pulling a cart that has the words bitsgalore.org written on it. In the background the sun is shining.">
  <figcaption>Donkey, cart and sun icons licensed from <a href="https://thenounproject.com/">the Noun Project</a>.</figcaption>
</figure>

I recently moved the two Internet domains I own away from the UK-based domain registrar I'd been using since 2004 to a EU-based registrar. While the actual domain transfer was fairly simple, finding a registrar that suited my specific situation turned out more difficult than expected. Leaving my old registrar also resulted in a surprise. It's unlikely that my situation is unique, so I thought it would be useful to share my experiences in this blog post, and point to some useful online resources that I found along the way. The move also allowed me to make my domains up to date with (mostly security-related) modern internet standards. I'll briefly address this in the final sections of this post. This includes some suggestions on how to make these optimizations work with a GitHub Pages-hosted sites like this one.

<!-- more -->

## A brief history of my domains

I registered my first (*.net*) domain in early 2004. At the time, I was only looking for a "stable" e-mail address that was independent of any proprietary service or internet service provider. Having my own domain seemed the best way to realise this. I've been using this domain ever since, and my private e-mail address has remained unchanged throughout that 19 year period. By the end of 2013 I registered a second (*.org*) domain, which is linked to this very site[^1]. Both domains were registered with a UK-based registrar and domain hosting company, and I was also using their name servers. Both domains are linked to servers that are run by other providers for e-mail and web hosting. For example, this site uses [GitHub Pages](https://pages.github.com/) for web hosting.

## Why move?

After my registrar was bought out by another company in 2018, it seems the new owners weren't all that interested in keeping up with the latest Internet standards. The Dutch Internet Standards Platform has an [online tool](https://internet.nl/) that checks a domain for the use of modern (mostly security-related) Internet standards, and I used this to analyse this site some time ago. [This result](https://web.archive.org/web/20230210222235/https://internet.nl/site/www.bitsgalore.org/1921771/), which is from a re-run I did just a few days before the domain transfer, shows that the site's performance in this regard was very poor indeed:  

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2023/02/internetnl-10022023.png" alt="Screenshot of website test summary info for domain www.bitsgalore.org. It shows the site got an overall score of 55%, reporting problems related to IPv6 reachability, DNSSEC and HTTPS.">
</figure>

Without going too much into detail here, the main culprits were:

1. My registrar's domain servers weren't reachable via [IPv6](https://en.wikipedia.org/wiki/IPv6) addresses.
2. My registrar didn't offer [DNSSEC](https://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions) (Domain Name System Security Extensions).

These technical shortcomings aside, in this post-Brexit era I'm not keen on routing my e-mail traffic through domain servers that are located in a non-EU country. On top of that, after a series of price hikes, my registrar had become significantly more expensive than most of its competitors. This was made even worse by the introduction of substantial bank transfer fees after Brexit. In the end, I felt the premium pricing I was paying was out of balance with the comparatively poor service provided.

## 2030: A Domain Odyssey 

With the renewal date of my email domain approaching again, I started the search for a new registrar. This was complicated somewhat by something that happened in November 2019. At that time, [Internet Society](https://en.wikipedia.org/wiki/Internet_Society) made an [announcement](https://www.internetsociety.org/news/press-releases/2019/ethos-capital-to-acquire-public-interest-registry-from-the-internet-society/) that they were about to sell the .org top-level domain to a private equity firm. Just a few months earlier, [ICANN](https://en.wikipedia.org/wiki/ICANN) had lifted the price caps on *.org* domains. This would give the new owner total freedom to raise prices on *.org* domains without any limitations[^3]. As this would include renewals of existing domains, I immediately renewed the *bitsgalore.org* domain for the maximum period possible (ten years), while it was still relatively cheap. In the end my ten year renewal proved to be unnecessary, as the sale of the *.org* top-level domain was [vetoed by ICANN](https://www.theregister.com/2020/05/01/icann_stops_dot_org_sale/) several months later. But by that time I already was the proud owner of a domain that wouldn't expire until late 2030[^4].

## What happens to the remaining registration period?

So what are the implications of transferring a domain that has already been paid for in advance? Does the new registrar take over the remaining contract period (until 2030) in that case? Or would the transfer reset the registration period, meaning I would lose the remaining 8 years at my previous registrar, and I'd have to pay for them again? My online research on this didn't come up with any definitive answers, but most sources I found suggested that in most cases, any remaining years at the "old" registrar are automatically transferred to the "new" registrar. Additional payments are usually limited to either a transfer fee, or the renewal of the domain by one more year. See for example [this thread](https://webmasters.stackexchange.com/questions/44258/domain-name-transfer-do-you-lose-years-left-at-current-registrar) on Webmasters Stack Exchange, and [this article](https://www.thesitewizard.com/domain/renewing-domain-name.shtml).

I also came across [this document that is part of ICANN's registry agreements](https://www.icann.org/en/registry-agreements/multiple/revised-verisign-registry-agreements-appendix-c-16-4-2001-en), which states (section 2.4 Transfer Grace Period):

> Transfer (other than ICANN-approved bulk transfer). If a domain is transferred within the Transfer Grace Period, there is no credit. The expiration date of the domain is extended by one year up to a maximum term of ten years.

If I'm reading this correctly, this also suggests that transferring a domain to another registrar doesn't result in the loss of the remaining registration period.

## Dutch treat

However, it seems not all registrars work that way. TransIP, which is a major Dutch domain registrar and hosting provider, [explicitly mention on their website](https://www.transip.nl/knowledgebase/artikel/24-een-domeinnaam-naar-transip-verhuizen/) that transferring an existing domain to them will start a new contract period, and that they will not take over any contract periods with the existing registrar. Two other Dutch registrars I contacted by e-mail told me this as well, adding that the transfer would reset the existing renewal date. Both advised me to stick with my current registrar. I don't know if this behaviour is unique to Dutch registrars. In any case, the responses I got did prompt me to widen the search net to registrars in other EU countries.  

## Enter the Franco-German axis

[One of the replies](https://webmasters.stackexchange.com/a/44347) in the aforementioned Webmasters Stack Exchange thread mentions [Gandi](https://en.wikipedia.org/wiki/Gandi), a French registration and hosting company. The link in that post doesn't work anymore, but a quick search on Gandi's documentation site turned up [this table](https://docs.gandi.net/en/domain_names/transfer/transfer_table.html#ot). For a *.org* domain, this states that, after transfer, the "new expiration date will be 1 year after current expiration date". While searching for some more information about Gandi, I stumbled across [this website](https://european-alternatives.eu/category/domain-name-registrar), which lists several EU-based domain name registrars. I'm not sure how independent the site is, and what its selection of registrars is exactly based on. With that said, I looked up the mentioned registrars, checked some online reviews, and this directed my attention to German domain and hosting company [INWX](https://www.inwx.com/) as another potential candidate.

I e-mailed both Gandi and INWX, and asked them what would be the status of my *bitsgalore.org* domain in case  I transferred it to them. Both companies confirmed they would take over the remaining 8 years of the contract with my current registrar, and would only charge for renewing the domain by one more year (extending the registration until 2031). This was exactly the answer I was hoping for, as this would save me 8 years worth of renewal fees. I ultimately settled on INWX, although both companies look like good choices to me.

## Dealing with transfer-out fees

I then started the transfer out with my old registrar[^7], who turned out to charge a £10 + VAT administration fee for this on each domain. From what I understand, this practice is quite unusual, and in my case it would set me back for some £24. I don't consider myself overly stingy, but paying money like that just to leave a company that's not providing a great service to begin with seemed a bit much. A quick search turned up [this forum thread](https://forums.digitalspy.com/discussion/comment/91500345/#Comment_91500345). It links to ICANN's [FAQs for Registrants: Transferring Your Domain Name](https://www.icann.org/resources/pages/name-holder-faqs-2017-10-10-en), which states (under "*My registrar is charging me a fee to transfer to a new registrar. Is this allowed?*"):

> [...] Registrars are allowed to set their own prices for this service so some may choose to charge a fee. However, a transfer cannot be denied due to non-payment of this transfer fee. There are other reasons your registrar can deny transfer request. See FAQ #8 above for more information.

So, I contacted my old registrar, and asked them to waive the transfer fee, referring to the ICANN document. To their credit, they granted my request without any fuss, and promptly sent me the authorization codes!

## Starting the incoming transfer

With these authorization codes, I could now start the incoming end of the transfer at INWX. It's worth pointing out it took five days for the transfer to take effect (I received a notification e-mail about this); from what I understand this is usually the case for *.org* and *.net* domains. However, I was able to edit the DNS records at INWX well before the actual transfer date. When the transfer happened, it didn't result in any noticeable downtime on either domain.

## Modern Internet standards

Once I had verified that everything was working OK after the transfer, I activated [DNSSEC](https://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions) on both of my domains. I then did a couple of tests with the [online tool](https://internet.nl/) of The Dutch Internet Standards Platform[^5]. A test on the *bitsgalore.org* apex (root) domain now [resulted](https://web.archive.org/web/20230218140111/https://internet.nl/site/bitsgalore.org/1922728/) in a respectable 94% score, but repeating the test for the *www* subdomain [only yielded a score of 70%](https://web.archive.org/web/20230218140314/https://internet.nl/site/www.bitsgalore.org/1925412/). A closer look at the detailed results showed that the *www* subdomain was failing on some of the DNSSEC tests.

## Making DNSSEC work on the subdomain

The [documentation of GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain-and-the-www-subdomain-variant) on custom domains recommends to insert a [CNAME](https://en.wikipedia.org/wiki/CNAME_record) record for *www* subdomains, and this is also how I originally set up this site. After checking with INWX's support (which, by the way, is both excellent and fast) about my subdomain issue, they advised me to replace this CNAME record with a series of [A](https://en.wikipedia.org/wiki/List_of_DNS_record_types#A) records for the www subdomain. In the end I went one step further, and also added corresponding [AAAA](https://en.wikipedia.org/wiki/List_of_DNS_record_types#AAAA) records[^6].

To illustrate this further, I removed this record:

|Name|Type|TTL|Value|
|:-|:-|:-|:-|
|`www`|`CNAME`|`86400`|`bitsgalore.github.io`|

And replaced it with the records below:

|Name|Type|TTL|Value|
|:-|:-|:-|:-|
|`www`|`A`|`86400`|`185.199.111.153`|
|`www`|`A`|`86400`|`185.199.110.153`|
|`www`|`A`|`86400`|`185.199.108.153`|
|`www`|`A`|`86400`|`185.199.109.153`|
|`www`|`AAAA`|`86400`|`2606:50c0:8001::153`|
|`www`|`AAAA`|`86400`|`2606:50c0:8000::153`|
|`www`|`AAAA`|`86400`|`2606:50c0:8003::153`|
|`www`|`AAAA`|`86400`|`2606:50c0:8002::153`|

Here, the A records point to the [IPv4](https://en.wikipedia.org/wiki/Internet_Protocol_version_4) addresses of GitHub's servers, and the AAAA records point to the [IPv6](https://en.wikipedia.org/wiki/IPv6) addresses of the same servers. I didn't need to make any changes to the site's GitHub repo.

After this change, DNSSEC worked as intended for the *www* subdomain, and the Internet Standards tool now [resulted in a 97% score](https://web.archive.org/web/20230217140008/https://internet.nl/site/www.bitsgalore.org/1928399/):

<figure class="image">
  <img src="{{ BASE_PATH }}/images/2023/02/internetnl-14022023.png" alt="Screenshot of website test summary info for domain www.bitsgalore.org. It shows the site got an overall score of 97%.">
</figure>

There are still a couple of minor things that could be improved (mainly related to GitHub's servers it seems). Overall I'm happy with this result though, especially when keeping in mind that the site only scored a meagre [55%](https://web.archive.org/web/20230210222235/https://internet.nl/site/www.bitsgalore.org/1921771/) just a week earlier.

## Email 

I also ran separate tests on [email](https://internet.nl/test-mail/) for both my domains, and used the results to apply some optimizations. This is beyond the scope of this post (which is already getting too long), but I think it deserves a brief mention. For example, in November 2022 Google implemented a [policy](https://support.google.com/mail/answer/81126?hl=en&ref_topic=7279058#auth-reqs) that requires that the domains of incoming email messages are secured with either [SPF](https://en.wikipedia.org/wiki/Sender_Policy_Framework) or [DKIM](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail). If an incoming message doesn't comply with this, Google's servers will either mark it as spam, or even outright reject it. I learnt about this the hard way myself at the time, when all of a sudden I was unable to reach any of my email contacts with a Gmail address!

## Conclusion

The main thing I learnt about this domain move, is the importance of researching registrars' terms and conditions for domains that have been paid for several years in advance. I was surprised at how different registrars handle this in unexpectedly different ways. The somewhat nebulous situation around transfer-out fees was another surprise, and it's good to know one's rights here. In the end, I estimate my online research in both areas saved me a total of around €150, which is quite substantial!

This exercise also made me more aware of the importance of selecting a registrar that keeps up with modern Internet standards, by using name servers with IPv6 web addresses, and offering signed domain names (DNSSEC). Tools like [the one by the Dutch Internet Standards Platform](https://internet.nl/) are hugely helpful for compliance testing against these standards. Overall, I'm very happy with the outcome. Both my domains are now largely compliant with modern Internet standards, while running at a lower cost compared to my previous registrar. 

## Useful links and resources

- [Internet.nl](https://internet.nl/) - online tool by the Dutch Internet Standards Platform that tests web and email domains for use of modern Internet standards.

- [European domain name registrars](https://european-alternatives.eu/category/domain-name-registrar) - lists some more EU-based domain registrars.

- [FAQs for Registrants: Transferring Your Domain Name](https://www.icann.org/resources/pages/name-holder-faqs-2017-10-10-en) - by ICANN.

- [Revised VeriSign Registry Agreements: Appendix C](https://www.icann.org/en/registry-agreements/multiple/revised-verisign-registry-agreements-appendix-c-16-4-2001-en) - part of ICANN's [Generic Top-Level Domain](https://en.wikipedia.org/wiki/Generic_top-level_domain) (gTLD) Registry Agreements.
 

[^1]: This was originally meant to be a personal blog covering a variety of subjects. That never really worked out, and in 2019 I re-launched the site as a home for my digital preservation writings old and new. 

[^3]: See The Register for details - ["Internet world despairs as non-profit .org sold for \$$$$ to private equity firm, price caps axed"](https://www.theregister.com/2019/11/20/org_registry_sale_shambles/)

[^4]: My other domain, which I only use for e-mail, is a .net domain, and I renew this annually.

[^5]: After the transfer it took several hours for the tool to pick up the new name servers. This is probably caching-related, and something to keep in mind when using tools like these on freshly transferred domains.

[^6]: I'm not completely sure the AAAA records are really necessary here.

[^7]: Before starting a domain transfer, it's a good idea to first copy your domain's DNS settings (which you can find in your registrar's admin interface) to a text file. This will enable you to enter the relevant DNS records at your new registrar later.