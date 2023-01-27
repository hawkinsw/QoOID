---
title: "Quality of Outcome"
abbrev: "QoO"
category: info

docname: draft-olden-ippm-qoo-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Transport"
workgroup: "IP Performance Measurement"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "IP Performance Measurement"
  type: "Working Group"
  mail: "ippm@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/ippm/"
  github: "domoslabs/QoORFC"
  latest: "https://domoslabs.github.io/QoORFC/draft-teigen-ippm-qoo.html"

author:
 -
    fullname: Magnus Olden
    organization: Domos
    email: magnus@domos.no
    street: Gaustadalléen 21
    code: 0349
    country: NORWAY
 -
    fullname: Bjør Ivar Teigen
    organization: Domos
    email: bjorn@domos.no
    street: Gaustadalléen 21
    code: 0349
    country: NORWAY

normative:

informative:
  RFC9318: # IAB Workshop report
  RFC8290: # FQ_CoDel
  RFC8033: # PIE
  TR-452.1:
    title: "TR-452.1: Quality Attenuation Measurement Architecture and Requirements"
    author:
      org: Broadband Forum
    date: September 2020
    format:
      PDF: https://www.broadband-forum.org/download/TR-452.1.pdf
  BITAG:
    title: Latency Explained
    author:
      org: BITAG
    format:
      PDF: https://www.bitag.org/documents/BITAG_latency_explained.pdf
    date: October 2022
  RPM:
    title: Responsiveness under Working Conditions
    target: https://datatracker.ietf.org/doc/html/draft-ietf-ippm-responsiveness
    date: July 2022
  RRUL:
    title: Real-time response under load test specification
    target: https://www.bufferbloat.net/projects/bloat/wiki/RRUL_Spec/
  Bufferbloat:
    title: "Bufferbloat: Dark buffers in the Internet"
    target: https://queue.acm.org/detail.cfm?id=2071893
  Haeri22:
    title: "Mind Your Outcomes: The ΔQSD Paradigm for Quality-Centric Systems Development and Its Application to a Blockchain Case Study"
    target: https://www.mdpi.com/2073-431X/11/3/45


--- abstract

This document describes a new network quality framework named Quality of Outcome(QoO). The QoO framework is uniqe among network quality frameworks in satisfying all the requirements layed out in "Requirements for a Network Quality Framework Useful for Applications, Users and Operators".

The framework proposes a way of sampling network quality, setting network quality requirements and a formula for calculating the probability for a sampled network to reach network requirements.

--- middle

# Introduction

Quality attenuation is a network quality metric that meets the three criteria we set out in the requirements; it captures the probabilty of good or bad application outcomes, it is composable, and it can be compared to a variety of application requirements. The part that is still missing is how to present quality attenuation results to end-users and application developers in an understandable way. We believe a per-application (or per application-type) approach is appropriate here. The challenge lies in how to simplify. We must specify how and when it is apporpriate to throw away information without loosing too much precision and accuracy in the results.

We propose measuring network quality as a set of latency percentiles and for application developers to describe network requirements in the same manner. We propose a formula for a distance measure between perfect and useless quality. This distance measure can calculate something that can be simplified into statements such as “A Video Conference has a 93% chance of being lag free on this network” all while making it possible to use the framework both for end-to-end test and from within.

The work proposes a minimum viable framework, and often trades precision for simplicity. The justification for this is to ensure adoption and usability in many different contexts such as active testing from applications and monitoring from network equipment. To counter the loss of precision, we require some parameters that allow for analysis of the precision.

# Proposal:

The foundation of the framework is Latency Distributions. This work will not go into detail about how to gather them, but some techniques are:

* Active probing with TWAMP Light / STAMP / IRTT
* Varying Latency Under Load Tests
* Varying Speed Tests with latency measures
* Simulating real traffic
* End-to-end measurements of real traffic
* TCP SYN ACK / DNS Lookup RTT Capture
* Estimation based on other performance data

Using Latency distributions to measure network quality is nothing new and has been proposed by various researchers/practitioners. The novelty of the Quality Attenuation metric is to view packet Loss is seen as infinite (or too late to be of use e.g. >3 seconds) latency {{TR-452.1}}.

Latency Distributions can be gathered via both passive monitoring and active testing. The active testing can use any type of IP traffic. It is OSI Layer and network technology independent, meaning it can be gathered in an end-user application, within some network equipment, or anywhere in between.

A key assumption behind the choice of latency distribution is that different applications and application categories fail at different points of the latency distribution. Some applications, typically downloads, have lenient latency requirements. Video Conferences typically are sensitive to high 90th percentile latency and to the difference between the 90th and the 99th percentile. Online gaming typically have a low tolerance for high 99th percentile latency. All applications require some level of throughput and packet loss rate. A network quality metric that aims to generalize network quality must take the latency distribution, throughput, and packet loss into consideration.

Two distributions can be composed using convolution {{TR-452.1}}.

Latency Distributions are often illustrated with Cumulative Distributions Function (CDFs). CDFs make it easy to show the share of packets that have arrived at their destination after specific amounts of time, and vica versa. Packet Loss can also be illustrated by the CDF not reaching 1 (because less than 100% of packets arrive at their destination).

# Sampling requirements:
To reach the design goal of being useful in many different contexts, this work imposes no requirement on the time period or the network loading situation. This choice has pros and cons. Latency under load is extremely important, but average or median latency has a role too. However, a network quality metric that does not take latency under load into account is bound to fail at predicting application outcome.

This framework only requires a latency distribution. If one samples latency over a time period where the network is loaded, latency under load will be part of the distribution, which is encouraged, but is not always possible, for example when passively monitoring the latency of real traffic.

One needs quite a few samples to have a statistically significant distribution and modeling a distribution may be a challenging software engineering task, hence we need to sample the latency distribution at certain percentiles. A list of 10 percentiles in a logarithmic-esque fashion has already been suggested in industry [0th, 10th, 25th, 50th, 75th, 90th, 95th, 99th, 99.9th, 100th] and seems adequate. This is all that the framework defines. Note that convolution still works with a list of percentiles, but precision takes a hit.

The framework is flexible when it comes to the direction of traffic that is being sampled, but does require that it is noted whether the latency distribution is measured one-way or two-way. The framework does not require a bandwidth measure, but requires a note on the maximal observed throughput in the time period.

By not requiring a specific number of samples, this framework allows taking 10 samples and calling it a distribution, which of course is not ideal. On the other hand, making the framework too complex and difficult to adhere to using real-world equipment and applications is the best way to ensure that this framework goes unused. Constraints will vary for different network equipment and applications.

To make sure we can trust measurements from others and analyze their precision, we require:

* Timestamp of first sample
* Duration of the sampling period
* Number of samples

* Type of measurement:

  * Cyclic (a sample every Nth ms) - Specify N
  * Bursts (X samples every Nth ms) - Specify X and N
  * Passive (observing traffic and therefore unevenly sampled)


# Describing Network Requirements:
This work builds upon the work already proposed in the Broadband Forum standard called Quality of Experience Delivered (QED/TR-452) {{TR-452.1}}. In essence, it describes network requirements as a list of percentile and latency requirement tuples. In  words: The network requirement for this app quality level/app/app category/SLA “at 4 Mbps, 90% of packets needs to arrive within 100 ms, 100% of packets needs to arrive within 200ms”. This list can be as simple as “100% of packets need to arrive within 200ms” or as long as you would like. For the sake of simplicity, the requirements percentiles must match one or more of the percentiles defined in the measurements,  i.e., one can set requirements at the [1,50,75,...] percentiles. The last specified percentile marks the acceptable packet loss. I.e. if the 99th percentile is 100, and the 99.9th or 100th percentile is not, 1% packet loss (100-99) is inferred.

Throughput or bandwidth requirements are yet to be mentioned, applications do of course have throughput requirements. With classical TCP and typical UDP flows, latency and packet loss would be enough, as they are bound to create some latency or packet loss when ramping up throughput if subsequently they become hindered by insufficient bandwidth. However, we cannot always rely on monitoring latency exclusively, as low bandwidth may give poor application outcomes without necessarily inducing a lot of latency. Therefore, the network requirements should include a minimum bandwidth requirement.

Whether the requirements are one-way or two-way must be specified.

Until now, network requirements and measurements are what is already standardized in BBF TR-452 (aka QED) framework {{TR-452.1}}. The novel part of this work is what comes next. A method for going from Network Requirements and Network Measurements to probabilities of outcomes, or Quality of Outcomes if you will.

To do that we need to make articulating the network requirements a little bit more complicated. A key design goal was to have a distance measure between perfect and useless, and have a way of quantifying what is ‘better’.

We extend the requirements (new highlighted in **bold**):

**Network Requirements for Perfection (NRP):** At 4 Mbps,  99% of packets need to arrive within 100ms, 99.9% within 200ms [implying that 0.1% packet loss is acceptable] **for the outcome to be perfect**.
**Network Requirement points of uselessness (NRPoU): If 99% of the packets have not arrived after 200ms, or 99.9% within 300ms, the outcome will be useless.**

Where the NRPoU percentiles and NRP are a required pair. I.e., if the 99.9th percentile is part of the point of uselessness then the  network requirements must also include the 99.9th percentile.

# Calculating Quality of Outcome (QoO)

At this point we have everything to calculate the quality of the application outcome. The QoO. There are 3 scenarios:

1. The network meets all the requirements for perfection. There is a 100% chance that the application will work lag-free on the network
2. The network does meet one of the points of uselessness. There is a 0% chance that the application will work on the network
3. The network does not meet NRP but is not beyond NRPoU.

1 and 2 require nothing more from the framework. For 3, we will now specify the calculation between to translate these distances to a 0 to 100 measure. We use the percentile pair where the measured latency is the closest to the NRPoU as the application is only as good as its weakest link.

Mathematically:
 QoO = min(ML, NRP, NRPoU) = (1-(ML-NRP)/(NRPoU-NRP)) * 100)

Essentially, where on the relative distance between Network Requirement for Perfection (NRP) and Network Requirement Point of Uselessness  (NRPoU) the Measured Latency (ML) lands, normalized to a percentage.

Where:
NRP is network requirements for perfection. With minimum throughput and with percentiles and milliseconds
NRPoU is the points of uselessness. With percentiles and milliseconds
i is the length of NRP / NRPoU latency per percentile requirements
ML is Measured Latency in percentiles and milliseconds
Example requirements and measured latency:
NRP: 4 Mbps {99,250},{99.9,350}
NRPoU:  {99,400},{99.9,401}
Measured Latency: .... 99p = 350ms, 99.9p = 352ms
i = 2




Then the QoO is defined:
QoO = Min(  ((1-(350 - 250)/(400-250))*100),((1-(352 - 350)/(401-350))*100))
= Min (33.33,96.08) = 33.33


In this example, we would say:
This [application/SLA/application category] has a 33% chance of being lag-free on this network

# How to find network requirements
A key advantage of having a measurement that stretches between perfect and useless, as opposed to having a binary (Good/Bad) or other low resolution (Superbad/Bad/OK/Great/Supergreat) metrics, is that we have some leeway . The leeway is useful, for instance: a lower than 20% chance of lag free experience is intuitively not good and a greater than 90% chance of lag free experience is intuitively good -  meaning we don’t have to find perfection for making the QoO metric useful.

Nevertheless we have to find some points for uselessness and perfection. There is no strict definition of when the network is so bad that the application is useless. For perfect, we may have a definition for some apps, but for apps like web browsing and gaming, lower latency is simply better. But to assist those who wish to make a requirement, we can say that if the end-user experience does not change when reducing the latency, the network quality is sufficient for the Network Requirements for Perfection (NRP) .

Someone who wishes to make a network requirement for an application in the simplest possible way, should do something along these lines.
Simulate increasing levels of latency
Observe the application and note the threshold where the application stops working perfectly
Observe the application and note the threshold where the application stops being useful at all
Someone who wishes to find sophisticated network requirements for and
Set thresholds for acceptable fps, animation fluidity, i/o latency (voice, video, actions), or other observable user-facing metrics
Create a tool for measuring these user-facing metrics
Simulate varying latency distribution with increasing levels of latency while measuring the user facing metrics.

A QoO score at 94 can be communicated as "Johns iPhone has a 94% chance of lag-free Video Conferencing", however, this does not mean that at any point of time there is a 6% chance of lag. It means there is a 6% chance of experiencing lag during the entire session/time-period, and the network requirements should be adjusted accordingly.

The reason for making the QoO metric for a session or time-period is to make it understandable for an end-user, an end-user like a should not have to relate to the time period the metric is for.
	
# An example:
Microsofts own video-conferencing service [7],[8] can be translated into the QoO Framework. For best performance for video meetings they specify 4/4 Mbps, 100 ms latency, <1% packet loss, and <30 ms jitter. This can be translated to an NRP (if we take some liberties with interpreting their jitter score);

NRP Microsoft Teams:
At minimum 4/4 Mbps.
{0p=70ms,99p=100ms}

For minimum requirements Microsoft does not specify anything, but at 500ms latency or 1000ms 99p latency, a video conference is very unlikely to work in a remotely satisfactory way.

NRPoU
{0p=500,99p=1000ms}

Of course, it is possible to specify network requirements for Teams with multiple NRP/NRPoU, for different quality levels or one/two way video and so on. Then one can calculate the QoO at each level.
….

# Summarizing the framework
Calculating QoO:
Measure latency
Build a distribution, sample it at [1p,50p,...99.9p]
Map the distribution with one or more network requirements. E.g., A network requirement for a Video Conference
If there is no overlay, the network requirement for the Video Conference is met and the likelihood of perfect application outcome is 100%. I.e., the QoO for Video Conferencing is 100
If there is an overlay. Use the probability formula to quantify the risk of non-ideal outcome.

Describing Network Requirements:
A minimum throughput
A list of percentiles and associated ms if reaches guarantees perfection (or at least no degradation because of the network)
A list of percentiles and associated ms where beyond this point, the application will be useless

This framework makes few requirements. This does of course have pros and cons. Precision vs useability and flexibility. How, and under what conditions, a latency distribution is gathered is undefined. This flexibility is key to be able to capture a large range of different types of network usage. It is also futureproof, a known problem with many existing network tests is that they define, for example, TCP as the protocol for the test. If all applications adopted BBR, the test would be rendered useless.
Example Use-Cases:
Web/Mobile App Network Test for specific application. For example, a Video Conference Service can actively test end-to-end whether the network is good enough for a lag free experience
Network Equipment Traffic Monitoring. A network equipment can passively monitor latency of real traffic and show QoOs for different types of traffic
An end-user application can test the network to check whether the QoO, or which  configurations (resolution, fps, etc.)  would be optimal
An end-user application can ask the network what latency distributions are typical, and thereby calculate their QoO
Network operators can test and monitor performance on their networks.
Before and after an equipment / software  upgrade


Comparing Apps:
On this network:
Macrohard Teams have a 93% chance of a lag free experience (Note! Made up numbers!)
Googol Meets have a 90% chance of a lag free experience (Note! Made up numbers!)
mooZ have a 88% chance of a lag-free experience (Note! Made up numbers!)


Application Category / Specific Application QoO scores as an addition to Apple Network Quality test (with responsiveness)
networkquality
==== SUMMARY ====
Upload capacity: 225.713 Mbps
Download capacity: 110.458 Mbps
Upload flows: 20
Download flows: 20
Responsiveness: High (3426 RPM)

Quality of Outcome (QoO) (probability of lag-free):
Video Conference: 100%
FaceTime: 100%
Cloud Gaming: 99%
Interactive VR: 98%

—------------------------
networkquality more
==== SUMMARY ====
Upload capacity: 225.713 Mbps
Download capacity: 110.458 Mbps
Upload flows: 20
Download flows: 20
Responsiveness: High (3426 RPM)

Quality of Outcome (QoO) (probability of lag-free):
Video Conference: 100%
FaceTime: 100%
Cloud Gaming: 99%
Interactive VR: 98%
3000 cyclic samples, start 00:08:00, end 00:08:30
0p=1ms,....,100p=21ms
// should this include probing type?

Known Weaknesses:
Missing Temporal Information in Distributions.
These two latency series: 1,200,1,200,1,200,1,200,1,200 and 1,1,1,1,1,200,200,200,200,200 Will have identical distributions, but may have different application performance. Ignoring this information is a tradeoff between simplicity and precision. To capture all information necessary to perfectly capture outcomes we are getting into extreme computational complexity. As an application's performance is bound by how the developers change to varying network performance, meaning nearly all different series of latencies may have different application outcomes. I.e., this is O(NN ) territory*. Adding another data point such as a pairwise distance measure to the network requirement will also significantly increase the difficulties of determining Network Requirements as one would need to test a huge amount of series to get it correctly.

Additionally, we cannot capture latency on every packet that is sent. We can probe and sample, but there will always be unknowns. We are now in the realm of probability. Perfection is impossible, but instead of denying this,  we should embrace it, which is why talking about the probability of outcomes is the way forward. An advantage of the QoO framework is having a single distance measure between perfect and useless as it lets us approximate probabilities in between and treating it as a probability. 99% chance of being lag free, means there is a 1% chance of lag.

*If we imagine network latency could only be either 1 or 2 ms and that no flow was more than 3 packets long, for an application to check whether without discarding any temporal information, a network requirement would have to account for all combinations of latency. I.e.,:
1,1,1 -  1,2,1 - ,1,1,2 - 1,2,2 - 2.1,1 - 2,2,1 - 2,1,2, - 2-2-2.
In big O notation: N = 1 or 2 ms = 2. M = 3 packets, 2^3=8 or O(N^M)

Assuming Linear Relationship between Perfect (and that it is not really a probability)
One can conjure up scenarios where 50ms latency is actually worse than 51ms latency as developers may have chosen 50ms as the threshold for changing quality, and the threshold may be imperfect.Taking these scenarios into account would add another magnitude of complexity to determining network requirements and finding a distance measure (between requirement and actual measured capability), and the scenarios are likely rare.

In a scenario where:
50ms p99.9 latency = 50% QoO,
51ms p99.9 latency = 49% QoO,
52ms p99.9 latency = 48% QoO
The linear relationship is assumed. This may not always hold true, but will likely be adequate.  Taking non-linear approaches into account would make creating network requirements at least one order of magnitude more difficult.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
