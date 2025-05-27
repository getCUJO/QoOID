---
title: "Quality of Outcome"
abbrev: "QoO"
category: info

docname: draft-ietf-ippm-qoo-latest
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
  github: "getCUJO/QoOID"
  latest: "https://github.com/getCUJO/QoOID"

author:
 -
    fullname: Bjørn Ivar Teigen Monclair
    organization: CUJO AI
    email: bjorn.monclair@cujo.com
    street: Gaustadalléen 21
    code: 0349
    country: NORWAY
 -
    fullname: Magnus Olden
    organization: CUJO AI
    email: magnus.olden@cujo.com
    street: Gaustadalléen 21
    code: 0349
    country: NORWAY

normative:

informative:
  #RFC8290: # FQ_CoDel
  #RFC8033: # PIE
  RFC7942: # Implementation details section
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
  #RFC3393: # IP Packet Delay Variation (IPDV)
  RFC5481: # Packet Delay Variation Applicability Statement
  RFC6049: # Spatial Composition of Metrics
  RFC6390: # Guidelines for Considering New Performance Metric Development
  #RFC8033: # PIE
  #RFC8290: # FQ_CoDel
  RFC9318: # IAB Workshop report
  # SHARED:
  #   title: "The Internet is a Shared Network"
  #   author:
  #     name: "Stuart Cheshire"
  #   format:
  #     PDF: https://www.iab.org/wp-content/IAB-uploads/2021/09/draft-cheshire-internet-isshared-00b.pdf
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
  RFC5357: # TWAMP light
  RFC8762: # STAMP
  IRTT:
    title: "Isochronous Round-Trip Tester"
    target: https://github.com/heistp/irtt
  Kelly:
    title: "Networks of Queues"
    author:
      name: "Frank P. Kelly"
    target: "https://www.cambridge.org/core/journals/advances-in-applied-probability/article/abs/networks-of-queues/38A1EA868A62B09C77A073BECA1A1B56"
  RFC8239: # Refers to histogram of latency samples
  QoOSimStudy:
    title: "Quality of Outcome Simulation Study"
    author:
      name: "Bjørn Ivar Teigen Monclair"
    target: "https://github.com/getCUJO/qoosim"   # <--- Replace with actual URL
  QoOUserStudy:
    title: "Application Outcome Aware Root Cause Analysis"
    author:
      name: "Bjørn Ivar Teigen Monclair"
    target: "https://assets.ycodeapp.com/assets/app24919/Documents/LaiW4tJQ2kj4OOTiZbnf48MbS22rQHcZQmCriih9-published.pdf"

--- abstract

This document introduces the Quality of Outcome (QoO) framework, a novel
approach to network quality assessment designed to align with the needs of
application developers, users, and operators. By leveraging the Quality Attenuation
metric, QoO provides a unified method for defining and evaluating
application-specific network requirements while ensuring actionable insights
for network optimization and simple quality scores for end-users.

--- middle

# Introduction

This document introduces the Quality of Outcome network score. Quality of
Outcome is a network quality score designed to be easy to understand, while at
the same time being objective, adaptable to different network quality needs, and
allowing advanced analysis to identify the root cause of network problems. This
document defines a network requirement framework that allows application
developers to specify their network requirements, along with a way to create a simple
user-facing metric based on comparing application requirements to measurements
of network performance. The framework builds on Quality Attenuation
{{TR-452.1}}, enabling network operators to achieve fault isolation and
effective network planning through composability.

Quality attenuation is a network quality metric that meets most of the criteria
set out in the requirements; it can capture the probability of a network
satisfying application requirements, it is composable, and it can be compared to
a variety of application requirements. The part that is yet missing is how to
present quality attenuation results to end-users and application developers in
an understandable way. A per-application, per application-type, or
per-SLA approach is most appropriate here. The challenge lies in specifying how to
simplify enough without losing too much in terms of precision and accuracy.

Taking a probabilistic approach is key because the network stack and
application's network quality adaptation can be highly complex. Applications and
the underlying networking protocols make separate optimizations based on their
perceived network quality over time and saying something about an outcome with
absolute certainty is practically impossible. It is possible, however, to make
educated guesses on the probability of outcomes.

This document proposes representing network quality as a minimum required
throughput and set
of latency and loss percentiles. Application developers, regulatory bodies and
other interested parties can describe network requirements in the same manner.
This document defines a distance measure between perfect and unusable. This
distance measure can, with some assumptions, calculate something that can be
simplified into statements such as “A Video Conference has a 93% chance of being
lag free on this network” all while making it possible to use the framework both
for end-to-end test and analysis from within the network.

The work proposes a minimum viable framework, and often trades precision for
simplicity. The justification for this is to ensure adoption and usability in
many different contexts such as active testing from applications and monitoring
from network equipment. To counter the loss of precision, is it necessary to combine measurement results with a description of the measurement approach that allow
for analysis of the precision.

# Motivation

This section describes the features and attributes a network quality framework
must have to be useful for different stakeholders: application developers,
end-users, and network operators/vendors. At a high level, end-users need an
understandable network metric. Application developers require a network metric
that allows them to evaluate how well their application is likely to perform
given the measured network performance. Network operators and vendors need a
metric that facilitates troubleshooting and optimization of their networks.
Existing network quality metrics and frameworks typically address the needs of
one or two of these stakeholders, but the authors have yet to find one that
bridges the needs of all three.

A key motivation for the Quality of Outcome (QoO) framework is to bridge the
gap between the technical aspects of network performance and the practical
needs of those who depend on it. While solutions exist for many of the problems
causing high and unstable latency in the Internet, the incentives to deploy
them have remained relatively weak. A unifying framework for
assessing network quality, can serve to strengthen these incentives significantly.

Bandwidth alone is necessary but not sufficient for high-quality modern network
experiences. Idle latency, working latency, jitter, and unmitigated packet loss
are major causes of poor application outcomes. The impact of latency is widely
recognized in network engineering circles {{BITAG}}, but benchmarking the
quality of network transport remains complex. Most end-users are unable to
relate to metrics other than Mbps, which they have long been conditioned to
think of as the only dimension of network quality.

Real Time Response under load tests {{RRUL}} and Responsiveness {{RPM}} make
significant strides toward creating a network quality metric that is far closer
to application outcomes than bandwidth alone. The latter, in particular, is
successful at being relatively relatable and understandable to end-users.
However, as noted in {{RPM}}, “Our networks remain unresponsive, not from a
lack of technical solutions, but rather a lack of awareness of the problem.”
This lack of awareness means operators have little incentive to improve network
quality beyond increasing throughput. Despite the availability of open-source
solutions, vendors rarely implement them. The root cause lies in the absence of
a universally accepted network quality framework that captures how well
applications are likely to perform.

A recent IAB workshop on measuring internet quality for end-users identified a
key insight: users care primarily about application performance rather than
network performance. Among the conclusions was the statement, "A really
meaningful metric for users is whether their application will work properly or
fail because of a lack of a network with sufficient characteristics"
{{RFC9318}}. Therefore, one critical requirement of a meaningful framework is
its ability to answer the question, "Will an application work properly?"

Answering this question requires several considerations. First, the Internet is
inherently stochastic from the perspective of any given client, so certainty i
unattainable. Second, different applications have varying needs and adapt
differently to network conditions. A framework aiming to answer this question
must accommodate diverse application requirements. Third, end-users have
individual tolerances for degradation in network conditions and the resulting
effects on application experience. These variations must be factored into the
framework's design.

## Design Goal

The overall goal is to describe the requirements for an objective network
quality framework and metric that is useful for end-users, application
developers, and network operators/vendors alike.

## Requirements

This section outlines the three main requirements and their motivation.

In general, all stakeholders ultimately care about the success of applications
running over the network. Application success depends not just on bandwidth but
also on the delay of network links and computational steps involved in making
the application function. These delays depend on how the application places
load on the network, how the network is affected by environmental conditions,
and the behavior of other users sharing the network resources.

Different applications have different needs from the network, and they place
different patterns of load on it. To determine whether applications will work
well or fail, a network quality framework must compare measurements of network
performance to a wide variety of application requirements. Flexibility in
describing application requirements and the ability to capture the delay
characteristics of the network in sufficient detail are necessary to compute
application success with satisfactory accuracy and precision.

How can operators take action when measurements show that applications fail too
often? The framework must support spatial composition {{RFC6049}}, {{RFC6390}}
to answer this question. Spatial composition allows results to be divided into
sub-results, each measuring the performance of a required sub-milestone that
must be reached in time for the application to succeed.

To summarize, the framework and "meaningful metric" should
have the following properties:

1. **Capture the information necessary to compute the probability that
  applications will work well.** (Useful for end-users and application
  developers.)
2. **Compare meaningfully to different application requirements.**
3. **Compose.** Allow operators to isolate and quantify the contributions of
   different sub-outcomes and sub-paths of the network. (Useful for operators
   and vendors.)


### Requirements for end-users
The quality framework should facilitate a metric that is objective, relatable,
and relatively understandable for an end-user. A middle
ground between objective QoS metrics (Throughput, packet loss, jitter, average
latency) and subjective but understandable QoE metrics (MOS, 5-star ratings).
The ideal framework should be objective, like QoS metrics, and understandable,
like QoE metrics.

If these requirements are met, the end-user can understand if a network can
reliably deliver what they care about: the outcomes of applications. Examples
are how quickly a web page loads, the smoothness of a video conference, or
whether or not a video game has any lag.

Each end user will have an individual tolerance of session quality, below which
their quality of experience becomes personally unacceptable. However it may not
be feasible to capture and represent these tolerances _per user_ as the user
group scales. A compromise is for the quality of experience framework to place
the responsibility for sourcing and representing end-user requirements onto the
application developer. Application developers should perform user-acceptance
testing (UAT) of their application across a range of users, terminals and
network conditions to determine the terminal and network requirements that will
meet the end-user quality threshold for an acceptable subset of their end users.
Some real world examples where 'acceptable levels' have been derived by
application developers include (note: developers of similar applications may
have arrived at different figures):

* Remote music collaboration: 28ms latency note-to-ear for direct monitoring,
  <2ms jitter
* Online gaming: 6Mb/s downlink throughput and 30ms RTT to join a multiplayer
  game
* Virtual reality: <20ms RTT from head motion to rendered update in VR

Performing this UAT helps the developer understand what likelihood a new
end-user has of an acceptable Quality of Experience based on the application's
existing requirements towards the network. These requirements can evolve and
improve based on feedback from end users, and in turn better inform the
application's requirements towards the network.


### Requirements from Application and Platform Developers
The framework needs to give developers the ability to describe the network
requirements of their applications. The format for specifying network
requirements must include all relevant dimensions of network quality so that
different applications which are sensitive to different network quality
dimensions can all evaluate the network accurately. Not all
developers have network expertise, so to make it easy for developers to use
the framework, developers must be able to specify network requirements
approximately. Therefore, it must be possible to describe both simple and
complex network requirements. The framework also needs to be flexible so that it
can be used with different kinds of traffic and that extreme network
requirements which far exceed the needs of today's applications can also be
articulated.

If these requirements are met, developers of applications or platforms can state
or test their network requirements and evaluate if the network is sufficient for
a great application outcome. Both the application developers with networking
expertise and those without can use the framework.


### Requirements for Network Operators and Network Solution Vendors
From an operator perspective, the key is to have a framework that lets operators
find the network quality bottlenecks and objectively compare different networks
and technologies. The framework must support mathematically sound
compositionality ('addition' and 'subtraction') to achieve this. Why? Network
operators rarely manage network traffic end-to-end. If a test is purely
end-to-end, the ability to find bottlenecks may be gone. If, however,
measurements can be taken both end-to-end (e.g., a-b-c-d-e) and not-end-to-end (e.g., a-b-c), the results can be
subtracted to isolate the areas outside the influence of the network
operator. In other words, the network quality of a-b-c and d-e can be
separated. Compositionality is essential for fault detection and
accountability.

By having mathematically correct composition, a network operator can measure two
segments separately, perhaps even with different approaches, and add them
together to understand the end-to-end network quality.

For another example where composition is useful, look at a
typical web page load sequence. If web page load times are too slow, DNS resolution time, TCP
round-trip time, and the time it takes to establish TLS connections can be measured separately to get a
better idea of where the problem is. A network quality framework should support
this kind of analysis to be maximally useful for operators. The quality
framework must be applicable in both lab testing and monitoring of production
networks. It must be useful on different time scales, and it can't have a
dependency on network technology or OSI layer.

If these requirements are met, a network operator can monitor and test their
network and understand where the true bottlenecks are, regardless of network
technology.

# Background

The foundation of the framework is Quality Attenuation {{TR-452.1}}. This work
will not go into detail about how to measure Quality Attenuation, but some
relevant techniques are:

* Active probing with TWAMP Light {{RFC5357}} / STAMP {{RFC8762}} / IRTT
  {{IRTT}}
* Latency Under Load Tests
* Speed Tests with latency measures
* Simulating real traffic
* End-to-end measurements of real traffic
* TCP SYN ACK / DNS Lookup RTT Capture
* Estimation

Quality Attenuation represents quality measurements as distributions. Using
Latency distributions to measure network quality is nothing new and has been
proposed by various researchers/practitioners {{Kelly}}{{RFC8239}}{{RFC6049}}.
The novelty of the Quality Attenuation metric is to view packet loss as infinite
(or too late to be of use e.g. > 3 seconds) latency {{TR-452.1}}.

Latency Distributions can be gathered via both passive monitoring and active
testing. The active testing can use any type of traffic, such as TCP, UDP, or
QUIC. It is OSI Layer and network technology independent, meaning it can be
gathered in an end-user application, within some network equipment, or anywhere
in between. Passive methods rely on observing and time-stamping packets traversing the network. Examples of this include TCP SYN and SYN/ACK packets and the QUIC spin bit.

A key assumption behind the choice of latency distribution is that different
applications and application categories fail at different points of the latency
distribution. Some applications, such as downloads, have lenient latency
requirements when compared to real-time application. Video Conferences are
typically sensitive to high 90th percentile
latency and to the difference between the 90th and the 99th percentile. Online
gaming typically has a low tolerance for high 99th percentile latency. All
applications require a minumum level of throughput and a maximum packet loss
rate. A network quality metric that aims to generalize network quality must take
the latency distribution, throughput, and packet loss into consideration.

Two distributions can be composed using convolution {{TR-452.1}}.

## Discussion of other performance metrics
Numerous network performance metrics and associated frameworks have been proposed,
adopted, and, at times, misapplied over the years.
The following is a brief overview of several key network quality metrics.

Each metric is evaluated against the three criteria established in the requirements section.
Throughput is related to user-observable application outcomes because there must
be *enough* bandwidth available. Adding extra bandwidth above a certain
threshold will, at best, receive diminishing returns (and any returns are often
due to reduced latency). It is not possible to compute the probability of
application success or failure based on throughput alone for most applications.
Throughput can be compared to a variety of application requirements, but since
there is no direct correlation between throughput and application performance,
it is not possible to conclude that an application will work well even if it is
known that enough throughput is available.

Throughput cannot be composed.

### Average Latency
Average latency relates to user-observable application outcomes in the sense
that the average latency must be low enough to support a good experience.
However, it is not possible to conclude that a general application will work
well based on the fact that the average latency is good enough {{BITAG}}.

Average latency can be composed. If the average latency of links a-b and b-c is
known, then the average latency of the composition a-b-c is the sum of a-b and
b-c.

### 99th Percentile of Latency
The 99th percentile of latency relates to user-observable application outcomes
because it captures some information about how bad the tail latency is. If an
application can handle 1% of packets being too late, for instance by maintaining
a playback buffer, then the 99th percentile can be a good metric for measuring
application performance. It does not work as well for applications that are very
sensitive to overly delayed packets because the 99th percentile disregards all
information about the delays of the worst 1% of packets.

It is not possible to compose 99th-percentile values.

### Variance of latency
The variance of latency can be calculated from any collection of samples, but
network latency is not necessarily normally distributed, and so it can be
difficult to extrapolate from a measure of the variance of latency to how well
specific applications will work.

The variance of latency can be composed. If the variance of links a-b and b-c is
known, then the variance of the composition a-b-c is the sum of the variances
a-b and b-c.

### Inter-Packet Delay Variation (IPDV)
The most common definition of IPDV {{RFC5481}} measures the difference in
one-delay between subsequent packets. Some applications are very sensitive to
this because of time-outs that cause later-than-usual packets to be discarded.
For some applications, IPDV can be useful in assessing application performance,
especially when it is combined with other latency metrics. IPDV does not contain
enough information to compute the probability that a wide range of applications
will work well.

IPDV cannot be composed.

### Packet Delay Variation (PDV)
The most common definition of PDV {{RFC5481}} measures the difference in
one-delay between the smallest recorded latency and each value in a sample.

PDV cannot be composed.

### Trimmed Mean of Latency
The trimmed mean of latency is the average computed after the worst x percent of
samples have been removed. Trimmed means are typically used in cases where there
is a known rate of measurement errors that should be filtered out before
computing results.

In the case where the trimmed mean simply removes measurement errors, the result
can be composed in the same way as the average latency. In cases where the
trimmed mean removes real measurements, the trimming operation introduces errors
that may compound when composed.

### Round-trips Per Minute
Round-trips per minute {{RPM}} is a metric and test procedure specifically
designed to measure delays as experienced by application-layer protocol
procedures such as HTTP GET, establishing a TLS connection, and DNS lookups. It,
therefore, measures something very close to the user-perceived application
performance of HTTP-based applications. RPM loads the network before conducting
latency measurements and is, therefore, a measure of loaded latency (also known
as working latency) well-suited to detecting bufferbloat {{Bufferbloat}}.

RPM is not composable.

### Quality Attenuation
Quality Attenuation is a network performance metric that combines latency and
packet loss into a single variable {{TR-452.1}}.

Quality Attenuation relates to user-observable outcomes in the sense that
user-observable outcomes can be measured using the Quality Attenuation metric
directly, or the quality attenuation value describing the time-to-completion of
a user-observable outcome can be computed if the quality attenuation of
each sub-goal required to reach the desired outcome are known {{Haeri22}}.

Quality Attenuation is composable because the convolution of quality attenuation
values allows us to compute the time it takes to reach specific outcomes given
the quality attenuation of each sub-goal and the causal dependency conditions between them {{Haeri22}}.

### Summary of performance metrics

This table summarizes the properties of each of the metrics surveyed.

The column "Capture probability of general applications working well" records
whether each metric can, in principle, capture the information necessary to
compute the probability that a general application will work well, assuming
measurements capture the properties of the end-to-end network path that the
application is using.


| Metric                         | Capture probability of general applications working well | Easy to articulate Application requirements   | Composable |
|--------------------------------|----------------------------------------------------------|-----------------------------------------------|------------|
| Average latency                | Yes for some applications                                | Yes                                           | Yes        |
| Variance of latency            | No                                                       | No                                            | Yes        |
| IPDV                           | Yes for some applications                                | No                                            | No         |
| PDV                            | Yes for some applications                                | No                                            | No         |
| Average Peak Throughput        | Yes for some applications                                | Yes                                           | No         |
| 99th Percentile of Latency     | No                                                       | No                                            | No         |
| Trimmed mean of latency        | Yes for some applications                                | Yes                                           | No         |
| Round Trips Per Minute         | Yes for some applications                                | Yes                                           | No         |
| Quality Attenuation            | Yes                                                      | No                                            | Yes        |

Explanations:

- "Captures probability" refers to whether the metric captures enough
  information to compute the likelihood of an application succeeding.
- "Articulate requirements" refers to the ease with which
  application-specific requirements can be expressed using the metric.
- "Composable" means whether the metric supports mathematical composition to
  allow for detailed network analysis.

# Sampling requirements
To ensure broad applicability across diverse use cases, this framework deliberately avoids prescribing specific conditions for sampling such as fixed time intervals or defined network load levels. This flexibility enables deployment in both controlled and real-world environments.

At its core, the framework requires only a latency distribution. When measurements are taken during periods of network load, the result naturally includes latency under load. In scenarios such as passive monitoring of production traffic, capturing artificially loaded conditions may not always be feasible, whereas passively observing the actual network load may be possible.

Modeling the full latency distribution may be too complex to allow for easy adoption of the framework, and reporting latency at selected percentiles offers a practical compromise between accuracy and deployment considerations. A commonly accepted set of percentiles spanning from the 0th to the 100th in a logarithmic-like progression has been suggested by others {{BITAG}} and is recommended here:
[0th, 10th, 25th, 50th, 75th, 90th, 95th, 99th, 99.9th, 100th].

The framework is agnostic to traffic direction but mandates that measurements specify whether latency is one-way or round-trip.

Importantly, the framework does not enforce a minimum sample count. This means that even a small number of samples (e.g., 10) could technically constitute a distribution—though such cases are clearly insufficient for statistical confidence. The intent is to balance rigor with practicality, recognizing that constraints vary across devices, applications, and deployment environments.

To support reproducibility and enable confidence analysis, each measurement must be accompanied by the following metadata:

* Description of the measurement path
* Timestamp of first sample
* Total duration of the sampling period
* Number of samples collected
* Sampling method, including:
  * Cyclic: One sample every N milliseconds (specify N)
  * Burst: X samples every N milliseconds (specify X and N)
  * Passive: Opportunistic sampling of live traffic (non-uniform intervals)

These metadata elements are essential for interpreting the precision and reliability of the measurements. As demonstrated in [QoOSimStudy], low sampling frequencies and short measurement durations can lead to misleadingly optimistic or imprecise Quality of Outcome (QoO) scores.

# Describing Network Requirements
This work builds upon the work already proposed in the Broadband Forum standard
called Quality of Experience Delivered (QED/TR-452) {{TR-452.1}}, which defines
the Quality Attenuation metric. In essence, QoO
describes network requirements as a list of percentile and latency requirement
tuples. In other words, a network requirement may be expressed as: The network
requirement for this app quality level/app/app category/SLA is “at 4 Mbps, 90%
of packets needs to arrive within 100 ms, 100% of packets needs to arrive within
200ms”. This list can be as simple as “100% of packets need to arrive within
200ms” or as long as you would like. For the sake of simplicity, the
requirements percentiles must match one or more of the percentiles defined in
the measurements, i.e., one can set requirements at the \[0th, 10th, 25th, 50th,
75th, 90th, 95th, 99th, 99.9th, 100th\] percentiles. Packet loss must be
reported as a separate value.

Applications do of course have throughput requirements, and thus a complete framework for application-level network quality cannot rely on monitoring latency exclusively. Insufficient bandwidth may give poor application
outcomes without necessarily inducing a lot of latency. Therefore, the network
requirements should include a minimum throughput requirement. A fully specified
requirement can be thought of as specifying the latency and loss requirements to
be met while the end-to-end network path is loaded in a way that is at least as
demanding of the network as the application itself. This may be achieved by
running the actual application and measuring delay and loss alongside it, or by
generating artificial traffic to a level at least equivalent to the application traffic load.

Whether the requirements are one-way or two-way must be specified. Where the
requirement is one-way, the direction (uplink or downlink) must be specified. If
two-way, a decomposition into uplink and downlink measurements may be specified.

Until now, network requirements and measurements are what is already
standardized in the BBF TR-452 (aka QED) framework {{TR-452.1}}. The novel part
of this work is what comes next. A method for going from Network Requirements
and Network Measurements to probabilities of good application outcomes.

To do that it is necessary to make articulating the network requirements a little bit
more complicated. A key design goal was to have a distance measure between
perfect and unusable, and have a way of quantifying what is ‘better’.

The requirements specification is extended to include the quality required for perfection and a
quality threshold beyond which the application is considered unusable.

This is named Network Requirements for Perfection (NRP). As an example: At 4
Mbps, 99% of packets need to arrive within 100ms, 99.9% within 200ms (implying
that 0.1% packet loss is acceptable) for the outcome to be perfect. Network
Requirements Point of Unusability (NRPoU): If 99% of the packets have not
arrived after 200ms, or 99.9% within 300ms, the outcome will be unusable.

Where the NRPoU percentiles and NRP are a required pair then neither should
define a percentile not included in the other - i.e., if the 99.9th percentile
is part of the NRPoU then the NRP must also include the 99.9th percentile.

# Calculating Quality of Outcome (QoO)
The QoO metric calculates the likelihood of application success based on network performance, incorporating both latency and packet loss. There are three key scenarios:

- The network meets all the requirements for perfection. Probability of success: 100%.
- The network fails one or more criteria at the Point of Unusableness (NRPoU). Probability of success: 0%.
- The network performance falls between perfection and unusable. In this case, a
  QoO score is computed. The QoO score is calculated by taking the worst score
  derived from latency and packet loss.

Latency Component
The latency-based QoO score is computed as follows:

QoO_latency = min_{i}(min(max((1 - ((ML_i - NRP_i) / (NRPoU_i - NRP_i))) * 100, 0), 100))

Where:

- ML_i is the Measured Latency at percentile i.
- NRP_i is the Network Requirement for Perfection at percentile i.
- NRPoU_i is the Network Requirement Point of Unusableness at percentile i.

Packet Loss Component
Packet loss is considered as a separate, single measurement that applies across the entire traffic sample, not at each percentile. The packet loss score is calculated using a similar interpolation formula, but based on the total measured packet loss (MLoss) and the packet loss thresholds defined in the NRP and NRPoU:

QoO_loss = min(max((1 - ((MLoss - NRP_Loss) / (NRPoU_Loss - NRP_Loss))) * 100, 0), 100)

Where:

- MLoss is the Measured Packet Loss.
- NRP_Loss is the acceptable packet loss for perfection.
- NRPoU_Loss is the packet loss threshold beyond which the application becomes unusable.

Final QoO Calculation
The overall QoO score is the minimum of the latency and packet loss scores:

QoO = min(QoO_latency, QoO_loss)

Example Requirements and Measured Data:

- NRP: 4 Mbps {99%, 250 ms, 0.1% loss}, {99.9%, 350 ms, 0.1% loss}
- NRPoU: {99%, 400 ms, 1% loss}, {99.9%, 401 ms, 1% loss}
- Measured Latency: 99% = 350 ms, 99.9% = 352 ms
- Measured Packet Loss: 0.5%
- Measured Minimum Bandwidth: 32 Mbps / 28 Mbps

Then the QoO is defined:

QoO_latency
= min(
(min(max((1 - (350 ms - 250 ms) / (400 ms - 250 ms)) * 100, 0), 100),
(min(max((1 - (352 ms - 350 ms) / (401 ms - 350 ms)) * 100, 0), 100))
)
= min(33.33, 96.08)
= 33.33

QoO_loss
= min(max((1 - (0.5% - 0.1%) / (1% - 0.1%)) * 100, 0), 100)
= 55.56

Finally, the overall QoO score is:

QoO = min(33.33, 55.56)
= 33.33

In this example, the application has a 33% chance of meeting the quality expectations on this network, considering both latency and packet loss.

**Implementation Note: Sensitivity to Sampling Accuracy**

Based on the simulation results in [QoOSimStudy], overly noisy or inaccurate
latency samples can artificially inflate worst-case percentiles, thereby driving
QoO scores lower than actual network conditions would warrant. Conversely,
coarse measurement intervals can miss short-lived spikes entirely, resulting in
an inflated QoO. Users of this framework should consider hardware/software
measurement jitter, clock offset, or other system-level noise sources when
collecting data, and configure sampling frequency and duration to suit their
specific needs.

# How to find network requirements
A key advantage of a measurement that spans the range from perfect to unusable, rather than relying on binary (Good/Bad) or other low-resolution (Terrible/Bad/OK/Great/Excellent) metrics, is the flexibility it provides. For example, a chance of lag-free experience below 20% is intuitively undesirable, while a chance above 90% is intuitively favorable—demonstrating that absolute perfection is not required for the QoO metric to be meaningful.

However, it remains necessary to define points representing unusableness and perfection. There is no universally strict threshold at which network conditions render an application unusable. For perfection, some applications may have clear definitions, but for others, such as web browsing and gaming, lower latency is always preferable. To assist in establishing requirements, it is recommended that the Network Requirements for Perfection (NRP) be set at the point where further reductions in latency do not result in a perceivable improvement in end-user experience.

Someone who wishes to make a network requirement for an application in the
simplest possible way, should do something along these lines.

* Simulate increasing levels of latency
* Observe the application and note the threshold where the application stops
  working perfectly
* Observe the application and note the threshold where the application stops
  being useful at all

Someone who wishes to find sophisticated network requirements might proceed in
this way

* Set thresholds for acceptable fps, animation fluidity, i/o latency (voice,
  video, actions), or other metrics capturing outcomes that directly affects the
  user experience
* Create a tool for measuring these user-facing metrics
* Simulate varying latency distribution with increasing levels of latency while
  measuring the user facing metrics.

A QoO score at 94 can be communicated as "John's smartphone has a 94% chance of
lag-free Video Conferencing", however, this does not mean that at any point of
time there is a 6% chance of lag. It means there is a 6% chance of experiencing
lag during the entire session/time-period, and the network requirements should
be adjusted accordingly.

The reason for making the QoO metric for a session is to make it understandable
for an end-user, an end-user should not have to relate to the time period the
metric is for.

## An example
Example.com's video-conferencing service requirements can be translated into the
QoO Framework. For best performance for video meetings, they specify 4/4 Mbps,
100 ms latency, <1% packet loss, and <30 ms jitter. This can be translated to an
NRP:

NRP example.com video conferencing service: At minimum 4/4 Mbps.
{0p=70ms,99p=100ms}

For minimum requirements example.com does not specify anything, but at 500ms
latency or 1000ms 99p latency, a video conference is very unlikely to work in a
remotely satisfactory way.

NRPoU {0p=500,99p=1000ms}

Of course, it is possible to specify network requirements for Example.com with
multiple NRP/NRPoU, for different quality levels, one/two way video, and so on.
Then one can calculate the QoO at each level.

# Insights from Simulation Results  {#simulation-insights}

While the QoO framework itself places no strict requirement on sampling patterns
or measurement technology, a recent simulation study {{QoOSimStudy}} examined the
metric’s real-world applicability under varying conditions of:

1. **Sampling Frequency**: Slow sampling rates (e.g., <1 Hz) risk missing rare,
   short-lived latency spikes, resulting in overly optimistic QoO scores.
2. **Measurement Noise**: Measurement errors on the same scale as the thresholds
   (NRP, NRPoU) can distort high-percentile latencies and cause artificially lower QoO.
3. **Requirement Specification**: Slightly adjusting the latency thresholds or
   target percentiles can cause significant changes in QoO, especially when the
   measurement distribution is near a threshold.
4. **Measurement Duration**: Shorter tests with sparse sampling tend to
   underestimate worst-case behavior for heavy-tailed latency distributions,
   biasing QoO in a positive direction.

In practice, these findings mean:

- Calibrate the combination of sampling rate and total measurement period to
  capture fat-tailed distributions of latency with sufficient accuracy.
- Avoid significant measurement noise where possible (e.g., by calibrating time
  sources, accounting for clock drift).
- Thorougly test application requirement thresholds so that the resulting QoO
  scores accurately reflect application performance.

These guidelines are *non-normative* but reflect empirical evidence on how QoO
performs.

# Insights from user testing {#user-testing}

A study involving 25 participants tested the Quality of Outcome (QoO) framework
in a real-world settings {{QoOUserStudy}}. Participants used specially equipped
routers in their homes for 10 days, providing both network performance data and
feedback through pre- and post-trial surveys.

Participants found QoO metrics more intuitive and
actionable than traditional metrics (e.g., speed tests). QoO directly aligned
with their self-reported experiences, increasing trust and engagement.

These results provide supporting evidence for QoO's value as a user-focused
tool, bridging technical metrics with real-world application performance to enhance end-user satisfaction.

# Known Weaknesses and open questions
A method has been described for simplifying the comparison between application network requirements and quality attenuation measurements. This simplification introduces several artifacts, the significance of which may vary depending on context. The following section discusses some known limitations.

Volatile networks - in particular, mobile cellular networks - pose a challenge
for network quality prediction, with the level of assurance of the prediction
likely to decrease as session duration increases. Historic network conditions
for a given cell may help indicate times of network load or reduced transmission
power, and their effect on throughput/latency/loss. However: as terminals are
mobile, the signal bandwidth available to a given terminal can change by an
order of magnitude within seconds due to physical radio factors. These include
whether the terminal is at the edge of cell, or undergoing cell handover, the
interference and fading from the local environment, and any switch between radio
bearers with differing signal bandwidth and transmission-time intervals (e.g. 4G
and 5G). This suggests a requirement for measuring quality attenuation to and
from an individual terminal, as that can account for the factors described
above. How that facility is provisioned onto indiviudal terminals, and how
terminal-hosted applications can trigger a quality attenuation query, is an open
question.

## Missing Temporal Information in Distributions.
These two latency series: 1,200,1,200,1,200,1,200,1,200 and
1,1,1,1,1,200,200,200,200,200 will have identical distributions, but may have
different application performance. Ignoring this information is a tradeoff
between simplicity and precision. To capture all information necessary to
perfectly capture outcomes quickly gets into extreme levels of overhead and high computational complexity.
An application's performance depends on reactions to varying
network performance, meaning nearly all different series of latencies may have
different application outcomes.

## Subsampling the real distribution
Additionally, it is not feasible to capture latency for every packet transmitted. Probing and sampling can be performed, but some aspects will always remain unknown. This introduces an element of probability. Absolute perfection cannot be achieved; rather than disregarding this reality, it is more practical to acknowledge it. Therefore, discussing the probability of outcomes provides a more accurate and meaningful approach.

## Assuming Linear Relationship between Perfect and Unusable (and that it is not really a probability)
One can conjure up scenarios where 50ms latency is actually worse than 51ms
latency as developers may have chosen 50ms as the threshold for changing
quality, and the threshold may be imperfect. Taking these scenarios into account
would add another magnitude of complexity to determining network requirements
and finding a distance measure (between requirement and actual measured
capability).

## Binary Bandwidth threshold
Choosing this is to reduce complexity, but it must be acknowledged that the
applications are not that simple. Network requirements can be set up per
quality level (resolution, fps etc.) for the application if necessary.

## Arbitrary selection of percentiles
A selection of percentiles is necessary for simplicity, because more complex methods may slow adoption of the framework. The 0th (minimum) and 50th (median) percentiles are commonly used for their inherent significance. According to {{BITAG}}, the 90th, 98th, and 99th percentiles are particularly important for certain applications. Generally, higher percentiles provide more insight for interactive applications, but only up to a certain threshold—beyond which applications may treat excessive delays as packet loss and adapt accordingly. The choice between percentiles such as the 95th, 96th, 96.5th, or 97th is not universally prescribed and may vary between application types. Therefore, percentiles must be selected arbitrarily, based on the best available knowledge and the intended use case.

# Implementation status
Note to RFC Editor: This section MUST be removed before publication of the
document.

This section records the status of known implementations of the protocol defined
by this specification at the time of posting of this Internet-Draft, and is
based on a proposal described in [RFC7942]. The description of implementations
in this section is intended to assist the IETF in its decision processes in
progressing drafts to RFCs. Please note that the listing of any individual
implementation here does not imply endorsement by the IETF. Furthermore, no
effort has been spent to verify the information presented here that was supplied
by IETF contributors. This is not intended as, and must not be construed to be,
a catalog of available implementations or their features. Readers are advised to
note that other implementations may exist.

According to [RFC7942], "this will allow reviewers and working groups to assign
due consideration to documents that have the benefit of running code, which may
serve as evidence of valuable experimentation and feedback that have made the
implemented protocols more mature. It is up to the individual working groups to
use this information as they see fit".

## qoo-c

* Link to the open-source repository:

  https://github.com/getCujo/qoo-c

* The organization responsible for the implementation:

  CUJO AI

* A brief general description:

  A C library for calculating Quality of Outcome

* The implementation's level of maturity:

  A complete implentation of the specification described in this document

* Coverage:

  The library is tested with unit tests

* Licensing:

  GPL 2.0

* Implementation experience:

  Tested by the author. Needs additional testing by third parties.

* Contact information:

  Bjørn Ivar Teigen: bjorn@domos.no

* The date when information about this particular implementation was last
updated:

  10th of January 2024

## goresponsiveness

* Link to the open-source repository:

  https://github.com/network-quality/goresponsiveness

  The specific pull-request:
  https://github.com/network-quality/goresponsiveness/pull/56

* The organization responsible for the implementation:

  University of Cincinatti for goresponsiveness as a whole, Domos for the QoO
  part.

* A brief general description:

  A network quality test written in Go. Capable of measuring RPM and QoO.

* The implementation's level of maturity:

  In active development

* Coverage:

  The QoO part is tested with unit tests

* Licensing:

  GPL 2.0

* Implementation experience:

  Needs testing by third parties

* Contact information:

  Bjørn Ivar Teigen: bjorn.monclair@cujo.com

  William Hawkins III: hawkinwh@ucmail.uc.edu

* The date when information about this particular implementation was last
updated:

  10th of January 2024

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
