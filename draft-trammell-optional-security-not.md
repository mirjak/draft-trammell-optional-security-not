---
title: Optional Security Is Not An Option
abbrev: optional-security
docname: draft-trammell-optional-security-not
date:
category: info

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: B. Trammell
    name: Brian Trammell
    org: ETH Zurich
    email: ietf@trammell.ch
    street: Universitatstrasse 6
    city: 8092 Zurich
    country: Switzerland

normative:

informative:
  Axelsson99:
    author:
      -
        ins: S. Axelsson
    title: The Base-Rate Fallacy and its Implications for the Difficulty of Intrusion Detection (in ACM CCS 1999)
    target: http://www.raid-symposium.org/raid99/PAPERS/Axelsson.pdf
    date: 1999
  Lychev13:
    author:
      -
        ins: R. Lychev
      -
        ins: S. Goldberg
      -
        ins: M. Schapira
    title: BGP Security in Partial Deployment - Is the Squeeze Worth the Juice? (in SIGCOMM 2013)
    target: https://conferences.sigcomm.org/sigcomm/2013/papers/sigcomm/p171.pdf
    date: 2013
  Gilad17:
    author:
      -
        ins: Y. Gilad
      -
        ins: A. Cohen
      -
        ins: A. Herzberg
      -
        ins: M. Schapira
      -
        ins: H. Schulman
    title: Are We There Yet? On RPKI’s Deployment and Security (in NDSS 2017)
    target: https://www.ndss-symposium.org/ndss2017/ndss-2017-programme/are-we-there-yet-rpkis-deployment-and-security/
    date: 2017-11
  Chung17:
    author:
      -
        ins: T. Chung
      -
        ins: R. van Rijswijk-Deij
      -
        ins: D. Choffnes
      -
        ins: D. Levin
      -
        ins: B. Maggs
      -
        ins: A. Mislove
      -
        ins: C. Wilson
    title: Understanding the Role of Registrars in DNSSEC Deployment
    target: https://conferences.sigcomm.org/imc/2017/papers/imc17-final53.pdf
    date: 2017-11
  DNSSEC-DEPLOYMENT:
    author:
      -
        ins: Internet Society
    title: State of DNSSEC Deployment 2016
    target: https://www.internetsociety.org/resources/doc/2016/state-of-dnssec-deployment-2016/
    date: 2016-12
  IAB-CONFIDENTIALITY:
    author:
      -
        ins: Internet Architecture Board
    title: IAB Statement on Internet Confidentiality
    target: https://www.iab.org/2014/11/14/iab-statement-on-internet-confidentiality/
    date: 2014-11
--- abstract

This document explores the common properties of optional security protocols
and extensions, and notes that due to the base-rate fallacy and general issues
with coordinated deployment of protocols under uncertain incentives, optional
security protocols have proven difficult to deploy in practice.

--- middle

# Introduction

Many of the protocols that make up the Internet architecture were designed and
first implemented in an envrionment of mutual trust among network engineers,
operators, and users, on computers that were incapable of using cryptographic
protocols to provide confidentiality, integrity, and authenticity for those
protocols, in a legal environment where cryptographic technology was largely
protected by restricted licensing and/or prohibited by law. The result has
been a protocol stack where security properties have been added to core
protocols using those protocol's extension mechanisms.

As extension mechanisms are by design optional features of a protocol, this
has led to a situation where security is optional up and down the protocol
stack. Protocols with optional security have proven to be difficult to deploy.
This document describes and examines this problem, and provides guidance for
future evolution of the protocol, based on current work in network measurement
and usable security research.

# Problem statement

Consider an optional security extension with the following properties:

1. The extension is optional: a given connection or operation will succeed
   without the extension, albeit without the security properties the extension
   guarantees.
2. The extension has a true positive probability P: the probability that it
   will cause any given operation to fail, thereby successfully preventing an
   attack that would have otherwise succeeded had the extension not been
   enabled. This probability is a function of the extension's effectiveness as
   well as the probability that said operation will be an instance of the
   attack the extension prevents.
3. The extension has a false positive probability Q: the probability it will
   cause any given operation to fail due to some condition other than an
   attack, e.g. due to a misconfiguration.

Moving from no deployment of an optional security extension to full deployment
is a protocol transition as described in {{?RFC8170}}. We posit that the
implicit transition plans for these protocols have generally suffered from an
underestimation of a disincentive (section 5.2) linked to the relationship
between P and Q for any given protocol.

Specifically, if Q is much greater than P, then any user of an optional
security extension will face an overwhelming incentive to disable that
extension, as the cost of dealing with spuriously failing operations becomes
greater than the cost of dealing with relatively rare successful attacks. This
incentive becomes stronger when the cause of the false positive is someone
else's problem; i.e. not a misconfiguration the user can possibly fix. This
situation can arise when poor design, documentation, or tool support elevates
the incidence of misconfiguration (high Q), in an environment where the attack
models addressed by the extension are naturally rare (low P).

This is not a novel observation; a similar phenomenon following from the
base-rate fallacy has been studied in the literature on operational security,
where the false positive and true positive rates for intrusion detection
systems have a similar effect on the applicability of these systems. Axelsson
showed {{Axelsson99}} that the false positive rate must be held extremely low,
on the order of 1 in 100,000, for the probability of an intrusion given an
alarm to be worth the effort of further investigation.

Indeed, the situation is even worse than this. Experience with operational
security monitoring indicates that when Q is high enough, even true positives
P may be treated as "in the way".

# Case studies

Here we examine four optional security extensions,  BGPSEC {{?RFC8205}}, RPKI
{{?RFC6810}}, DNSSEC {{?RFC4033}}, and the addition of TLS to HTTP/1.1
{{?RFC2818}}, to see how the relationship of P and Q has affected their
deployment.

## Routing security: BGPSEC and RPKI

The Border Gateway Protocol {{?RFC4271}} (BGP) is used to propagate
interdomain routing information in the Internet. Its original design has no
integrity protection at all, either on a hop-by-hop or on an end-to-end basis.
In the meantime, the TCP Authentication Option {{?RFC5925}} (and MD5
authentication {{?RFC2385}}, which it replaces) have been deployed to add
hop-by-hop integrity protection.

End-to-end protection of the integrity of BGP announcements is protected by
two complementary approaches. Route announcements in BGP updates protected by
BGPSEC {{?RFC8205}} have the property that the every Autonomous System (AS) on
the path of ASes listed in the UPDATE message has explicitly authorized the
advertisement of the route to the subsequent AS in the path. RPKI {{?RFC6810}}
protects prefixes, granting the right to advertise a prefix (i.e., be the
first AS in the AS path) to a specific AS. RPKI serves as a trust root for
BGPSEC, as well.

These approaches are not yet universally deployed. BGP route origin
authentication approaches provide little benefit to individual deployers until
it is almost universally deployed {{Lychev13}}. RPKI route origin validation
is similarly deployed in about 15% of the Internet core; two thirds of these
networks only assign lower preference to non-validating announcements. This
indicates significant caution with respect to RPKI mistakes {{Gilad17}}. In
both cases the lack of incentives for each independent deployment, including
the false positive risk, greatly reduces the speed of incremental deployment
and the chance of a successful transition {{?RFC8170}}.

## DNSSEC

The Domain Name System (DNS) {{?RFC1035}} provides a distributed protocol for
the mapping of Internet domain names to information about those names. As
originally specified, an answer to a DNS query was considered authoritative if
it came from an authoritative server, which does not allow for authentication
of information in the DNS. DNS Security {{?RFC4033}} remedies this through an
extension, allowing DNS resource records to be signed using keys linked to
zones, also distributed via DNS. A name can be authenticated if every level of
the DNS hierarchy from the root up to the zone containing the name is signed.

The root zone of the DNS has been signed since 2010. As of 2016, 89% of TLD
zones were also signed. However, the deployment status of DNSSEC for
second-level domains (SLDs) varies wildly from region to region and is
generally poor: only about 1% of .com, .net. and .org SLDs are properly signed
{{DNSSEC-DEPLOYMENT}}. Chung et al found recently that second-level domain
adoption was linked incentives for deployment: TLDs which provided direct
financial incentives to SLDs for having correctly signed DNS zones tend to
have much higher deployment {{Chung17}}.

However, the base-rate effect tends to reduce the use of DNSSEC validating
resolvers, which remains below 15% of Internet clients {{DNSSEC-DEPLOYMENT}}.

## HTTP over TLS

Security was added to the Web via HTTPS, running HTTP over TLS over TCP, in
the 1990s {{?RFC2818}}. Deployment of HTTPS crossed 50% of web traffic in
2017, due to accelerated deployment in the wake of the Snowden revelations in
2013, and increased confidentiality of Web content delivery was considered
useful to address the attacker model laid out in {{?RFC7624}}.

Base-rate effects didn't hinder the deployment of HTTPS per se; however, until
recently, warnings about less-safe HTTPS configurations (e.g. self-signed
certificates) were less forceful due to the prevalence of these
configurations. The reduction of misconfigurations and the cost of obtaining
certificates with basic authentication checks through automation
{{?I-D.ietf-acme-acme}} has been a major force in improving Web security.

The ubiquitous deployment of HTTPS is a rare, eventual success story in the
deployment of an optional security mechanism. We note that each endpoint
deciding to use HTTPS saw an immediate benefit, which indicates good chances
of success for incremental deployment. However, the acceleration of deployment
since 2013 is the result of the coordinated effort of actors throughout the
Web application and operations stack, unified around a particular event (the
Snowden relevations) which provided a "call to arms".

# Discussion and guidelines

It has been necessary for all new protocol work in the IETF to consider
security since 2003 {{?RFC3552}}, and the Internet Architecture Board
recommended that all new protocol work provide confidentiality by default in
2014 {{IAB-CONFIDENTIALITY}}; new protocols should therefore already not rely on
optional extensions to provide security guarantees for their own operations or
for their users.

In many cases in the running Internet, the ship has sailed: it is not at this
point realistic to replace protocols relying on optional features for security
with new, secure protocols: while these full replacements are less susceptible
to base-rate effects, they have the same misaligned incentives to deploy. In
these cases, we note that there are, however, some small reasons for hope:

- When natural incentives are not enough to overcome base-rate effects,
  external incentives (such as financial incentives) have been shown to be
  effective to motivate single deployment decisions.
- Efforts to automate configuration of security protocols, and thereby reduce
  the incidence of misconfiguration Q, also has a positive impact on
  deployability.

# Acknowledgments

Many thanks to Peter Hessler, Geoff Huston, and Roland van Rijswijk-Deij for
conversations leading to the problem statement presented in this document. The
title shamelessly riffs off that of Berkeley tech report about IP options
written by Rodrigo Fonseca et al., via a paper at IMC 2017 by Brian Goodchild
et al.

This work is partially supported by the European Commission under Horizon 2020
grant agreement no. 688421 Measurement and Architecture for a Middleboxed
Internet (MAMI), and by the Swiss State Secretariat for Education, Research, and
Innovation under contract no. 15.0268. This support does not imply endorsement.
