---
title: A Framework for Computing-Aware Traffic Steering (CATS)
abbrev: CATS Framework
category: info

docname: draft-ldbc-cats-framework-latest
submissionType: IETF
number:
date:
consensus: true
v: 3
area: Routing area
workgroup: cats
keyword:
 - User Experience
 - Collaborative Networking
 - Service optimization

coding: us-ascii
pi: [toc, sortrefs, symrefs]

author:
 -
  ins: C. Li
  name: Cheng Li
  organization: Huawei Technologies
  email: c.l@huawei.com
  role: editor
  country: China
 -
  ins: Z. Du
  name: Zongpeng Du
  organization: China Mobile
  email: duzongpeng@chinamobile.com
  country: China
 -
    fullname: Mohamed Boucadair
    organization: Orange
    role: editor
    country: France
    email: mohamed.boucadair@orange.com
 -
   fullname: Luis M. Contreras
   organization: Telefonica
   country: Spain
   email: luismiguel.contrerasmurillo@telefonica.com
 -
  ins: J. Drake
  name: John E Drake
  organization: Juniper Networks, Inc.
  email: jdrake@juniper.net
  country: United States of America
 -
  ins: G. Huang
  name: Guangping Huang
  organization: ZTE
  email: huang.guangping@zte.com.cn
  country: China
 -
  ins: G. Mishra
  name: Gyan Mishra
  organization: Verizon Inc.
  email: hayabusagsm@gmail.com
  country: United States of America


contributor:
 -
  name: Huijuan Yao
  org: China Mobile
  email: yaohuijuan@chinamobile.com
 -
  name: Yizhou Li
  org: Huawei Technologies
  email: liyizhou@huawei.com
 -
  name: Dirk Trossen
  org: Huawei Technologies
  email: dirk.trossen@huawei.com
 -
  name: Luigi Iannone
  org: Huawei Technologies
  email: luigi.iannone@huawei.com
 -
  name: Hang Shi
  org: Huawei Technologies
  email: shihang9@huawei.com
 -
  ins: C. Lin
  name: Changwang Lin
  organization: New H3C Technologies
  email: linchangwang.04414@h3c.com
 -
  ins: X. Wang
  name: Xueshun Wang
  organization: CICT
  email: xswang@fiberhome.com
 -
  ins: X. Wang
  name: Xuewei Wang
  organization: Ruijie Networks
  email: wangxuewei1@ruijie.com.cn
 -
  ins: C. Jacquenet
  name: Christian Jacquenet
  organization: Orange
  email: christian.jacquenet@orange.com

normative:

informative:


--- abstract

This document describes a framework for Computing-Aware Traffic Steering (CATS). Particularly, the document identifies a set of CATS components, describes their interactions, and exemplifies the workflow of the control and data planes.

--- middle

# Introduction

Computing service architectures have been expanding from single service site to multiple, sometimes collaborative, service sites to address various issues (e.g., long response times or suboptimal service and network resource usage).

The underlying networking infrastructures that include computing resources usually provide relatively static service dispatching (that is, the selection of the sevice instances that will be invoked for a request). In such infrastructures, service-specific traffic is often directed to the closest service site from a routing perspective without considering the actual network state (e.g., traffic congestion conditions).

As described in {{?I-D.ietf-cats-usecases-requirements}}, traffic steering that takes into account computing resource metrics would benefit several services, including latency-sensitive service like immersive services that rely upon the use of augmented reality or virtual reality (AR/VR) techniques. This document provides an architectural framework that aims at facilitating the making of compute- and network-aware traffic steering decisions in networking environments where computing service resources are deployed.

The Computing-Aware Traffic Steering (CATS) framework assumes that there may be multiple service instances that are providing one given service. Each of these service instances can be accessed via a service contact instance running on different service sites. A single service site may have limited computing resources available at a given time, whereas the various service sites may experience different resource availability issues over time. A single service site may host one or multiple service contact instances.

Steering in CATS is about selecting the appropriate service contact instance that will service a request according to a set of network and computing metrics.
That selection may not necessarily reveal the actual service instance that will be invoked, e.g., in hierarchical or recursive contexts.
The metrics are aggerated; that is, they reflect the collective resources involved in a service instance.

The CATS framework is an overlay framework for the selection of the suitable service contact instance(s) from a set of candidates. The exact characterization of 'suitable' is determined by a combination of networking and computing metrics.

Also, this document describes a workflow of the main CATS procedures that are executed in both the control and data planes.


# Terminology

This document makes use of the following terms:

Client:
: An endpoint that is connected to a service provider network.

Computing-Aware Traffic Steering (CATS):
 :  A traffic engineering approach {{?I-D.ietf-teas-rfc3272bis}} that takes into account the dynamic nature of computing resources and network state to optimize service-specific traffic forwarding towards a given service contact instance. Various relevant metrics may be used to enforce such computing-aware traffic steering policies.

CATS Service ID (CS-ID):
 : An identifier representing a service, which the clients use to access it. See {{cats-ids}}.

CATS Instance Selector ID (CIS-ID):
 : An identifier of a specific service contact instance. See {{cats-ids}}.

Service:
  : An offering that is made available by a provider by orchestrating a set of resources (networking, compute, storage, etc.).
  : Which and how these resources are solicited is part of the service logic which is internal to the provider. For example, these resources may be:

    * Exposed by one or multiple processes (a.k.a. Service Functions (SFs) {{?RFC7665}}).
    * Provided by virtual instances, physical, or a combination thereof.
    * Hosted within the same or distinct nodes.
    * Hosted within the same or multiple service sites.
    * Chained to provide a service using a variety of means.

  : How a service is structured is out of the scope of CATS.
  : The same service can be provided in many locations; each of them constitutes a service instance.

Computing Service:
  : An offering is made available by a provider by orchestrating a set of computing resources (without networking resources).

Service instance:
  : An instance of running resources according to a given service logic.
  : Many such instances can be enabled by a provider. Instances that adhere to the same service logic provide the same service.
  : An instance is typically running in a service site. Clients’ requests are serviced by one of these instances.

Service site:
 : A location that hosts the resources that are required to offer a service.
 : A service site may be a node or a set of nodes.
 : A CATS-serviced site is a service site that is connected to a CATS-Forwarder.

Service contact instance:
  : A client-facing service function instance that is responsible for receiving requests in the context of a given service. A service request is processed according to the service logic (e.g., handle locally or solicit backend resources). Steering beyond the service contact instance is hidden to both clients and CATS components.
  : a service contact instance is reachable via at least one Egress CATS Forwarder.
  : A service can be accessed via multiple service contact instances running at the same or different locations (service sites).
  : The same service contact instance may dispatch service requests to one or more service instances (e.g., an instance that behaves as a service load-balancer).

Computing-aware forwarding (or steering, computing):
  :  A forwarding (or steeting, computing) scheme which takes as input a set of metrics that reflect the capabilities and state of computing resources.

Service request:
 : A request to access or invoke a specific service. Such a request is steered to a service contact instance via CATS-Forwarders.
 : A service request is placed using service-specific protocols.
 : Service requests are not explicitly sent by clients to CATS-Forwarders.

CATS-Forwarder:
 : A network entity that makes forwarding decisions based on CATS information to steer traffic specific to a  service request towards a corresponding yet selected service contact instance. The selection of a service contact instance relies upon a multi-metric path computation.
 : A CATS-Forwarder may behave as Ingress or Egress CATS-Forwarder.

Ingress CATS-Forwarder:
 : An entity that steers service-specific traffic along a CATS-computed path that leads to an Egress CATS-Forwarder that connects to the most suitable service site that host the service contact instance selected to satisfy the initial service request.

Egress CATS-Forwarder:
: An entity that is located at the end of a CATS-computed path and which connects to a CATS-serviced site.

CATS Path Selector (C-PS):
 : A computation logic that calculates and selects paths towards service locations and instances and which accommodates the requirements of service requests. Such a path computation engine takes into account the service and network status information. See {{sec-cps}}.

CATS Service Metric Agent (C-SMA):
 : A functional entity that is responsible for collecting service capabilities and status, and for reporting them to a CATS Path Selector (C-PS). See {{sec-csma}}.

CATS Network Metric Agent (C-NMA):
  :  A functional entity that is responsible for collecting network capabilities and status, and for reporting them to a C-PS. See {{sec-cnma}}.

CATS Traffic Classifier (C-TC):
 : A functional entity that is responsible for determining which packets belong to a traffic flow for a particular service request. It is also responsible for forwarding such packets along a C-PS computed path that leads to the relevant service contact instance. See {{sec-ctc}}.

# Framework and Components {#Framework-and-concepts}

## Assumptions

CATS assumes that there are multiple service instances running on different service sites, and which provide a given service that is represented by the same service identifier (see {{cats-ids}}). However, CATS does not make any assumption about these instances other than they are reachable via one or multiple service contact instances.

## CATS Identifiers {#cats-ids}

CATS uses the following identifiers:

CATS Service ID (CS-ID):
  : An identifier representing a service, which the clients use to access it. Such an ID identifies all the instances of a given service, regardless of their location.
  : The CS-ID is independent of which service contact instance serves the service request.
  : Service requests are spread over the service contact instances that can accommodate them, considering the location of the initiator of the service request and the availability (in terms of resource/traffic load, for example) of the service instances resource-wise among other considerations like traffic congestion conditions.

CATS Instance Selector ID (CIS-ID):
: An identifier of a specific service contact instance.

## CATS Components {#sec-cat-arch}

The network nodes make forwarding decisions for a given service request that has been received from a client according to the capabilities and status information of both service instances and network. The main CATS functional elements and their interactions are shown in {{fig-cats-fw}}.

~~~ aasvg
    +-----+              +------+           +------+
  +------+|            +------+ |         +------+ |
  |client|+            |client|-+         |client|-+
  +---+--+             +---+--+           +---+--+
      |                    |                  |
      | +----------------+ |            +-----+----------+
      +-+    C-TC#1      +-+      +-----+    C-TC#2      |
        |----------------|        |     |----------------|
        |     |C-PS#1    |    +------+  |CATS-Forwarder 4|
  ......|     +----------|....|C-PS#2|..|                |...
  :     |CATS-Forwarder 2|    |      |  |                |  .
  :     +----------------+    +------+  +----------------+  :
  :                                                         :
  :                                            +-------+    :
  :                         Underlay           | C-NMA |    :
  :                      Infrastructure        +-------+    :
  :                                                         :
  :                                                         :
  : +----------------+                +----------------+    :
  : |CATS-Forwarder 1|  +-------+     |CATS-Forwarder 3|    :
  :.|                |..|C-SMA#1|.... |                |....:
    +---------+------+  +-------+     +----------------+
              |         |             |   C-SMA#2      |
              |         |             +-------+--------+
              |         |                     |
              |         |                     |
           +------------+               +------------+
          +------------+ |             +------------+ |
          |  Service   | |             |  Service   | |
          |  Contact   | |             |  Contact   | |
          |  Instance  |-+             |  Instance  |-+
          +------------+               +------------+
           service site 1              service site 2
~~~
{: #fig-cats-fw title="CATS Functional Components"}

### Service Sites and Services Instances {#sec-service-sites}

Service sites are the premises that host a set of computing resources. As mentioned in {{cats-ids}}, a compute service (e.g., for face recognition purposes or a game server) is uniquely identified by a CATS Service IDentifier (CS-ID).

Service instances can be instantiated and accessed through different service sites so that a single service can be represented and accessed by several contact instances that run in different regions of a network.

{{fig-cats-fw}} shows two CATS nodes ("CATS-Forwarder 1" and "CATS-Forwarder 3") that provide access to service contact instances. These nodes behave as Egress CATS-Forwarders ({{sec-ocr}}).

> Note: "Egress" is used here in reference to the direction of the service request placement. The directionality is called to explicitly identify the exit node of the CATS infrastructure.

### CATS Service Metric Agent (C-SMA) {#sec-csma}

The CATS Service Metric Agent (C-SMA) is a functional component that gathers information about service sites and server resources, as well as the status of the different service instances. The C-SMAs are located adjacent to the service contact instances and may be hosted by the Egress CATS-Forwarders ({{sec-ocr}}) or located next to them.

{{fig-cats-fw}} shows one C-SMA embedded in "CATS-Forwarder 3", and another C-SMA that is adjacent to "CATS-Forwarder 1".

### The CATS Network Metric Agent (C-NMA) {#sec-cnma}

The CATS Network Metric Agent (C-NMA) is a functional component that gathers information about the state of the underlay network. The C-NMAs may be implemented as standalone components or may be hosted by other components, such as CATS-Forwarders or CATS Path Selectors (C-PS) ({{sec-cps}}).

{{fig-cats-fw}} shows a single, standalone C-NMA within the underlay network. There may be one or more C-NMAs for an underlay network.

### CATS Path Selector (C-PS) {#sec-cps}

The C-SMAs and C-NMAs share the collected information with CATS Path Selectors (C-PSes) that use such information to select the Egress CATS-Forwarders (and potentially the service contact instances) where to forward traffic for a given service request. C-PSes also determine the best paths (possibly using tunnels) to forward traffic, according to various criteria that include network state and traffic congestion conditions. The collected information is encoded into one or more metrics that feed the C-PS path computation logic. Such an information also includes CS-ID and possibly CIS-IDs.

There might be one or more C-PSes used to compute CATS paths in a CATS infrastructure.

A CS-PS can be integrated into CATS-Forwarders (e.g., "C-PS#1" in {{fig-cats-fw}}) or may be deployed as a standalone component (e.g., "C-PS#2" in {{fig-cats-fw}}).

### CATS Traffic Classifier (C-TC) {#sec-ctc}

CATS Traffic Classifier (C-TC) is a functional component that is responsible for associating incoming packets from clients with existing service requests. CATS classifiers also ensure that packets that are bound to a specific service contact instance are all forwarded along the same path that leads to the same service contact instance, as instructed by a C-PS.

CATS classifiers are typically hosted in CATS-Forwarders.

### Overlay CATS-Forwarders {#sec-ocr}

The Egress CATS-Forwarders are the endpoints that behave as an overlay egress for service requests that are forwarded over a CATS infrastructure. A service site that hosts service instances may be connected to one or more Egress CATS-Forwarders (that is, multi-homing is of course a design option). If a C-PS has selected a specific service contact instance and the C-TC has marked the traffic with the CIS-ID, the Egress CATS-Forwarder then forwards traffic to the relevant service contact instance. In some cases, the choice of the service  contact instance may be left open to the Egress CATS-Forwarder (i.e., traffic is marked only with the CS-ID). In such cases, the Egress CATS-Forwarder selects a service contact instance using its knowledge of service and network capabilities as well as the current load as observed by the CATS-Forwarder, among other considerations. Absent explicit policy, an Egress CATS-Forwarder must make sure to forward all packets that pertain to a given service request towards the same service contact instance.

Note that, depending on the design considerations and service requirements, per-service  contact instance computing-related metrics or aggregated per-site computing related metrics (and a combination thereof) can be used by a C-PS. Using aggregated per-site computing related metrics appears as a privileged option scalability-wise, but relies on Egress CATS-Forwarders that connect to various service  contact instances to select the proper service  contact instance.

### Underlay Infrastructure

The "underlay infrastructure" in {{fig-cats-fw}} indicates an IP/MPLS network that is not necessarily CATS-aware. The CATS paths that are computed by a P-CS will be distributed among the overlay CATS-Forwarders ({{sec-ocr}}), and will not affect the underlay nodes.

A CATS implementation may rely upon a control or management plane to distribute service metrics and network metrics - this document does not define a specific solution.

## Deployment Considerations

This document does not make any assumption about how the various CATS functional elements are implemented and deployed. Concretely, whether a CATS deployment follows a fully distributed design or relies upon a mix of centralized (e.g., a C-PS) and distributed CATS functions (e.g., CATS traffic classifiers) is deployment-specific and may reflect the savoir-faire of the (CATS) service provider.

Centralized designs where the computing related metrics from the C-SMAs are collected by a (logically) centralized path computation logic (e.g., a Path Computation Element (PCE) {{?RFC4655}}) that also collects network metrics may be adopted. In the latter case, the CATS computation logic may process incoming service requests to compute and select paths and, therefore, service contact instances. The outcomes of such a computation process may then be communicated to CATS traffic classifiers (C-TCs).

# CATS Framework Workflow

The following subsections provide an overview of how the CATS workflow operates assuming a distributed CATS design.

## Provisioning of CATS Components

TBC: --detail required provisioning at CAST elements (booptsrapping, credentials of peer CAST nodes, services, optimization metrics per service, etc.)--

## Service Announcement

A service is associated with a unique identifier called a CS-ID. A CS-ID may be a network identifier, such as an IP address. The mapping of CS-IDs to network identifiers may be learned through a name resolution service, such as DNS {{?RFC1034}}.

## Metrics Distribution

As described in {{sec-cat-arch}}, a C-SMA collects both service-related capabilities and metrics, and associates them with a CS-ID that identifies the service. The C-SMA may aggregate the metrics for multiple service  contact  instances, or maintain them separately or both. The C-SMA then advertises the CS-IDs along with the metrics to be received by all C-PSes in the network. The service metrics include computing-related metrics and potentially other service-specific metrics like the number of end-users who access the service  contact  instance at any given time, their location, etc.

Computing metrics may change very frequently (see {{?I-D.ietf-cats-usecases-requirements}} for a discussion). How frequently such information is distributed is to be determined as part of the specification of any communication protocol (including routing protocols) that may be used to distribute the information. Various options can be considered, such as (but not limited to) interval-based updates, threshold-triggered updates, or policy-based updates.

Additionally, the C-NMA collects network-related capabilities and metrics. These may be collected and distributed by existing routing protocols, although extensions to such protocols may be required to carry additional information (e.g., link latency). The C-NMA distributes the network metrics to the C-PSes so that they can use the combination of service and network metrics to determine the best Egress CATS-Forwarder to provide access to a service contact instance and invoke the compute function required by a service request.

Network metrics may also change over time. Dynamic routing protocols may take advantage of some information or capabilities to prevent the network from being flooded with state change information (e.g., Partial Route Computation (PRC) of OSPFv3 {{?RFC5340}}). C-NMAs should also be configured or instructed like C-SMAs to determine when and how often updates should be notified to the C-PSes.

{{fig-cats-example-overlay}} shows an example of how CATS metrics can be distributed. There is a client attached to the netowrk via "CATS-Forwarder 1". There are three instances of the service with CS-ID "1": two are located at "Service Site 2" attached via "CATS-Forwarder 2" and have CIS-IDs "1" and "2"; the third service contact instance is located at "Service Site 3" attached via "CATS-Forwarder 3" and with CIS-ID "3". There is also a second service with CS-ID "2" with only one service contact instance located at "Service Site 2".

In {{fig-cats-example-overlay}}, the C-SMA collocated with "CATS-Forwarder 2" distributes the service metrics for both service contact instances (i.e., (CS-ID 1, CIS-ID 1) and (CS-ID 1, CIS-ID 2)). Note that this information may be aggregated into a single advertisement, but in this case, the metrics for each service contact instance are indicated separately. Similarly, the C-SMA agent located at "Service Site 2" advertises the service metrics for the two services hosted by "Service Site 2".

The service metric advertisements are processed by the C-PS hosted by "CATS-Forwarder 1". The C-PS also processes network metric advertisements sent by the C-NMA. All metrics are used by the C-PS to compute and select the most relevant path that leads to the Egress CATS-Forwarder according to the initial  client's service request, the service that is requested ("CS-ID 1" or "CS-ID 2"), the state of the service contact instances as reported by the metrics, and the state of the network.

~~~ aasvg
          Service CS-ID 1, instance CIS-ID 1 <metrics>
          Service CS-ID 1, instance CIS-ID 2 <metrics>

                 :<----------------------:
                 :                       :              +--------+
                 :                       :              |CS-ID 1 |
                 :                       :           +--|CIS-ID 1|
                 :              +----------------+    |  +--------+
                 :              |    C-SMA       |----|   Service Site 2
                 :              +----------------+    |  +--------+
                 :              |CATS-Forwarder 2|    +--|CS-ID 1 |
                 :              +----------------+       |CIS-ID 2|
 +--------+      :                        |             +--------+
 | Client |      :  Network +----------------------+
 +--------+      :  metrics | +-------+            |
      |          : :<---------| C-NMA |            |
      |          : :        | +-------+            |
 +---------------------+    |                      |
 |CATS-Forwarder 1|C-PS|----|                      |
 +---------------------+    |       Underlay       |
                 :          |     Infrastructure   |     +--------+
                 :          |                      |     |CS-ID 1 |
                 :          +----------------------+ +---|CIS-ID 3|
                 :                    |              |   +--------+
                 :          +----------------+  +-------+
                 :          |CATS-Forwarder 3|--| C-SMA | Service Site 3
                 :          +----------------+  +-------+
                 :                                :  |   +-------+
                 :                                :  +---|CS-ID 2|
                 :                                :      +-------+
                 :<-------------------------------:
          Service CS-ID 1, instance CIS-ID 3 <metrics>
          Service CS-ID 2, <metrics>
~~~
{: #fig-cats-example-overlay title="An Example of CATS Metric Distribution"}

The example in {{fig-cats-example-overlay}} mainly describes a per-instance computing-related metric distribution. In the case of distributing aggregated per-site computing-related metrics, the per-instance CIS-ID information will not be included in the advertisement. Instead, a per-site CIS-ID may be used in case multiple sites are connected to the Egress CATS-Forwarder to explicitly indicate the site the aggregated metrics come from.

A CIS-ID is not required if the service site can support consistently service contact instance selection.

## Service Request Processing

A C-PS computes paths that lead to Egress CATS-Forwarders according to the service and network metrics that have been advertised. A C-PS may be collocated with an Ingress CATS-Forwarder (as shown in {{fig-cats-example-overlay}}) or logically centralized.

This document does not specify any algorithm for path computation and selection purposes to be supported by C-PSes. These algorithms are out of the scope of this document. However, it is expected that a service request or local policy may feed the C-PS computation logic with Objective Functions that provide some information about the path characteristics (e.g., in terms of maximum latency) and the selected service contact instance.

In the example shown in {{fig-cats-example-overlay}}, when the client sends a service request via "CATS-Forwarder 1", the forwarder solicits
the C-PS to select a service contact instance hosted by a service site that can be accessed through a particular Egress CATS-Forwarder. The C-PS also determines a path to that Egress CATS-Forwarder. This information is provided to the Ingress CATS-Forwarder ("CATS-Forwarder 1") so that it can forward packets to their proper destination, as computed by the C-PS.

A service transaction consists of one or more service packets sent by the client to an Ingress CATS-Forwarder to which the client is connected to. The Ingress CATS-Forwarder classifies incoming packets received from clients by soliciting the CATS classifier (C-TC). When a matching classification entry is found for the packets, the Ingress CATS-Forwarder encapsulates and forwards them to the C-PS selected Egress CATS-Forwarder. When these packets reach the Egress CATS-Forwarder, the outer header of the possible overlay encapsulation is removed and inner packets are sent to the relevant service contact instance.

> Note that multi-homed clients may be connected to multiple CATS infrastructures that may be operated by the same or distinct service providers. This version of the framework does not cover multihoming specifics.

## Service Contact Instance Affinity

Instance affinity means that packets that belong to a flow associated with a service should always be sent to the same Egress CATS-Forwarder which will forward them to the same service contact instance. Furthermore, packets of a given flow should be forwarded along the same path to avoid mis-ordering and to prevent the introduction of unpredictable latency variations.

The affinity is determined at the time of newly formulated service requests.

Note that different services may have different notions of what constitutes a 'flow' and may, thus, identify a flow differently. Typically, a flow is identified by the 5-tuple transport coordinates (source and destination addresses, source and destination port numbers, and protocol). However, for instance, an RTP video stream may use different port numbers for video and audio channels: in that case, affinity may be identified as a combination of the two 5-tuple flow identifiers so that both flows are addressed to the same service contact instance.

Hence, when specifying a protocol to communicate information about service contact instance affinity, a certain level of flexibility for identifying flows should be supported. Or, from a more general perspective, there should be a flexible mechanism to specify and identify the set of packets that are subject to a service contact instance affinity.

More importantly, the means for identifying a flow for the purpose of ensuring instance affinity should be application-independent to avoid the need for service-specific instance affinity methods. However, service contact instance affinity information may be configurable on a per-service basis. For each service, the information can include the flow/packets identification type and means, affinity timeout value, etc.

This document does not define any mechanism for defining or enforcing service contact instance affinity.

# Security Considerations

The computing resource information changes over time very frequently, especially with the creation and termination of service contact instances. When such an information is carried in a routing protocol, too many updates may affect network stability. This issue could be exploited by an attacker (e.g., by spawning and deleting service contact instances very rapidly). CATS solutions must support guards against such misbehaviors. For example, these solutions should support aggregation techniques, dampening mechanisms, and threshold-triggered distribution updates.

The information distributed by the C-SMA and C-NMA agents may be sensitive. Such information could indeed disclose intel about the network and the location of compute resources hosted in service sites. This information may be used by an attacker to identify weak spots in an operator's network. Furthermore, such information may be modified by an attacker resulting in disrupted service delivery for the clients, up to and including misdirection of traffic to an attacker's service implementation. CATS solutions must support authentication and integrity-protection mechanisms between C-SMAs/C-NMAs and C-PSes, and between C-PSes and Ingress CATS-Forwarders. Also, C-SMA agents need to support a mechanism to authenticate the services for which they provide information to C-PS computation logics, among other CATS functions.

# Privacy Considerations

Means to prevent that on-path nodes in the underlay infrastructure to fingerprint and track clients (e.g., determine which client accesses which service) must be supported by CATS solutions. More generally, personal data must not be exposed to external parties by CATS beyond what is carried in the packet that was originally issued by the client.

Since the service will, in some cases, need to know about applications, clients, and even user identity, it is likely that the C-PS computed path information will need to be encrypted if the client/service communication is not already encrypted.

For more discussion about privacy, refer to {{?RFC6462}} and {{?RFC6973}}.

# IANA Considerations

This document makes no requests for IANA action.

--- back

# Acknowledgements

The authors would like to thank Joel Halpern, John Scudder, Dino Farinacci, Adrian Farrel,
Cullen Jennings, Linda Dunbar, Jeffrey Zhang, Peng Liu, Fang Gao, Aijun Wang, Cong Li,
Xinxin Yi, Jari Arkko, Mingyu Wu, Haibo Wang, Xia Chen, Jianwei Mao, Guofeng Qian, Zhenbin Li,
Xinyue Zhang, and Nagendra Kumar for their comments and suggestions.