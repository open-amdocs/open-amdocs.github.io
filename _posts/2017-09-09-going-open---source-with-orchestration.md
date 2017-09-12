---
layout: post
title:  Going Open-Source with Orchestration
date:   2017-09-09 13:12:47
image: /assets/article_images/2017-09-09-going-open---source-with-orchestration/os-orchastration.jpg
image2: /assets/article_images/2017-09-09-going-open---source-with-orchestration/os-orchastration_mobile.jpg
imageSrc: https://www.flickr.com/photos/oregondot/13976835716
author: Prakhar Pandey
---
The breakthrough of virtualization technology in the field of telecom has been accompanied by a host of possibilities for provisioning network services as virtual functions to the customers. Network Function Virtualization (NFV) limits the amount of physical effort required in onboarding hardware devices at customer premises and replaces them with their equivalent virtual functions, thereby reducing costs, labor, and intensive hours of management.

Revolutions in NFV also meant that the orchestrator solutions used in achieving this purpose were not far behind and undergoing technical advancements of their own. One of the most recent advancements was going open-source. Companies are making the source code of such solutions available to the public not only to set standards but also for the overall enhancement of their ideas, designs and implementations. One of these open-source solutions is the Open Network Automation Platform (ONAP).

ONAP is a merger of two different software platforms – AT&amp;T&#39;s Open ECOMP, and OPEN-O backed by the Linux Foundation. As co-founder of ECOMP, Amdocs was out to prove that we were ready to implement customized use cases on the open platform.

# The Motivation

With respect to AT&amp;T&#39;s ECOMP, Amdocs was already an integral part of co-creation and had a major role to play in its development. Given the vast knowledge we had already acquired during this partnership, we were inclined to keep pace with the developments. Our chief motivation was to present to the community how easy it was to onboard an end-to-end use case with the new open-source solution. We wanted to explore this solution which had quickly attracted a lot of popularity in the community. At this point of time, the aim was to come up with a use case and build it on top of ONAP. This post is a summary of this experience.

# The Problem Statement

We had already got our hands a bit dirty with ONAP 1.0 running the out-of-the-box use cases provided by ONAP, to get the initial know-how and look and feel of the solution. But this was just the start. Our agenda was to come up with a time-to-market demo which we could present to our customers and further strengthen our position as one of the leaders in the ONAP solution domain.

We chose SD-WAN Site Connectivity as our primary use case. This selection was merited by the popularity of the use case among customers and also supported by the fact that we had great deal of knowledge in it based on previous work with customers. The advantage was that we already knew how SD-WAN connectivity works and we could focus our efforts more on the ONAP part of the entire solution.

The actual problem statement in the first phase was to provide site-to-site connectivity between multiple sites for a given customer. We came up with two sub use cases: one, to provide full-mesh connectivity among the sites, and two, to mark one of the sites as a hub and route all traffic between any two sites through it.

Apart from ONAP, we supplemented the solution by using Nokia Nuage as our SD-WAN Controller (based on our success and know-how of the solution from previous work experience) and BSS-Lite to provide business consumer side capabilities.

# Challenges 

We were working with a team that had no prior experience on ONAP or even ECOMP for that matter. Everything ranging from architecture to code was new for us. Hence, it was bound to bring a lot of complexities and black-boxes. To gain some experience, we tried to play around with the out-of-the-box demos that ONAP provides.

Onboarding a complex use case using a solution we just got to know a bit is never easy. While the out-of-the-box demos helped us gain knowledge across the breadth of the ONAP solution, at this point of time there was still lots to explore. The main challenge was working with the very first release of ONAP. In this version, not all the modules were at the same maturity level. Remember that the first release had ONAP micro-services that were way behind their version equivalent of the propriety solutions which were open-sourced later, and hence this was a challenge to build an end-to-end solution. Then there were other issues, like coming up with the right architecture, design, and presentation value.

# Solution Architecture
![ONAP Integration with SD-WAN Solution using Nokia Nuage as Controller](/assets/article_images/2017-09-09-going-open---source-with-orchestration/ONAP-Integration-with-SD-WAN.png)

I still remember all the brainstorming over solution design. At first it was hard to come up with a proper workflow inside ONAP. Comparing it with the out-of-the-box demo design didn&#39;t prove to be of much use, since that was based upon the premise of VNF onboarding using OpenStack Heat Orchestration. But this use case was different. There were no VNF&#39;s to onboard thus far, and certainly OpenStack was not in picture as well.

This was the very first problem we faced in our journey. What should the solution design look like inside ONAP ? What components do we use ? What components do we not use ? Which component should do what – since the joint architecture was still unclear, and there were several components with shared responsibilities.

There were questions like which components require significant development. Where do we define our business flows. Who talks to the Nuage libraries - the Master Service Orchestrator (MSO) or the SDN-Controller (SDN-C). A lot of leading product owners and architects were involved in the discussions. Finally, we came up first with an effective workflow within ONAP, and then incorporated the external components – Nokia Nuage and BSS system – into the design.

 ![High-level Workflow of SD-WAN solution on top of ONAP](/assets/article_images/2017-09-09-going-open---source-with-orchestration/highe-level-WF.png)



# To Model or not to Model?

Once we started exploring the depths of ONAP, we encountered some really tough dilemmas on the implementation level. Service Design and Creation (SDC) is no doubt a great tool to model the services on ONAP, but one major issue with our specific use case was that the model definition was closely linked to the premise of VNF onboarding. The standard modelling process involves defining network functions in a HeatStack which is fed to SDC. This information is then used by SDC to come up with a model consisting of one or more VNFs and links between them.

However, with the SD-WAN use case, we were dealing with Physical Network Functions (PNFs). As per the scope of this phase, our aim was to provide dual-linked site-to-site connectivity and did not involve onboarding VNFs in the data center or the Physical CPE. And certainly, thus far, we had no use for OpenStack Heat Orchestration and hence, were not equipped with a HeatStack to feed SDC.

After consultation, we decided to handle PNFs as yet another VNF and model accordingly. The model was more like the depiction of the use case diagram showing the PNFs and network trace for routing MPLS and internet traffic.

# Black Boxes

Being able to run the ONAP out-of-the-box demo is surely a great thing, but it cannot be relied upon as the sole way of learning all that ONAP has to offer. You could learn some things about some components, but there is much more to it than meets the eye. We learned this the dirty way, getting our hands deep into the code. SDN-C and A&amp;AI (Active &amp; Available Inventory) are two such examples. Knowing what a component does is one thing, trying to code that component to help onboard our own use case is something completely different. It is natural to expect some surprises here and there and working with your first custom use case with ONAP is a time-intensive learning process.

It was only with hands-on experience that we could meet this challenge. For us, A&amp;AI and SDN-C seemed to be the most effort-intensive components. However, once we started getting the hang of things, we were able to succeed.

We were able to define a clear data model in A&amp;AI featuring a customer at the top-level of the hierarchy. In the same data model, we subscribed this customer to a service (in our case, SD-WAN) and then defined several service instances under it – each corresponding to a new customer site. Using this pyramid, we were easily able to group data logically in A&amp;AI, and then it was pretty easy to store Nuage generated data for a site in the respective service instance.

Regarding the SDN-C, we approached this by defining our own directed graphs and their respective endpoints to be called by the MSO. Thanks to the OpenDaylight Project framework, we were able to define our custom bundles, where we wrapped the Nuage libraries we had created on top of SDN-C. The directed graphs then referred to these Nuage libraries with the help of an adaptation layer we developed.

After getting some real hands-on experience like this, we can safely say that these black-boxes were not black-boxes anymore.

# Show Time

After three months of effort and dealing with the challenges mentioned above (and a few more), we came up with a full end-to-end demo for the use cases. This was the first-time a customized and sophisticated use case like SD-WAN was up and running, and it created a buzz around the organization and attracted a large audience.

 ![Launching a new Site from BSS](/assets/article_images/2017-09-09-going-open---source-with-orchestration/launching-new-site.png)

# Feedback and immediate improvements

While the demo was largely welcomed across the audience, the jury is still out whether we can take it as it is to the customer. There are some points one has to ponder.

# Presentation Value

Yes, anyone will say that the Core part of the solution is the engine of it, but taking a solution to the customers attracts a varied audience. The BSS and Nokia Nuage Director have fancy User Interfaces, but customers are more interested in ONAP right now. Unfortunately, for our use case, we were not in a position to use every component of ONAP as a demo-able interface. However, I would not describe this as a very alarming situation. With the subsequent releases, we will surely be able to add to the presentation value. One such example is A&amp;AI. Currently, we are only able to present on A&amp;AI the number and spawning status of sites for a given customer. With the new version, we will be able to map all the network entities created on Nokia Nuage to a data model on A&amp;AI which can provide an even better visualization and also provide real-time data such as site activation status, link bandwidth etc.

# Take it one level higher

While the site-to-site connectivity is great to begin with, eventually the market will have greater expectations. The task at hand is to supplement the already existing use case we have with the ability to onboard VNF&#39;s. That is the point of NFV. Customers want to see their firewalls and WAN optimizers virtualized in the data center (and even on the physical CPE). The initial use cases can be seen as a good learning tool to help build upon them even more complex use cases and complete a full set to show to customers.

We have taken the first leap, now we are ready to run the race.

# Shout out to upcoming program

We are well on-road to kick-start **&#39;Amdocs NFV powered by ONAP&#39;** in mid-September. One of the key motives behind this program is to accelerate our capability to adopt to NFV. Some of the highlights of the launch include:

- &#39;Single click&#39; ONAP installation – expedite the ONAP environment deployment by providing a user-friendly Interface to setup ONAP with one click.
- SD-WAN acceleration kit with business processes and best practices, allowing for the provision to install the SD-WAN solution on top of an ONAP Setup as a package.
- Hosting Accelerator development environment on public cloud for CSPs interested in testing and verifying virtual services on ONAP open source platform.

As one of the top-contributors to ONAP, we are happy to say that we are moving in the right direction and are keen to face new challenges that are to follow.
