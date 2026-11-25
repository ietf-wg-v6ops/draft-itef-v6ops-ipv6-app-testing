---
v: 3
stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]
title: "Recommendations for IPv6 Testing of Applications"
abbrev: "ipv6-app-testing"
docname: draft-ietf-v6ops-ipv6-app-testing-latest
submissiontype: IETF
category: bcp
ipr: trust200902
area: "Operations and Management"
wg: v6ops
keyword:
  - Interoperability
  - Deployment
  - Operation
  - IPv6 Failure
  - IPv6 service
  - Validation
  - Application developer

venue:
  github: ietf-wg-v6ops/draft-itef-v6ops-ipv6-app-testing
  group: "IPv6 Operations"

author:
 -
    fullname: Philipp S. Tiesel
    organization: SAP SE
    email:
      - philipp@tiesel.net
      - philipp.tiesel@sap.com
 -
    fullname: Kyle Ouellette
    organization: University of New Hampshire Interoperability Labs
    abbrev: UNH-IOL
    email: kouellette@iol.unh.edu
 -
    fullname: Ben Patton
    organization: University of New Hampshire Interoperability Labs
    abbrev: UNH-IOL
    email: bpatton@iol.unh.edu

normative:
  IPv6-ONLY: I-D.ietf-v6ops-ipv6-only
  CLAT: I-D.ietf-v6ops-claton

informative:
  RFC7084bis: I-D.ietf-v6ops-rfc7084bis
  V6MOPS: I-D.ietf-v6ops-6mops
  RFC6146-bis: I-D.ietf-v6ops-rfc6146-bis
  RFC6724-update: I-D.ietf-6man-rfc6724-update
  MASQUE: I-D.draft-schinazi-masque-proxy
  CN-CAC-2023:
    target: http://www.cac.gov.cn/2023-04/27/c_1684239012351367.htm
    title: 2023 Work Arrangement for Further Promoting Large-scale IPv6 Deployment and Application
    date: 2023-04-27
  US-OMB-M-21-07:
    target: https://www.whitehouse.gov/wp-content/uploads/2020/11/M-21-07.pdf
    title: M-21-07 - Completing the Transition to Internet Protocol Version 6 (IPv6)
    seriesinfo:
      United States of America Office of Management and Budget: Memorandum for Heads of Executive Departments and Agencies
    date: 2020-11-19
  DE-BIT-2020-14:
    title: Beschluss Nr. 2020/14 - Zukunftsfaehige Netzinfrastrukturen auf Basis von funktionsfiaehigem IPv6
    target: https://www.cio.bund.de/SharedDocs/downloads/Webs/CIO/DE/cio-bund/steuerung-it-bund/beschluesse_cio-board_KoITB/2020_14_Beschluss_Konferenz_IT_Beauftragte.pdf
    seriesinfo:
      Konferenz der IT-Beauftragten der Ressorts
    annotation: Original link is broken - cached copy available at https://raw.githubusercontent.com/ietf-wg-v6ops/draft-itef-v6ops-ipv6-app-testing/refs/tags/draft-ietf-v6ops-ipv6-app-testing-05/references/DE-BIT-2020-14.pdf
    date: 2020-11-11
  CZ-ENDv4:
    target: https://konecipv4.cz/
    title: Czech Republic sets IPv4 end date
    date: 2024-01-17
  US-NIST.SP.500-267Ar1:
    target: https://nvlpubs.nist.gov/nistpubs/specialpublications/NIST.SP.500-267Ar1.pdf
    title: NIST Special Publication 500-267 Revision 1 - NIST IPv6 Profile
    date: 2020-11-23
  iCloud-Private-Relay:
    target: https://developer.apple.com/videos/play/wwdc2021/10096/
    title: Apple iCLoud Private Relay (WWDC2021)
  IPvFoo:
    target: https://pmarks.net/ipvfoo
    title: IPvFoo - a Chrome/Firefox extension that adds an icon to indicate whether the current page was fetched using IPv4 or IPv6.
    author:
      -
        ins: P. Marks
        name: Paul Marks
  Wireshark:
    target: https://www.wireshark.org/
    title: Wireshark packet tracer
  C-ARES:
    target: https://c-ares.org/
    title: C-ARES - a modern DNS (stub) resolver library, written in C
  Selenium:
    target: https://www.selenium.dev/
    title: Selenium WebDriver
  NGINX.trac-552:
    target: https://trac.nginx.org/nginx/ticket/552
    title: Nginx doesn't honor net.ipv6.conf.all.disable_ipv6

--- abstract

This document provides guidance for application developers and Software-as-a-Service (SaaS) providers on how to approach IPv6 testing in Dual-stack (IPv4+IPv6) and IPv6-only scenarios, including "IPv6-only-strict" scenarios without any connectivity towards any relevant IPv4 endpoint.

The document also discusses common misconceptions about the degree to which Operating Systems and libraries can abstract IPv6 issues away
and explains common regressions to avoid when deploying IPv6 support.


--- middle

# Introduction

For the last 20 years, enabling applications for IPv6 has focused on coexistence with IPv4 and allowing traffic to shift towards IPv6 without breaking IPv4 operation.
This target has changed in part due to a series of national regulations mandating state entities to proceed in the transition to IPv6, e.g., in
China {{CN-CAC-2023}}, the United States of America {{US-OMB-M-21-07}}, Germany {{DE-BIT-2020-14}}, and the Czech Republic {{CZ-ENDv4}}.
IPv6 support, thus, means being fully functional in the absence of IPv4 and transition technologies providing connectivity to the IPv4 Internet.
Therefore, applications are expected to function regardless of whether they are used in an IPv4-only environment, a Dual-stack environment, or an IPv6-only environment, with or without connectivity to the IPv4 Internet. To achieve this, applications need to be verified against all these scenarios.

The availability of IPv6 support in applications, from small command-line tools to huge cloud-based distributed applications such as Enterprise Resource Planning (ERP) systems, has a considerable impact on the success of IPv6 deployment.
While the IETF does not define compliance tests, best current practices exist for the behavior of general IPv6 nodes {{?RFC8504}} and Customer Edge (CE) routers {{RFC7084bis}}, as well as for test SIP-based applications {{?RFC5118}}.
However, at the time of writing this document, there exists no documented best current practices on how to test whether applications are ready for Dual-stack and IPv6-only environments.

To fill that gap, this document provides guidance for application developers and cloud application providers on how to approach IPv6 testing.
It describes the parts of an application lifecycle to include, which communication scenarios they should consider validating against, and which common regressions to avoid when adding IPv6 support.
While many application developers assume that the network abstractions of an OS, communication libraries, application frameworks, or embedded platforms will handle the transition towards IPv6 transparently, leaky abstractions within these frameworks will make it difficult for an application developer to write address family-independent code for features such as allow/deny lists, tracing, and logging.

Testing cloud applications poses an additional challenge, as these are typically composed of hundreds to thousands of micro and macroservices, forming a complex distributed system that requires intricate communication and orchestration infrastructure to operate.
Enabling these applications to communicate over IPv6 requires careful analysis of data flows towards services, between components, and towards external services as well as analysis where IPv6 addresses may occur as metadata.

As the scope of applications to be guided by this document ranges from small command-line tools to huge cloud-based systems,
not all details discussed here are relevant for all applications.
However, the underlying idea of how to approach IPv6 application testing is the same for all of these:
Do not invent new tests for IPv6, but instead test whether the functional correctness is given irrespective of the address family used for communication. In cases where IP addresses are processed as data, make sure to update the tests to support IPv4 and IPv6 addressees.

The remainder of this document is organized as follows:
{{definitions}} contains the base definitions and scenarios used throughout the document.
{{objectives}} provides guidance on testing objectives and dimensions that should be considered in an IPv6 testing campaign.
{{strategies}} presents practical testing strategies for setting up the scenarios defined in {{objectives}} without building all scenarios within a lab environment.
{{failures}} describes common sources of IPv6-related failures and misbehavior that application developers and testers should be aware of.
Finally, {{ops}} describes operational considerations to keep in mind while designing end-to-end tests.

# Conventions and Definitions {#definitions}

## Requirements Language

{::boilerplate bcp14-tagged}

## Base Connectivity Scenarios

This document defines the following four "base connectivity scenarios"
in which applications ought to be verified for availability and functional correctness:

[^1]

[^1]: **Note to the RFC-Editor:** The capitalization of the following terms should be consistent with {{IPv6-ONLY}}.

IPv4-only:
: A node or application that has native IPv4 connectivity towards all endpoints relevant for the test scenario and no connectivity towards any relevant IPv6 endpoints.
: The definition is narrower than the one from {{IPv6-ONLY}} and mirrors the *IPv6-only-strict* scenario.
: This document refrains from calling the scenario *IPv4-only-strict* for the sake of simplicity, as transition technologies allowing IPv4-only endpoints to talk to arbitrary IPv6-only endpoints are not widely deployed. Transition technologies using encapsulation are covered by either the *Dual-stack* or *IPv6-only with NAT64* case.

Dual-stack:
: A node or application that has native connectivity towards all endpoints relevant for the test scenario using IPv4 as well as using IPv6.
: This case covers the *Dual-Stack* and *IPv6-Mostly (for clients not supporting Option 108 {{?RFC8925}})* cases in {{IPv6-ONLY}} and
narrows it down by defining the scope as test-relevant endpoints.

IPv6-only with NAT64:
: A node or application that has native connectivity towards all endpoints relevant for the test scenario using IPv6 and connectivity towards IPv4 endpoints using a transition technology such as NAT64, e.g., NAT64 in combination with CLAT {{?RFC6877}}, DNS64 {{?RFC6147}}, or local address synthesis. This document does not differentiate between stateful {{RFC6146-bis}} and stateless {{?RFC7915}} NAT64 variants.
: This case covers the *IPv6-Only* as well as *IPv6-Mostly (for clients supporting Option 108 {{?RFC8925}})* cases in {{IPv6-ONLY}} and
narrows them down by defining the scope as test-relevant endpoints.

IPv6-only-strict:
: A node or application that has native IPv6 connectivity towards all endpoints relevant for the test scenario and no connectivity towards any relevant IPv4 endpoints, neither encapsulated nor translated.
: This definition narrows down the *IPv6-Only-Strict* definition from {{IPv6-ONLY}} by defining the scope as test-relevant endpoints.

## Lifecycle Functions {#lifecycle-functions}

Orthogonal to the base scenarios, this document defines lifecycle functions (i.e., the phases in which an application is approached during a simplified lifecycle of the application) in accordance with {{US-NIST.SP.500-267Ar1}} as follows:

- Installation: The installation of the application including any initial configuration required for
  getting the application in a state where remote services are operational.

- User Interface: All forms of interactive access to the application (e.g., Web User Interface (UI) or API).

- Management: All forms of remote management and monitoring functions.

- Update: All forms of update functions, including both automatic and manual update mechanisms.

# Testing Objectives {#objectives}

As a basic principle, IPv6 application testing should always be derived from functional and integration testing.
Therefore, the goal is to verify that the application behavior is consistent across all connectivity scenarios (i.e., the application functions correctly in IPv4-only, Dual-stack, IPv6-only with NAT64, and IPv6-only-strict settings).

This section provides guidance on which connectivity scenarios and parts of the lifecycle of an application to include in a testing campaign
for an ideal application that is supposed to run anywhere.
It is RECOMMENDED to apply these scenarios to all existing functional, integration, and end-to-end test cases that the application's communication pattern may influence.
The document also explains how to approach testing of complex cloud applications and how to exclude connectivity scenarios based on the deployment environment.
Testing for application-specific or protocol-specific issues is out of scope.

While this section assumes that the communication scenarios are under full control of the tester,
{{strategies}} explains how to conduct tests in cases where control over the test environment is limited.

## Connectivity Scenarios {#scenarios}

{{scn_combinations}} lists the combinations of connectivity scenarios that should be considered when testing the application for IPv6 issues.
Not testing some of these cases may lead to failures in operations.
While the involved parties are listed here as "client" and "server" to reflect the most common case, the combinations can be used the same way when considering Peer-to-Peer (P2P) applications -- with "client" representing the initiating or first acting party.

The first five scenarios marked as *base* should cover all major code paths and fallback conditions and MUST be validated to work as expected unless the application's architecture or deployment makes them irrelevant (see {{complex-apps}} for further discussion).
These include Dual-stack clients combined with IPv4-only and IPv6-only-strict servers, to test whether the additional address family confuses the client.
The document also includes the cases with Dual-stack server and both IPv4-only and IPv6-only clients, to test whether a single address family at client side works as anticipated and to look at the transition case using NAT64.

This document does not include any special scenarios for 464XLAT {{?RFC6877}} and IPv6-Mostly {{V6MOPS}}, as these architectures are from the client side indistinguishable from the Dual-stack or IPv6-only with NAT64 (IPv6-Mostly without CLAT).

MTU issues, as described in {{Section 7.4.5 of V6MOPS}}, that may arise from these scenarios are covered in {{partially-broken}}.

This document also does not separate IPv4-only cases with and without NAT. While applications exist that assume NAT for IPv4 and do not work without, these issues are usually also revealed by the IPv6 scenarios.

For the IPv6-only data center case, where servers may be exposed to the IPv4-only Internet using NAT46, the case marked as IPv6-only-DC in {{scn_combinations}} SHOULD also be considered. This case MAY be omitted for deployments targeting IPv6-only clients.

The other combinations are unlikely to exhibit additional problems for 'client/server'-based applications and, therefore, are marked as *extended* in {{scn_combinations}}.
For P2P applications and applications with complex connection handling such as using Session Traversal Utilities for NAT (STUN) {{?RFC8489}} or Traversal Using Relays around NAT (TURN) {{?RFC8656}}, skipping scenarios classified as *extended* is strongly discouraged.

| Client               | Server               | Classification |
| :---                 | :---                 | :---:          |
| Dual-stack           | IPv4-only            | base           |
| Dual-stack           | IPv6-only-strict     | base           |
| IPv4-only            | Dual-stack           | base           |
| IPv6-only with NAT64 | IPv4-only            | base           |
| IPv6-only-strict     | Dual-stack           | base           |
| IPv4-only            | IPv6-only with NAT64 | IPv6-only-DC   |
| Dual-stack           | Dual-stack           | extended       |
| IPv4-only            | IPv4-only            | extended       |
| IPv6-only with NAT64 | IPv6-only-strict     | extended       |
| IPv6-only-strict     | IPv6-only-strict     | extended       |
{: #scn_combinations title="Connectivity scenario combinations to consider"}

## Testing with Intermediaries (e.g., Proxies)  {#intermediaries}

Many application protocols support communicating across intermediaries, most commonly HTTP CONNECT {{?RFC9110}}, SOCKS {{?RFC1928}}, or MASQUE {{MASQUE}} proxies.
P2P applications often support TURN {{?RFC8656}} as an intermediary to traverse NAT and provide connectivity between IPv4-only and IPv6-only hosts.
When testing connectivity scenarios for such an application, the additional test cases including a proxy MUST also be validated.

As a proxy can convert between address families, all combinations shown in {{scn_proxy}},
consisting of base scenarios towards the proxy and (assuming the same scenarios on both sides of the proxy) the respective base scenarios from the proxy to the server, should be considered for testing.

| Client               | Proxy                | Server           |
| :---                 | :---                 | :---             |
| Dual-stack           | IPv4-only            | Dual-stack       |
| Dual-stack           | IPv6-only-strict     | Dual-stack       |
| IPv4-only            | Dual-stack           | IPv4-only        |
| IPv4-only            | Dual-stack           | IPv6-only-strict |
| IPv6-only with NAT64 | IPv4-only            | Dual-stack       |
| IPv6-only-strict     | Dual-stack           | IPv4-only        |
| IPv6-only-strict     | Dual-stack           | IPv6-only-strict |
{: #scn_proxy title="Base scenario combinations including a proxy to consider for IPv6 testing"}


## Testing Name Resolution Issues

As most applications use name resolution to bootstrap their connectivity,
it is necessary to consider name resolution aspects when testing IPv6 readiness.
While some name resolution issues only manifest in certain connectivity scenarios or can be mitigated by using Happy Eyeballs {{!RFC8305}},
others will just map to different connectivity scenarios.

This section lists name resolution issues to consider for testing.

### Missing DNS Resource Records

While a server endpoint is intended to support Dual-stack connectivity,
A or AAAA DNS resource records for the endpoint may be missing (e.g., due to misconfiguration or broken tooling)
or may not reach the client endpoint (e.g., because they got filtered out by a middlebox or local resolver).
The same can happen for names discovered and resolved through mDNS {{?RFC6762}}, for example.

While deployment and integration testing should try to test for this kind of broken connectivity,
this scenario is usually indistinguishable from an IPv4-only or an IPv6-only server endpoint,
and, therefore, already addressed by testing the base scenarios above.

### Incorrect DNS Resource Records

Independent of the deployed server endpoint,
there may be an A and AAAA resource record pointing somewhere else (e.g., to an old or planned deployment).

For either IPv4-only or IPv6-only-strict clients, this scenario should always fail.

For Dual-stack clients, it MUST be tested whether they can use the working IPv4-only or IPv6-only connectivity scenario, either by using Happy Eyeballs {{!RFC8305}} or trying the next resolved address candidate after timeout.
Especially for the latter, it is advisable to verify whether the connection delay is acceptable for the desired use case.

IPv6-only clients with NAT64 are only expected to work with broken AAAA records when deployed with
CLAT (should behave like Dual-stack as discussed in {{scenarios}}) or
local NAT64 address (e.g., as when implementing Happy Eyeballs v2 {{RFC8305}}).
IPv6-only clients with NAT64 that rely on DNS64 only are expected to fail as the presence of AAAA resource records prevents synthesis of DNS64 records.

Testing with IPv4-Mapped IPv6 addresses {{RFC4291}} in AAAA resource records is also RECOMMENDED.
While this makes zero sense, it has been seen in the wild and should not confuse the client.

### DNS Delegation Issues

Integration testing for cloud applications MUST verify that the necessary domain names are resolvable from IPv4-only or IPv6-only-strict DNS resolvers.
{{?RFC10001}} describes common misconfigurations breaking IPv4-only or IPv6-only-strict in detail and provides guidance how to avoid these.
Applications providing endpoints SHOULD test against these misconfigurations.

### Testing with IP Address Literals

Most name resolution libraries support IP literals, i.e., textual representations of IP addresses.
Applications SHOULD be tested to determine whether they work as expected with IPv4 literals and all IPv6 address representations described in {{!RFC4291}}. Some applications (client or server) may not allow for IPv4 literals.

If there is a use case for link-local communication using IP literals, it MUST be tested whether the zone identifier can be entered as described in {{!RFC9844}} and works as expected.

### Testing Reverse DNS Lookups

Applications SHOULD be tested whether they correctly resolve and display reverse DNS lookups for all address families and deal correctly with missing DNS records if reverse DNS is used anywhere in the code or runtime environment.

### Testing with Link-local Names

A name such as `example.local` can resolve, e.g., through DNS-Based Service Discovery {{?RFC6763}},
to one or both of a link-local IPv4 address {{?RFC3927}} and a link-local IPv6 address.
The latter is incomplete and possibly ambiguous unless associated with the relevant zone identifier or zone index {{!RFC4007}}.
Applications SHOULD be tested to determine whether they work as expected with link-local names, particularly on a host with multiple network interfaces.

## Testing with Different Transports

Some applications can employ different transports or protocol variants for their communication flows.
Typical examples include HTTP-based applications running over TCP or QUIC {{?RFC9114}} (over UDP) and SIP-based applications that can do signaling over UDP, TCP, and SCTP.
While Transport Layer Security (TLS) {{?RFC9846}} or similar security layers are used for most application flows today, they may be optional from an application protocol's point of view.
IPv6-related issues that only arise with certain transports or protocol variants are usually hard to identify and debug in production. Because TCP transport is usually implemented in the OS, while UDP/QUIC transport is usually implemented in the application or a library shipped with it, diverging behaviors and failures in IP-family-specific corner cases are more likely.
Therefore, an IP connectivity testing strategy MUST also exercise all transport and protocol variants supported by the application and that might be available at the endpoints.
Testers have to assess whether these test cases use the expected transport protocol (or variant) or fall back to a less preferable transport under specific forwarding conditions.

## Testing with Partially Broken Connectivity, MTU, and Fragmentation Issues {#partially-broken}

When multiple address families are available, IP packets may traverse different forwarding paths depending on the address family.
Even when the same forwarding path is traversed, the path can exhibit distinct forwarding behaviors, especially in the presence of middleboxes, including:

 - Dropping all packets
 - Dropping particular packets, e.g., TCP vs. UDP packets, packets destined to different port numbers
 - Preventing the use of QUIC, specific port numbers, or encapsulation schemes
 - Behaving differently for the same values of ECN or DSCP fields
 - Exhibiting a different Path Maximum Transmission Unit (PMTU)
 - Dropping some ICMP messages and breaking PMTU discovery
 - Dropping fragmented packets
 - Blocking certain protocol features, e.g., certain extensions or ciphers in TLS handshake

In such scenarios, clients restricted to a single address family -- such as IPv6-only-strict clients -- may experience complete loss of connectivity,
while dual-stack clients often mask such failures by automatically falling back to another address family.
Disabling either address family may therefore break a scenario that seems to function correctly at first glance.
Testing applications against these scenarios can become a key enabler for users' acceptance of IPv6,
especially during a transition phase where partially broken connectivity is expected more frequently.

While this document provides no comprehensive guide on how to test for these network issues,
it is  REQUIRED to check applications with dual-stack servers for partial blackholing and MTU issues by testing with IPv4-only and IPv6-only-strict clients during deployment and integration testing.
Note that in some cases, connectivity issues may only become apparent late in the communication process, for example, after a successful TCP handshake / QUIC initial packet, but before a TLS handshake succeeds.
Because these issues can occur outside the testers' control, it is advisable to simulate them and ensure the application's behavior supports detecting and analyzing these errors.

## Testing without IPv4 Loopback Addresses (127.0.0.0/8)

Some applications and services may assume the existence and reachability of the IPv4 loopback addresses (127.0.0.0/8) when binding to a socket or for communicating with other services on the same host.
For example, a web server may explicitly listen on a 127.0.0.0/8 address by default.
For IPv6-only-strict scenarios, system administrators may choose to disable IPv4, including loopback.
In such cases, applications may fail to operate correctly. Applications expecting to bind to an IPv4 loopback address may fail to start when these addresses are unavailable due to a bind failure.
Applications expecting these addresses to be available for inter-service communication will result in these services being unable to communicate properly.

Because of this, when testing applications for the IPv6-only-strict scenario, it is RECOMMENDED to test the application in an environment without IPv4 on the loopback interface.

## Testing Lifecycle Function Considerations {#lifecycle-considerations}

To cover the whole lifecycle of an application including installation, user interface,
management, and update, it is recommended to test that the lifecycle functions defined in {{lifecycle-functions}} are operational within the connectivity scenarios defined in {{scn_combinations}}.
Testing the normal operations of the application is encompassed by the user interface lifecycle function.

In particular, the following MUST be considered:

- Installation: Installation may require communications with remote first-party services (e.g., activation/license server)
  or remote third-party services (e.g., package repositories). In these scenarios, the installer acts as
  the client, and the remote service acts as the server. In cases of remote third-party services, testing
  all server scenarios in {{scn_combinations}} may not be feasible and may impact the client scenarios that
  can be supported. For example, if a third-party service is IPv4-only, testing an IPv6-only-strict scenario will fail.

- User Interface: User interfaces can be incredibly complex with numerous contexts, views, API endpoints,
  CLI commands, etc. When testing an application's user interface(s), the normal operations of the application should be verified.
  Additionally, when testing non-web-based user interfaces, it is recommended to test components
  of the interface that involve communications with remote services, and those that handle network configuration parameters.
  For example, a network configuration interface might only accept IPv4 address literals for certain parameters.
  For testing web-based user interfaces, see {{web-app-considerations}}.

- Management: Depending on the application, management functions may be provided via the user interface.
  However, the application may have additional management functions (Network Configuration Protocol (NETCONF) {{?RFC6241}}, RESTCONF {{?RFC8040}}, SNMP {{?STD62}}, Open Telemetry {{?RFC9232}}, syslog {{?RFC5424}}, etc.) that should be tested.
  As the source triggering application behavior is crucial for logging and auditing functionality,
  addresses recorded need to be verified to be represented correctly. See {{addr-as-data}} about representation of addresses.

- Update: Depending on the application, update functions may be exercised during installation. However, the application
  may have additional update functions (e.g., automatic updates or manual update mechanisms) that should be tested.

## Testing Complex Cloud Applications and Applying Test Cases {#complex-apps}

Complex applications and especially cloud applications typically involve many data flows into the application, across components, and towards external services.
In such a system, an application or component may be considered as a server for some communication flows,
while being a client in others.
Therefore, test cases need to cover each data flow in all relevant scenarios.

As functional and integration tests are often defined as end-to-end test cases,
they often involve several components, e.g., micro-services, load-balancers, application gateways, logging, authentication, and authorization services, which use IP-based protocols between the components.
Therefore, an end-to-end test case breaks down to a series of flows between components, and for each of these flows,
it MUST be determined whether it is necessary to apply the connectivity scenarios from {{scn_combinations}} to it,
or whether the connectivity scenarios are only controlled by the deployment of the application.

For external flows (i.e., flows outside the application developers' control), all base scenarios from {{scenarios}} MUST be accounted for.

If one side of the flow is under administrative control, the number of scenario combinations can still be limited:
For example, a cloud software provider choosing to deploy Dual-stack endpoints can skip all non-Dual-stack cases on the respective side of the communication.
For internal flows, the relevant scenarios only depend on the applications' architecture, and only scenarios planned in the deployment need to be considered.
From a networking perspective, flows between components are typically independent, as long as endpoints are not signaled inside the protocol.
There is no need to run the Cartesian product of scenarios x communications as long as all relevant scenarios for a given flow are tested.

In addition to the data flows, an implementation may include metadata about the data flow when communicating with backend systems, e.g., for logging or authorization purposes.
While the flows towards these backend systems themselves may be safe to ignore as outlined above,
the functional correctness of the backend systems for all kinds of IP address MUST be verified as part of the test series.
See {{addr-as-data}} for details.

## Special Considerations for Web-based Applications {#web-app-considerations}

Web-based applications usually load resources from multiple parties, including Content Delivery Networks (CDNs) and analytic tools, involving data flows to all these parties.
When facing the requirement to support IPv6-only-strict users, being unable to load some resources due to missing/defective IPv6 support at the respective parties can have effects from missing analytics insights or ad revenue to severe functional defects rendering the application unusable.

When testing such applications, it is not sufficient to only focus on the initial/main interactions,
but it is REQUIRED to consider all resources and parties providing them.
As Web browsers load these resources dynamically and third-party resources may themselves may request resources from more parties, this kind of testing usually requires an instrumented Web browser,
e.g., using {{Selenium}}.

## Considerations for Addresses as Data {#addr-as-data}

When applications process IP addresses as data (e.g., as part of logging or management functionality),
this functionality MUST work for all possible address families and representations.
Ignoring IP addresses as data in the testing may result in malfunctions, such as always denying access over IPv6, or security issues, such as not logging access from IPv6 clients.

One challenge to consider is that the textual representation of IPv4 and IPv6 addresses are not canonical. Although {{!RFC5952}} recommends a canonical format, all implementations MUST accept and be able to handle any legitimate {{!RFC4291}} address format.
It allows several valid textual representations for the same address, which makes direct string comparison and arithmetic operations on addresses error-prone and slow.
For example, the IPv6 address `2001:db8::1` can be written as `2001:db8:0:0:0:0:0:1`, `2001:0db8::0.0.0.1`, or in several other forms.

While custom logic to check, parse and process addresses is often error-prone and should be validated thoroughly,
modern environments and frameworks usually provide data structures that encapsulate the canonical binary representation and include methods for parsing the various textual representations, comparing addresses, performing subnet operations, and rendering addresses in a consistent format.

For situations where textual representation of IPv6 addresses is needed -- such as in user interfaces, logging output, and text-based data formats such as JSON, YAML, TOML, and XML -- {{!RFC5952}} provides recommendations on which of the valid textual representations should be used.
Applications MUST be tested whether they follow {{!RFC5952}} when rendering IPv6 addresses in textual form,
as required by national regulations such as {{US-NIST.SP.500-267Ar1}},
while accepting all valid representations defined in {{!RFC4291}}.

## IPv4 addresses represented or embedded into IPv6 addresses {#ipv4-mapped}

IPv4 addresses can also be represented as *IPv4-Mapped IPv6 Addresses* (see {{?RFC4291}} Section 2.5.5.2) or as a part of a NAT64 prefix (see {{?RFC6052}}).
Depending on the environment an application is running in, logic treating IPv4 addresses (e.g., allow/deny lists) MUST be tested again all representations that are expected to occur in the particular environment and SHOULD behave consistent with IPv4 addresses (e.g., deny access from IPv4 addresses listed in a deny list even when received as IPv4-Mapped IPv6 Address or as NAT64 address when a server is expected to be deployed in an IPv6-only with NAT64 scenario).


# Testing Strategies {#strategies}

Naive IPv6 testing, based on end-to-end functional tests as outlined in {{objectives}}, would require running a set of functional tests in various connectivity scenarios.
In certain environments, setting up test cases for all scenarios can become forbiddingly expensive,
especially for complex cloud applications, application platforms, or when dealing with corporate IT environments.

This section gives recommendations how to set up scenarios defined in {{scenarios}} and
presents strategies to meet the relevant testing objective by modifying Dual-stack clients and servers to conclude the results for other scenario combinations (e.g., by tracing whether the right address family is used).

## IPv6-only-strict Clients

Testing from an IPv6-only-strict client is the most natural way to test whether an application behaves correctly in an IPv6-only-strict scenario.
The client device is either placed in a network without IPv4 connectivity or the IPv4 stack is disabled on the device while it is in a Dual-stack network.
While most desktop OSes allow disabling IPv4, mobile OSes, such as Android and iOS, do not.
For mobile OSes, an IPv6-only-strict environment is needed.

In both cases, it has to be ensured that there is no way to access IPv4-only resources.
In particular, fallback to NAT64 must be prevented by disabling CLAT {{CLAT}},
making sure DNS resolution does not perform DNS64 address synthesis {{!RFC6147}}
and blocking the Well-known Prefix (WKP) {{!RFC6052}} for these clients.
In addition, VPN services including privacy services such as {{iCloud-Private-Relay}} need to be disabled as they can provide connectivity towards the IPv4 Internet.

A note on the applicability of disabling IPv4:
Before disabling IPv4 make sure the environment supports IPv6-only operation.
Many desktop virtualization environments become unusable because IPv4 is needed to access and manage the virtual machines.
Some corporate environments may render the machines unusable as they require IPv4 connectivity for sign-on.

## IPv6-only Servers

IPv6-only servers are a good option when setting up an IPv6-only-strict client environment is infeasible and
it is clear that clients only contact a single server or a small number of servers under the testers' control.
Even if setting up an IPv6-only-strict server environment is infeasible,
most testing is also achievable by setting up a dedicated DNS name only containing an AAAA resource record pointing to the IPv6 addresses of an otherwise Dual-stack server.

## Client-based Tracing

If the tester cannot limit the available address families, the tester can still trace and verify whether the address family desired for the scenario is used.

Client-based tracing is especially useful when Dual-stack servers and clients are available and a conclusion for the IPv6-only-strict case is desired.
By using the clients' logging/tracing/debugging functionality, the tester can verify that the actual data flows happen over IPv6, which is preferred by most network abstractions. If the client allows changing the preference between IPv6 and IPv4, IPv4-only testing is also possible.

The most relevant case for this strategy is testing Web applications.
By examining the Web browser performance log or using a plugin such as {{IPvFoo}} that visualizes connectivity information, the tester can determine whether all resources are available using IPv6.

## Server-based Tracing

Analogously to tracing on the client side, it is also possible to look at the protocols used on the server side.

While this is functionally equivalent for protocols where clients only communicate to a single server,
this approach is not feasible for Web-based applications where a client usually needs flows towards many servers, where client or network based tracing are the only feasible alternatives to testing with an IPv6-only-strict client.

## Network-based Tracing

If the communication pattern of an application is known well enough, a packet tracer such as {{Wireshark}} can verify that an application in a Dual-stack environment uses IPv6 for all of its flows.
If this can be verified, failures in IPv6-only-strict environments are unlikely.

While this is the least invasive method of testing IPv6-only-strict scenarios in a Dual-stack setup,
it is the most error-prone as it requires the tester to fully understand the network flows of the application
and cross-check these expected flows with the flows shown in the packet trace.


# Common Sources of IPv6 Related Failures and Misbehavior {#failures}

This section discusses special failure modes that can cause unexpected application behavior that is hard to debug.

Some of these cases can be automatically mitigated, especially by using Happy Eyeballs {{?RFC8305}} or similar techniques.
In cases where this is not possible or where developers choose not to implement mitigation strategies, users and operators should be supported in manual resolution by providing detailed error or debug messages.

## IPv6 Feature Gates

Some applications completely ignore IPv6 unless explicitly configured to enable IPv6.
This adds another class of user or configuration errors, such as deploying an application without enabled IPv6 support in an IPv6-only environment.
As these feature gates are often buried deeply in the documentation and are often vendor, product, or component specific, every component needs to be checked to determine whether IPv6 support needs extra configuration.

## Neglecting to Test Management and Control Interfaces

Ignoring management and control interfaces when enabling and testing applications for IPv6-only operation turns out to be a common pattern.
As already discussed in {{lifecycle-considerations}}, special care should be taken to cover all of these flows and avoid setups that require Dual-stack deployment.

## Destination Address Selection Preference and Address Filtering

The destination address selection algorithm in {{!RFC6724}} filters unavailable address families (Rule 1),
de-prioritizes non-matching address families (Rule 2), and clearly prioritizes IPv6 Global Unicast Addresses (GUAs, see {{?RFC4291}} Section 2.5.4) over IPv4 addresses.
Some OSes and alternative resolver libraries, such as {{C-ARES}}, implement {{RFC6724}} correctly. However, at the time of writing this document, major operating systems are still implementing the old behavior from {{?RFC3484}} -- if an application relies on behavior updated through {{RFC6724}} or {{RFC6724-update}}, especially preference of ULA, integration testing against this behavior SHOULD be performed.

Beyond that, there are several notable and widely used implementations that implement an address selection algorithm different from {{RFC6724}} or its predecessor {{?RFC3484}}, causing unexpected behavior and other hard-to-debug errors:

- Most JAVA runtimes do the opposite and prefer IPv4 destinations over IPv6.
  To prefer IPv6 addresses over IPv4, one needs to set the system property ```java.net.preferIPv6Addresses=true```.

- Some applications only use the first address candidate from the ```getaddrinfo()``` and fail if the connection attempt to that one fails.

- Applications composed of services built on different programming languages or runtimes may behave inconsistently with regard to choosing destination addresses.

- NGINX (at the time of writing, since at least version 1.6 and unchanged in 1.31) has its own user DNS resolver without address filtering; thus, adding a _AAAA_ resource record to a backend can render that backend unusable.
  After having resolved a _AAAA_ resource record, it is trying to open an IPv6 socket, even when the IPv6 stack is disabled.
  As socket creation failure is not expected, an internal server error is sent back to the client {{NGINX.trac-552}}.
  While the developers marked the bug as invalid because it can be considered being a misconfiguration,
  this configuration can occur during IPv6 enablement of complex applications maintained by multiple teams.

- Some resolvers ignore address families for which no default route exists or where the default-route is pointing to an unsupported/ignored device.
  This becomes cumbersome especially in split-VPN use cases (e.g. when trying to contact IPv6-only endpoints via the VPN while having IPv4-only Internet connectivity).

## Listening on IPv4 Only

Many tutorials and programmer facing documentations have still not been updated to cover listening on multiple address families to accept connections from both IPv6 and IPv4.
Listening code should be checked whether it supports distinct listening sockets for IPv6 and IPv4, or configures IPv6 sockets to also bind to IPv4.
Listening code should also be able to deal with cases where IPv6 or IPv4 have been disabled in the OS.

In deployments, it is recommended to always use tools such as ```netstat``` or ```lsof``` (if available) to verify all relevant address families are listened on.

## Input Validation and Output Rendering

While most libraries and application frameworks have decent IPv6 support,
there often is still application logic that prevents taking advantage of the IPv6 support by the underlying components.
Checking whether user input is a valid IPv4 address or rendering output under the assumption that an address is always an IPv4 address are typical examples for this class of limitations.

Even applications that have been adapted to IPv6 may have based their input validation on wrong assumptions,
e.g., checking that an IPv6 address is in `2000::/3`.
Therefore, test cases should include IPv6 addresses from GUA (`2000::/3`), ULA (`fc00::/7`), and NAT64 WKP (`64:ff9b::/96`), as well as link-local (`fe80::/10`) range.

The link-local range needs special attention as addresses are incomplete and possibly ambiguous unless associated with the relevant zone identifier or zone index {{!RFC4007}}.
Testing should verify that the zone identifier or zone index are correctly displayed and can be passed if endpoints may be link-local addresses.

## Connectivity Checks

Some applications perform connectivity checks to determine whether Internet access is available by trying to connect to one or more well-known endpoints.
If the connectivity check endpoints differ from the actual endpoints used by the application, this approach can lead to incorrect conclusions -
especially in IPv6-only environments when connectivity checks are IPv4-only or in environments with strict network polices or split-tunnel VPNs.
Applications should prefer implementing appropriate error handling for connectivity issues than relying on connectivity pre-checks.

## Address Bindings in Server Backends

Some backend services use the remote IP address of previous requests as an additional security mechanism and prevent subsequent requests from different addresses.
Address changes, e.g., through Happy Eyeballs implementations trying to switch address family or carrier grade NAT64 implementations mapping a subsequent request to a different source, may therefore break within a session on these events.

## Misbehaving Middleboxes

In practice, many IPv6-related regressions uncovered during testing turn out to be caused by hidden components outside of the application developers' control.
Middleboxes (e.g., firewalls, virus scanners, and intrusion detection systems) can break end-to-end tests in surprising ways, such as terminating TLS sessions over IPv6 with certain extensions in the _TLS client hello_ while correctly passing the same flow over IPv4.

# Operational Considerations {#ops}

Lab testing of applications for IPv6 compliance should always have the next step in mind: Deploying the application and providing the users with decent IPv6 support.
Therefore, end-to-end tests, especially of cloud applications, should also keep deployment steps, prerequisites, and risks in mind.

This section discusses some issues to keep in mind when planning and executing IPv6 testing.

## Operational Scope & Software Lifecycle

Depending on the application and deployment model, the timing of deploying IPv6 support may be in control of the users' organization, the developers' organization, or neither of them.

Based on this setup, certain combinations of IPv6-enabled clients, servers, and infrastructure in between may or may not be excluded from consideration.
Therefore, it may be necessary to add test cases for old software versions with known and already fixed bugs against newly IPv6-enabled servers.
If regressions and service disruptions cannot be ruled out by the tests, a per-user or per-customer tenant opt-in/opt-out/roll-back scheme for the IPv6 enablement SHOULD be considered.

## Allow & Deny Lists

Application-level IP allow and deny lists pose a special challenge for deploying IPv6 in a cloud application.
As users may already have IPv6 connectivity, adding IPv6 support to the server may cause clients to use IPv6 immediately.
Having no allow list entry for the users' IPv6 addresses results in service disruptions.
Happy Eyeballs as defined in {{?RFC8305}} does not solve the problem as allow list checks usually take place after the transport connection has already been established.

To prevent unintended exposure of services and mitigate allow or deny lists causing service disruptions when enabling IPv6,
it is RECOMMENDED to enable including IPv6 addresses in allow and deny lists way before rolling out IPv6 on the transport and communicated towards the users.
To further limit the probability of service disruptions, generalizing Happy Eyeballs to re-try using IPv4 after certain error conditions should be evaluated.

## Component and Service Reuse

If components or cloud services can be reused in other products, special care needs to be taken when planning IPv6 deployment.
The interaction contracts between the reusing parties and the service need to be checked
whether IPv6 enablement of the services also affects the flows of these.
Additional end-to-end tests, including the reusing parties, are recommended.
This is often a recursive process.

## Ownership of Software Components

Sometimes enabling IPv6 requires touching components that are not actively maintained anymore.
Operators have to be prepared for this and plan extra time or budget for updating or replacing these components.


# Security Considerations

The testing procedures described in this document do not create any new security implications.
Some security-related issues that should be considered and ruled out by appropriate testing are discussed in {{failures}}.

# IANA Considerations

This document has no IANA actions.


--- back

# Example: Deriving IPv6 Test Cases for a Web Application {#example}

This appendix illustrates how the methodology of this document can be applied to a concrete application.
It starts from the application's existing end-to-end tests and its system architecture,
crosses the two to identify the relevant data flows between components as described in {{complex-apps}},
and finally derives the IPv6 test cases by applying the connectivity scenarios from {{scn_combinations}} to each flow.

The example is deliberately kept small; a real application will have many more components, flows, and tests.
The intent is to demonstrate the method, not to provide a complete test plan.
This appendix is informative.

## The Application and its End-to-End Tests {#example-app}

The example application is the online shop of a bookshop.
Customers use a browser-based frontend to browse the catalog, check availability, and place orders.

The bookshop already maintains a suite of functional end-to-end tests.
These tests predate any IPv6 testing effort and verify the functional correctness of the application; they form the basis from which the IPv6 test cases are derived.
{{example-e2e-tests}} provides a non-exhaustive list of such tests.

| Test  | Description                                                                 |
| :---  | :---                                                                        |
| E2E-1 | Load the shop front page and browse the catalog.                            |
| E2E-2 | Search the catalog and open a book's detail page.                           |
| E2E-3 | Open a book detail page showing real-time stock availability.               |
| E2E-4 | Register a new customer account and sign in.                                |
| E2E-5 | Add books to the cart and place an order, issuing a shipment.               |
| E2E-6 | Open the order history and track the shipment status of an existing order.  |
| E2E-7 | Load a page and confirm the third-party analytics resource loads and events are sent. |
{: #example-e2e-tests title="Existing end-to-end tests"}

## Architecture {#example-architecture}

While the frontend runs in a Web browser in the user's side, the application is delivered and served from a cloud-based backend operated by the bookshop, and it additionally loads a third-party analytics solution directly into the customer's browser.
The backend is composed of the following components:

- a Load Balancer (LB) that terminates TLS, serves the static content, and reverse-proxies API requests to the business logic;
- a Business Microservice (BMS) that implements the shop's business logic;
- a Database (DB) that keeps the business state (catalog, customer accounts, carts, and orders);
- an Audit Log Service (ALS), internal to the backend, to which the BMS writes audit records of security-relevant business operations; each record includes the originating client's IP address and other request metadata; and
- an external Logistics Partner API (LPA), operated by a third party, that the BMS queries for stock levels and uses to issue shipping orders.

{{example-arch}} shows these components and the connections between them.
The data flows are labeled F1 to F6 for reference; which of them are relevant for IPv6 testing is determined in {{example-flows-sec}}.
The direction on the edges points from client to the server of the flow.

~~~ aasvg
                                  Operator's cloud backend
                           .--------------------------------------.
                           |                                      |
.-----------------.        |   .------------.                     |
|    Browser      |        |   |    Load    |         .---------. |
|     based       |   F1   |   |  Balancer  |----F3-->| Business| |
|    frontend     |----------->|  + static  |         |  Micro- | |
'--------+--------'        |   |    content |         | service | |
         |                 |   '------------'         '-+--+--+-' |
         |                 |                            |  |  |   |
         | F2              |       .-----------.        |  |  |   |
         V                 |       | Database  |<---F4--+  |  |   |
 .---------------.         |       '-----------'           |  |   |
 |   3rd-party   |         |                               |  |   |
 |   analytics   |         |       .-----------.           |  |   |
 '---------------'         |       | Audit Log |<---F6-----+  |   |
                           |       '-----------'              |   |
                           '----------------------------------|---'
                                                              | F5
                                                              V
                                                 +-----------------+
                                                 |    Logistics    |
                                                 |   Partner API   |
                                                 +-----------------+
~~~
{: #example-arch title="Components and data flows of the example bookshop application"}

## Identifying the Relevant Flows {#example-flows-sec}

Tracing each end-to-end test through the architecture shows which component-to-component connections it exercises.
{{example-flows}} summarizes the resulting flows.
Crossing the tests of {{example-e2e-tests}} with the architecture of {{example-arch}} yields the matrix in {{example-matrix}}; its populated columns are the data flows relevant for IPv6 testing.

| Flow | From      | To                    | Scope    | Description                                                     |
| :--- | :---      | :---                  | :---     | :---                                                            |
| F1   | Browser   | LB                    | external | Static content (page bundle incl. style sheets, images) and API requests, reverse-proxied to the business microservice (HTTPS). |
| F2   | Browser   | 3rd-party analytics   | external | Analytics events sent from the customer's browser.              |
| F3   | LB        | BMS                   | internal | Reverse-proxied API requests inside the backend.                |
| F4   | BMS       | Database              | internal | Reads and writes of business state.                             |
| F5   | BMS       | Logistics Partner API | external | Stock queries and shipping orders towards the logistics partner.|
| F6   | BMS       | Audit Log Service     | internal | Audit records of business operations, carrying the originating client IP address as data. |
{: #example-flows title="Data flows of the example bookshop application"}

| Test  | F1 | F2 | F3 | F4 | F5 | F6 |
| :---  | :-:| :-:| :-:| :-:| :-:| :-:|
| E2E-1 | X  |    | X  | X  |    |    |
| E2E-2 | X  |    | X  | X  |    |    |
| E2E-3 | X  |    | X  | X  | X  |    |
| E2E-4 | X  |    | X  | X  |    | X  |
| E2E-5 | X  |    | X  | X  | X  | X  |
| E2E-6 | X  |    | X  | X  | X  |    |
| E2E-7 | X  | X  | X  |    |    |    |
{: #example-matrix title="End-to-end tests crossed with the architecture: the flows each test exercises"}

For the purpose of this example, the scope of each flow is classified following {{complex-apps}}:

- F1 and F5 are external flows for which one side is under the operator's administrative control (the LB for F1, the BMS for F5) and the other side is not (the customer's browser, the LPA).
- F2 is an external flow where neither side is under the operator's control; the bookshop controls neither the browser nor the third-party analytics provider.
- F3, F4, and F6 are internal flows, fully under the operator's administrative control, and thus only need to be considered in the connectivity scenarios actually planned for the deployment.

Note that, even though F3 and F6 are internal flows whose transport only needs to be tested against the deployed scenario, they carry the originating client's IP address as data and need to be tested to work correctly for all relevant address families as discussed in {{addr-as-data}}.
For example, the ALS's handling of that address value (storage, comparison, and rendering) must therefore be verified for all address families and representations, independent of the connectivity scenario used for the flow itself.

## Deriving the IPv6 Test Cases {#example-derivation}

The flows identified in {{example-flows-sec}} and their classification are used to derive the IPv6 test cases:
for each flow, all connectivity scenarios from {{scn_combinations}} that can occur based on the deployment assumptions described in {{complex-apps}} are applied.

For the bookshop example, a Dual-stack public edge (i.e., the LB endpoint) is used while internally IPv6-only with NAT64 for outbound IPv4 connectivity is used.

The internal flows F3, F4, and F6 are fully under the operator's administrative control.
Following {{complex-apps}}, their connectivity only needs to be validated in the single configuration that is actually deployed; they are not multiplied across connectivity scenarios.

Two of these flows, however, carry an IP address as data and must be tested with all connectivity scenarios that might occur on F1 in the data.

- F3 carries the originating client's address in the `Forwarded` {{?RFC7239}} header the LB adds when proxying to the BMS.
- F6 carries that same address where the BMS writes it to the audit log.

This leaves F1, F2, and F5 for connectivity-scenario testing, plus data tests on F3 and F6.
Each of these flows can be exercised either through the full end-to-end test or in isolation between its two endpoints.
Running everything end-to-end stays closest to production and reuses the existing tests unchanged, but it couples the flows:
one run fixes the browser scenario (F1, F2) and the logistics scenario (F5) at the same time.
Testing each flow individually avoids the Cartesian blow-up warned against in {{complex-apps}} but may be difficult to build as a test case.

This example keeps the end-to-end tests for the browser edge (F1, F2), where the browser genuinely drives the flow, as well as for the F3/F6 data handling.
The server-initiated flow F5 is tested individually, as it is largely independent of the browser scenario.

## Scenarios and Data Tests for the Remaining Flows {#example-remaining}

The connectivity scenarios to cover for F1, F2, and F5 follow directly from {{scenarios}} once the controlled side of each flow is fixed to the deployment above:

F1:
: The server (edge) is Dual-stack, so vary the client/browser side: IPv4-only and IPv6-only-strict (base cases).

F2:
: Follows the F1 browser scenario; even in the case of an IPv6-only-strict browser with a IPv4-only analytics provider, the shop must still render (see {{web-app-considerations}}).
: Whether the loss of analytics capabilities is critical is a business decision.

F5:
: The BMS is an IPv6-only-with-NAT64 client, so present the server as IPv4-only (exercising NAT64), Dual-stack, and IPv6-only-strict.

The data tests for F3 and F6 do not need their own connectivity setup; they are driven by the client addresses that the F1 scenarios above produce.
The IPv4-only browser scenario yields an IPv4 client address and the IPv6 browser scenarios yield an IPv6 GUA, and each must be carried unchanged into the `Forwarded` header on F3 and into the audit record on F6.
For every input, verify that the BMS parses the header correctly (F3) and that the ALS stores it.

In case the Logistics backend (F5) is dual stack, a trick from {{strategies}} can be applied to avoid testing against a mocked endpoint:
Point the BMS at a controlled DNS recursor that can filter A or AAAA resource records to simulate IPv4-only and IPv6-only servers.

# Acknowledgments
{:unnumbered}

Thanks to
Andrew Yourtchenko,
Axel Schemberg,
Brian E Carpenter,
Holger Fuessler,
Jeremy Duncan,
Jordi Palet,
Michael Perscheid,
Michael Richardson,
Nathan Sherrard,
Sulabh Soneji,
and
Tommy Jensen
for the discussions, the input, and all contribution.

Special thanks to Jen Linkova and Timothy Winters for advise on how to shape the document and finding the right set of authors
as well as Mohamed Boucadair for numerous fruitful discussions and suggestions.

{{example}} has been written with the help of Anthropic Opus 4.8.
