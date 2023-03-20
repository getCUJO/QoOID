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
 - Quality Attenuation
 - Application Outcomes
 - Quality of Outcome
 - Performance monitoring
 - Network quality
venue:
  group: "IP Performance Measurement"
  type: "Working Group"
  mail: "ippm@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/ippm/"
  github: "domoslabs/QoOID"
  latest: "https://domoslabs.github.io/QoOID/draft-olden-ippm-qoo.html"

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
  #RFC9318: # IAB Workshop report
  #RFC8290: # FQ_CoDel
  #RFC8033: # PIE
  draft-teigen-ippm-app-quality-metric-reqs:
    title: "Requirements for a Network Quality Framework Useful for Applications, Users, and Operators"
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
  #RPM:
  #  title: Responsiveness under Working Conditions
  #  target: https://datatracker.ietf.org/doc/html/draft-ietf-ippm-responsiveness
  #  date: July 2022
  #RRUL:
  #  title: Real-time response under load test specification
  #  target: https://www.bufferbloat.net/projects/bloat/wiki/RRUL_Spec/
  #Bufferbloat:
  #  title: "Bufferbloat: Dark buffers in the Internet"
  #  target: https://queue.acm.org/detail.cfm?id=2071893
  #Haeri22:
  #  title: "Mind Your Outcomes: The ΔQSD Paradigm for Quality-Centric Systems Development and Its Application to a Blockchain Case Study"
  #  target: https://www.mdpi.com/2073-431X/11/3/45


--- abstract

This document describes a new network quality framework named Quality of Outcome (QoO). The QoO framework is unique among network quality frameworks in satisfying all the requirements layed out in "Requirements for a Network Quality Framework Useful for Applications, Users and Operators".

The framework proposes a way of sampling network quality, setting network quality requirements and a formula for calculating the probability for the sampled network to satisfy network requirements.

--- middle

# Introduction
"Requirements for a Network Quality Framework Useful for Applications, Users and Operators" {{draft-teigen-ippm-app-quality-metric-reqs}} describes a set of requirements for a network quality framework. This document explores how the quality attenuation metric and framework {{TR-452.1}} can be extended to meet the full set of requirements.

Quality attenuation is a network quality metric that meets the most of the criteria set out in the requirements; it can capture the probability of a network satisfying application requirements, it is composable, and it can be compared to a variety of application requirements. The part that is yet missing is how to present quality attenuation results to end-users and application developers in an understandable way. We believe a per-application, per application-type, or per-SLA approach is appropriate here. The challenge lies in specifying how to simplify enough without losing too much in terms of precision and accuracy.

We believe the probabilistic approach is key as the network stack and application's network quality adaptation can be highly complex - especially on volatile, non-deterministic networks such as cellular radio. Applications and the underlying networking protocols makes separate optimizations based on their perceived network quality over time and saying something about an outcome with absolute certainty will be practically impossible. We can however make educated guesses on the probability of outcomes.

We propose representing network quality as minimum required throughput and set of latency percentiles. Application developers, regulatory bodies and other interested parties can describe network requirements in the same manner. We propose a formula for a distance measure between perfect and useless quality. This distance measure can, with some assumptions, calculate something that can be simplified into statements such as “A Video Conference has a 93% chance of being lag free on this network” all while making it possible to use the framework both for end-to-end test and analysis from within the network.

The work proposes a minimum viable framework, and often trades precision for simplicity. The justification for this is to ensure adoption and usability in many different contexts such as active testing from applications and monitoring from network equipment. To counter the loss of precision, we require some parameters that allow for analysis of the precision.

# Background

The foundation of the framework is Quality Attenuation {{TR-452.1}}. This work will not go into detail about how to measure Quality Attenuation, but some relevant techniques are:

* Active probing with TWAMP Light / STAMP / IRTT
* Varying Latency Under Load Tests
* Varying Speed Tests with latency measures
* Simulating real traffic
* End-to-end measurements of real traffic
* TCP SYN ACK / DNS Lookup RTT Capture
* Estimation

Quality Attenuation represents quality measurements as distributions. Using Latency distributions to measure network quality is nothing new and has been proposed by various researchers/practitioners. The novelty of the Quality Attenuation metric is to view packet loss as infinite (or too late to be of use e.g. > 3 seconds) latency {{TR-452.1}}.

Latency Distributions can be gathered via both passive monitoring and active testing. The active testing can use any type of IP traffic. It is OSI Layer and network technology independent, meaning it can be gathered in an end-user application, within some network equipment, or anywhere in between.

A key assumption behind the choice of latency distribution is that different applications and application categories fail at different points of the latency distribution. Some applications, typically downloads, have lenient latency requirements. Video Conferences typically are sensitive to high 90th percentile latency and to the difference between the 90th and the 99th percentile. Online gaming typically has a low tolerance for high 99th percentile latency. All applications require a minumum level of throughput, a maximum packet loss rate, and a maximum tolerance to variance in latency (jitter). A network quality metric that aims to generalize network quality must take the latency distribution (including jitter), throughput, and packet loss into consideration.

Two distributions can be composed using convolution {{TR-452.1}}.

# Sampling requirements
To reach the design goal of being useful in the contexts laid out in "Requirements for a Network Quality Framework Useful for Applications, Users and Operators" {{draft-teigen-ippm-app-quality-metric-reqs}}, this work imposes no requirement on the time period or the network loading situation. This choice has pros and cons. Latency under load is extremely important, but average or median latency has a role too. However, a network quality metric that does not take latency under load into account is bound to fail at predicting application outcome.

This framework only requires a latency distribution. If the sampling is done while the network is loaded, latency under load will be part of the distribution, which is encouraged, but is not always possible, for example when passively monitoring the latency of real traffic.

It takes quite a few samples to have a statistically significant distribution. Modeling a distribution may be a challenging software engineering task, hence we need to sample the latency distribution at certain percentiles. A list of 10 percentiles in a logarithmic-esque fashion has already been suggested in industry \[0th, 10th, 25th, 50th, 75th, 90th, 95th, 99th, 99.9th, 100th\] and seems adequate. We propose to define a shared set of percentile values to report.

The framework is flexible when it comes to the direction of traffic that is being sampled, but does require that it is noted whether the latency distribution is measured one-way or two-way. The framework does not require an explicit throughput measurement, but does require a note on the maximum observed throughput in the time period.

By not requiring a specific number of samples, this framework allows taking 10 samples and calling it a distribution, which of course is not ideal. On the other hand, making the framework overly complex and difficult to adhere to using real-world equipment and applications is the best way to ensure that this framework goes unused. Constraints will vary for different network equipment and applications.

To make sure we can trust measurements from others and analyze their precision, we require:

* Timestamp of first sample
* Duration of the sampling period
* Number of samples

* Type of measurement:

  * Cyclic (a sample every Nth ms) - Specify N
  * Bursts (X samples every Nth ms) - Specify X and N
  * Passive (observing traffic and therefore unevenly sampled)

By requiring the report of these variables, we ensure that the network measurements can be analyzed for precision and confidence.

# Describing Network Requirements
This work builds upon the work already proposed in the Broadband Forum standard called Quality of Experience Delivered (QED/TR-452) {{TR-452.1}}. In essence, it describes network requirements as a list of percentile and latency requirement tuples. In  other words, a network requirement may be expressed as: The network requirement for this app quality level/app/app category/SLA is “at 4 Mbps, 90% of packets needs to arrive within 100 ms, 100% of packets needs to arrive within 200ms”. This list can be as simple as “100% of packets need to arrive within 200ms” or as long as you would like. For the sake of simplicity, the requirements percentiles must match one or more of the percentiles defined in the measurements,  i.e., one can set requirements at the  \[0th, 10th, 25th, 50th, 75th, 90th, 95th, 99th, 99.9th, 100th\] percentiles. The last specified percentile marks the acceptable packet loss. I.e. if the 99th percentile is defined, and the 99.9th or 100th percentile is not, 1% packet loss (100-99) is inferred.

Applications do of course have throughput requirements. With classical TCP and typical UDP flows, latency and packet loss would be enough, as they are bound to create some latency or packet loss when ramping up throughput if subsequently they become hindered by insufficient throughput. However, we cannot always rely on monitoring latency exclusively, as low throughput may give poor application outcomes without necessarily inducing a lot of latency. Therefore, the network requirements should include a minimum throughput requirement.

Whether the requirements are one-way or two-way must be specified. If the requirement is one-way, then the requirement must make clear whether that direction is uplink (client->server) or downlink (server->client). If the requirement is two-way, then the individual uplink/downlink values of that requirement may be individually specified. 

Until now, network requirements and measurements are what is already standardized in BBF TR-452 (aka QED) framework {{TR-452.1}}. The novel part of this work is what comes next. A method for going from Network Requirements and Network Measurements to probabilities of outcomes, or Quality of Outcomes if you will.

To do that we need to make articulating the network requirements a little bit more complicated. A key design goal was to have a distance measure between perfect and useless, and have a way of quantifying what is ‘better’.

We extend the requirements to include the quality required for perfection and a quality threshold beyond which the application is considered useless.

This is named Network Requirements for Perfection (NRP). As an example: At 4 Mbps,  99% of packets need to arrive within 100ms, 99.9% within 200ms (implying that 0.1% packet loss is acceptable) for the outcome to be perfect.
Network Requirement points of uselessness (NRPoU): If 99% of the packets have not arrived after 200ms, or 99.9% within 300ms, the outcome will be useless.

Where the NRPoU percentiles and NRP are a required pair then neither should define a percentile not included in the other - i.e., if the 99.9th percentile is part of the NRPoU then the NRP must also include the 99.9th percentile.

# Calculating Quality of Outcome (QoO)

At this point we have everything we need to calculate the quality of the application outcome. The QoO. There are 3 scenarios:

1. The network meets all the requirements for perfection. There is a 100% chance that the application is not lagging because of the network
2. The network does meet one of the criteria of uselessness, including bandwidth. There is a 0% chance that the application will work because of the network
3. The network does not meet NRP but is not beyond NRPoU.

1 and 2 require nothing more from the framework. For 3, we will now specify the calculation between to translate these distances to a 0 to 100 measure. We use the percentile pair where the measured latency is the closest to the NRPoU as the application is only as good as its weakest link.

Mathematically:
 QoO = min(ML, NRP, NRPoU) = (1-(ML-NRP)/(NRPoU-NRP)) * 100)

Essentially, where on the relative distance between Network Requirement for Perfection (NRP) and Network Requirement Point of Uselessness (NRPoU) the Measured Latency (ML) lands, normalized to a percentage.

Where:
NRP is network requirements for perfection. With minimum throughput and with percentiles and milliseconds
NRPoU is the points of uselessness. With percentiles and milliseconds
i is the length of NRP / NRPoU latency per percentile requirements
ML is Measured Latency in percentiles and milliseconds

## Example requirements and measured latency:
NRP: 4 Mbps {99,250},{99.9,350}
NRPoU:  {99,400},{99.9,401}
Measured Latency: .... 99p = 350ms, 99.9p = 352ms
Measured Minumum bandwidth: 32 Mbps / 28Mbps

Then the QoO is defined:
QoO = Min(  ((1-(350 - 250)/(400-250))*100),((1-(352 - 350)/(401-350))*100))
= Min (33.33,96.08) = 33.33

In this example, we would say:
This application/SLA/application category has a 33% chance of being lag-free on this network

# How to find network requirements
A key advantage of having a measurement that stretches between perfect and useless, as opposed to having a binary (Good/Bad) or other low resolution (Superbad/Bad/OK/Great/Supergreat) metrics, is that we have some leeway. The leeway is useful, for instance: a lower than 20% chance of lag free experience is intuitively not good and a greater than 90% chance of lag free experience is intuitively good -  meaning we don’t have to find perfection for making the QoO metric useful.

Nevertheless we have to find some points for uselessness and perfection. There is no strict definition of when the network is so bad that the application is useless. For perfect, we may have a definition for some apps, but for apps like web browsing and gaming, lower latency is simply better. But to assist those who wish to make a requirement, we can say that if the end-user experience does not change when reducing the latency, the network quality is sufficient for the Network Requirements for Perfection (NRP) .

Someone who wishes to make a network requirement for an application in the simplest possible way, should do something along these lines.

* Simulate increasing levels of latency
* Observe the application and note the threshold where the application stops working perfectly
* Observe the application and note the threshold where the application stops being useful at all

Someone who wishes to find sophisticated network requirements might proceed in this way

* Set thresholds for acceptable fps, animation fluidity, i/o latency (voice, video, actions), or other metrics capturing outcomes that directly affects the user experience
* Create a tool for measuring these user-facing metrics
* Simulate varying latency distribution with increasing levels of latency while measuring the user facing metrics.

A QoO score at 94 can be communicated as "John's smartphone has a 94% chance of lag-free Video Conferencing", however, this does not mean that at any point of time there is a 6% chance of lag. It means there is a 6% chance of experiencing lag during the entire session/time-period, and the network requirements should be adjusted accordingly.

The reason for making the QoO metric for a session or time-period is to make it understandable for an end-user, an end-user should not have to relate to the time period the metric is for.

## An example
Example.com's video-conferencing service requirements can be translated into the QoO Framework. For best performance for video meetings they specify 4/4 Mbps, 100 ms latency, <1% packet loss, and <30 ms jitter. This can be translated to an NRP (if we take some liberties with interpreting their jitter score):

NRP example.com video conferencing service:
At minimum 4/4 Mbps.
{0p=70ms,99p=100ms}

For minimum requirements example.com does not specify anything, but at 500ms latency or 1000ms 99p latency, a video conference is very unlikely to work in a remotely satisfactory way.

NRPoU
{0p=500,99p=1000ms}

Of course, it is possible to specify network requirements for Teams with multiple NRP/NRPoU, for different quality levels or one/two way video and so on. Then one can calculate the QoO at each level.

# Known Weaknesses and open questions
We have described a way of simplifying how the network requirements of applications can be compared to quality attenuation measurements. The simplification introduces several artifacts that may or may not be significant. If new information emerges that indicate other tradeoffs are more fit for our purpose, we should switch before this Internet Draft moves further. In this section we discuss some known limitations.

Mobile cellular networks are known to be volatile and non-deterministic: the signal bandwidth (and as a result, throughput) can change by an order of magnitude within seconds due to radio interference and fading, cell handover,  radio scheduler load/network congestion. Whilst historic network measurements can give some indication of expected network performance for a certain location/time of day, this volatility, and the power level/load on the terminal, present a challenge in calculating the expected Quality of Outcome for a given mobile session.

An open question is how to model the expected quality variance over varying lengths of application session. For example, it may be desirable to query the network to ask the likelihood of a 30-minute video call performing acceptably, which involves network prediction (again taking into account factors such as whether the client is mobile and moving, historic data for that time,  location and application type etc.)

## Missing Temporal Information in Distributions.
These two latency series: 1,200,1,200,1,200,1,200,1,200 and 1,1,1,1,1,200,200,200,200,200 will have identical distributions, but may have different application performance. Ignoring this information is a tradeoff between simplicity and precision. To capture all information necessary to perfectly capture outcomes we are getting into extreme computational complexity. As an application's performance is bound by how the developers react to varying network performance, meaning nearly all different series of latencies may have different application outcomes.

## Subsampling the real distribution
Additionally, we cannot capture latency on every packet that is sent. We can probe and sample, but there will always be unknowns. We are now in the realm of probability. Perfection is impossible, but instead of denying this,  we should embrace it, which is why talking about the probability of outcomes is the way forward.

## Assuming Linear Relationship between Perfect and useless (and that it is not really a probability)
One can conjure up scenarios where 50ms latency is actually worse than 51ms latency as developers may have chosen 50ms as the threshold for changing quality, and the threshold may be imperfect. Taking these scenarios into account would add another magnitude of complexity to determining network requirements and finding a distance measure (between requirement and actual measured capability).

## Binary Bandwidth threshold
Choosing this is to reduce complexity, but we do acknowledge that the applications are not that simple. The defence for this trade off is that insufficient bandwidth will cause queues and therefore latency, and it should be possible to see this. Additionally, network requirements can be set up per quality level (resolution, fps etc.) for the application. However, having too many network requirements also increases the complexity for users of the framework, and it is still unclear if this is the optimal tradeoff.

## Low resolution on Packet Loss
To ensure simplicity, packet loss is described as infinite latency and the resolution will be bound to the percentiles we chose to sample. There is a good argument that some applications need higher resolution on packet loss for sufficiently describing application outcomes. If this good evidence is presented for this, packet loss should be measured separately and added to the QoO formula.

## Arbitrary selection of percentiles:
There is a need for a selection of percentiles, as we in the name of simplicity can’t use them all. But how should we select them? The 0th (minimal) and 50th (median) percentile have implicit usage by themselves. {{BITAG}} discusses that the 90th, 98th and 99th percentiles are key for some apps. In general the wisdom is that the higher percentiles are more useful for interactive applications, but only to a certain point. At this point an application sees it as packet loss and may adapt to it. Should we pick the 95th, 96th percentile, the 96.5th or the 97th? We don’t know, and as this is likely not universal across applications and applications classes, we simply have to choose arbitrarily, and to the best of our knowledge.

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
