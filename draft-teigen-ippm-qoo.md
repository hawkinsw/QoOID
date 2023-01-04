---
title: "Quality of Outcome"
abbrev: "QoO"
category: info

docname: draft-teigen-ippm-qoo-latest
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
    fullname: Bjørn Ivar Teigen
    organization: Domos
    email: bjorn@domos.no
    street: Gaustadalléen 21
    code: 0349
    country: NORWAY
 -
    fullname: Magnus Olden
    organization: Domos
    email: magnus@domos.no
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

This document aims to provide guidelines for understanding and improving network performance in a way that is easily understood by the general public. Internet performance measurements capture objective metrics about the performance of a network. Example metrics are average throughput, average latency, and the percentage of lost packets. One of the major challenges of internet performance measurement is to translate measurement results into an approximate measure of user-perceived quality (often called Quality of Experience). This is challenging because different users can have different expectations, and different applications can have very different performance needs. This document aims to describe current best practices for measuring the success or failure of networked applications in a way that generalizes to a wide range of different applications. The proposed framework captures how likely it is that user-observable outcomes, such as website loading times, are delivered in a timely and reliable manner, and provides a way to show end-users this information in an understandable way.

--- middle

# Introduction
The goal of this document is to describe network performance in a way that is useful and understandable to people. A recent IAB workshop report on measuring internet quality for end users identifies several important aspects of this problem {{RFC9318}}. Among the conclusions is the statement "A really meaningful metric for users is whether their application will work properly or fail because of a lack of a network with sufficient characteristics." We aim to document current best practices for achieving such a meaningful metric.

All networked applications rely on sequences of back-and-forth messaging to enable some functionality. The performance of the network delivering the messages affect how much time it takes to reach certain milestones in delivering the functionality. Example milestones can be completing the loading of a website, showing a frame in a video conference call, or successfully transmitting an email to the receiver. The user-perceived quality depends on timely and reliable delivery of certain user-observable milestones. Most users do not care how quickly a DNS lookup is resolved, specifically, but they do care about web-page load times (which is why DNS lookup times are very important!). A metric that intends to approximate the user-experienced network quality must therefore somehow capture how likely it is that user-observable milestones are delivered in a timely and reliable fashion.

How can we take action when our measurements show that a user-observable milestone is unlikely to be delivered quickly enough? This can more easily be answered if our metric supports composition. Composition gives us the ability to divide results into sub-results, each measuring the performance of a required sub-milestone that must be reached before the user-observable outcome can be delievered. The most intuitive way to think about composition (in our opinion) is as addition and subtraction, or alternatively as function composition. If we measure web-page load-time and find it is too slow, we may then separately measure the DNS resolution time, the TCP round-trip time and the time it takes to establish a TLS connection to get a better idea of where the problem is. If one of them stand out then perhaps there is a problem with one of the servers. However, if DNS, TCP and TLS are all too slow then it is more likely that some part of the network path which is shared by all three protocols is causing the problems. Our metric should support this kind of analysis.

To summarize, the "meaningful metric" we're looking for should have the following properties:
- Relates to user-observable outcomes such as web-page load times
- Can be composed so that the contributions of different sub-outcomes can be quantified
- Provides a way to present results to end-users in an understandable way

# Discussion of other performance metrics
Many network performance metrics have been proposed, used, and abused througout the years. We cannot name all of them here, but we will nevertheless compile a list of some of the most relevant metrics. It should not come as a surprise that we find all of them lacking for our specific purpose. That is not a condemnation of the usefulness of each of these metrics for their intended purpose. We only mean to say that we find each of them insufficient for the specific task we are aiming to perform.

For each of the metrics below we discuss, briefly, whether or not they meet each of the three criteria set out in the introduction.

*Average Peak Througphut*

Throughput relates to user-observable outcomes in the sense that there must be *enough* bandwidth available. Adding extra bandwidth above a certain threshold will at best receive diminishing returns.

Throughput cannot be composed.

Throughput is relatively well understood amongst consumers.

*Average Latency*

TODO

*99th Percentile of Latency*

TODO

*Trimmed Mean of Latency*

TODO

*Round-trips Per Minute*

Round-trips per minute {{RPM}} is a metric and test procedure specifically designed to measure delays as experienced by application-layer protocol procedures such as HTTP GET, establishing a TLS connection and DNS lookups. RPM loads the network before conducting latency measurements, and is therefore a measure of loaded latency (or working latency) well-suited to detecting bufferbloat {{Bufferbloat}}.

RPM is not composable.

RPM is designed to be easily understandable to end-users.

*Quality Attenuation*

Quality Attenuation is a network performance metric that combines latency and packet loss into a single variable {{TR-452.1}}.

Quality Attenuation relates to user-observable outcomes in the sense that user-observable outcomes can be measured using the Quality Attenuation metric directly, or the quality attenuation value describing the time-to-completion of a user-observable outcome can be computed if we know the quality attenuation of each sub-goal required to reach the desired outcome.

Quality Attenuation is composable because convolution of quality attenuation values allow us to compute the time it takes to reach specific outcomes given the quality attenuation of each sub-goal {{Haeri22}}.

Quality Attenuation is not easily understandable for end-users.

*Summary of performance metrics*

| Metric                         | Relates to end-user outcomes | Composable | Understandable        |
|--------------------------------|------------------------------|------------|-----------------------|
| Average latency                | No                           | Yes        | Yes                   |
| Average Peak Throughput        | No                           | No         | Yes                   |
| 99th Percentile of Latency     | Yes                          | No         | Yes                   |
| Trimmed mean of latency        | Yes                          | No         | Yes                   |
| Round Trips Per Minute {{RPM}} | Yes                          | No         | Yes                   |
| Quality Attenuation            | Yes                          | Yes        | No                    |
| Quality of Outcome (proposed)  | Yes                          | Yes        | Yes (Once we're done) |

# Proposed solution
This work proposes a new network quality framework that extends existing network quality metrics (i.e., responsiveness {{RPM}}, Quality Attenuation {{TR-452.1}}). We describe a framework which is useful for end-users, network operators, vendors and applications. Responsiveness does a great job for improving visibility of network quality issues beyond throughput, but is inherently about end-to-end tests and is not designed to help network operators monitor, test, and understand their networks from within. Quality Attenuation {{TR-452.1}}, on the other hand, is a great tool for understanding the performance of a network from within, but is not meant for end-users or application developers. Quality of Outcome addresses these limitations by defining how to use Quality Attenuation to approximate the probability of application success.

Quality attenuation is a network quality metric that meets the two first criteria we set out in the introduction; it relates to user-observable outcomes, and it is composable. The part that is still missing is how to present quality attenuation results to end-users and applications in an understandable way. We believe a per-application (or per application-type) approach is appropriate here. The challenge lies in how to simplify. We must specify how and when it is apporpriate to throw away information without loosing too much precision and accuracy in the results. This depends on the needs and behaviours of different applications, and therefore the IETF seems like a good place to develop this. We need feedback from a wide range of application developers, protocol designers, and other users to make sure the resulting framework is as robust and general as possible.

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
