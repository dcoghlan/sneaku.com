---
date: '2026-09-15T09:02:25+10:00'
draft: true
title: "Guess Who's Back"

Params:
  ShowReadingTime: true
  ShowCodeCopyButtons: true
  ShowBreadCrumbs: true

cover:
  image: "images/Uhana.png"
  alt: "Uhana"
  relative: true
  hiddenInSingle: true #controls visibility of cover image in single page post
---

After 2114 days, I'm back and will be starting to post again. So where did I go? 

<!--more-->

Lets wind back the clock to late 2020, right when the world was just starting to
hear rumours of a suspicious virus cirulcating in China. The team that I was
a part of within VMware was on its last legs, and by Jan 2021 the team was
disbanded. Some of the team were let go and others were transferred to the local
Professional Services teams where we could stay whilst we looked for other
opportunities if we wanted. I was lucky enough to have stayed on within PS, but
I was on the hunt for my next opportunity.

My old [PowerNSX](https://powernsx.github.io/) colleague Nick Bradford had jumped ship earlier to go and work
at a startup and as luck would have it, in 2019 VMware acquired the startup he
had moved to, a small company called Uhana.

As I am not great a "marketing speak", here is what
[sdxcentral](https://www.sdxcentral.com/news/vmware-snaps-up-ai-expert-uhana-to-bolster-5g-efforts/)
said about the product when the VMware aquisition was announced

>Uhana’s core technology can be deployed in an operator’s private cloud or in
>its public cloud infrastructure. It includes a stream processing engine tha
>ingests subscriber-level network telemetry from the radio access network (RAN),
>the core network, and over-the-top (OTT) applications. It uses that data to
>provide real-time, per-subscriber visibility.
>
>It also has an AI engine that can discover and predict anomalies in the network or the application, prioritize anomalies by their estimated impact on overall operations, infer the root cause of the anomaly, and recommend ways to fix the issues.

So Nick and I had been staying in contact, and the opportunity arose to take a
step away from working on NSX and move into a completely new role as a Site
Reliability Engineer (SRE) for the Uhana SAAS platform. So I took the plunge to
get completely out of my comfort zone.

Joining the Uhana team exposed me to the telco world, specifically the Radio
Access Network (RAN) side of things and it was a massive learning curve. There
were so many acronyms and pieces of technology which I had never heard before,
at times it was quite over-whelming.

The Uhana software platform was also built upon a cloud native architecture,
think Kubernetes, Helm, Docker, Kafka, Protobuf, Minio, Cassandra, ArgoCD, Envoy,
Nginx, Flink, Prometheus, Grafana and probably a lot more stuff I've now forgotten.

Everythng was built with the automation first mindset which suited me to a tee.
Looking back now, it was the perfect time to learn on this platform as in the
beginning there was no Chat-GPT etc so I had to learn the old school way.

Over the next few years we kept the platforms lights on, running through
quarterly upgrades, adding field developed functionality and implementing a
GitOps & DevOps pipelines to manage the platforms. It was extremely rewarding
and we learnt a lot about how to actually run cloud native applications in the
real world.

By the end of 2023, Broadcom had completed its acquisition of VMware and we were
lucky enough to have survived the transition. However, the writing was on the
wall once again. Broadcom was going to focus on supplying the software to power
private clouds (VCF) and Uhana, along with the broader Telco division, were not
part of that vision.

So by mid 2025, Uhana was officially End Of Life'd and once again I was on the
hunt for my next opportunity within the business.

Which brings me to today. I have landed in the Application & Network Security
(ANS) Business Unit within Broadcom, specifically in a team called the ANS Centre
Of Excellence (COE). It is working back with NSX, but with a focus on the
security side of things which has been rebranded as vDefend, think Distributed 
Firewall, Gateway Firewall, IDPS, NTA/NDR as well as the new [Security Services Platform (SSP)](https://techdocs.broadcom.com/us/en/vmware-security-load-balancing/vdefend/security-services-platform/5-2/security-services-platform-overview.html).

I didn't post at all during my period with Uhana as the technology was so niche,
however you may see me posting some more content again as there is likely a bit
more of an audience for the vDefend security related stuff.

