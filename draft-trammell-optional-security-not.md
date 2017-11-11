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
    street: Gloriastrasse 35
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

--- abstract

write me

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

# Case studies

Here we examine three optional security extensions, DNSSEC {{?RFC4033}},
BGPSEC {{?RFC8205}}, and the addition of TLS to HTTP/1.1 {{?RFC2818}}, to see how
the relationship of P and Q has affected their deployment.

## DNSSEC

\[EDITOR'S NOTE: see https://www.internetsociety.org/resources/doc/2016/state-of-dnssec-deployment-2016/]

\[EDITOR'S NOTE: cite IMC17 paper (and earlier work) measuring deployment.]

## BGPsec

\[EDITOR'S NOTE: find citations and write me.]

## HTTP over TLS

\[EDITOR'S NOTE: write me, this is so far a partial success story but has required coordinated
effort up and down the stack.]

# Discussion and guidelines

\[EDITOR'S NOTE: write me. the inertia here is severe but can be mitigated. through external
distortion of incentives; i.e. paying people to turn DNSSEC on seems to work,
as does reducing the cost of certificates to zero and the effort (and
therefore misconfiguration incidence Q) to epsilon. ]

# Acknowledgments

Many thanks to  Peter Hessler, Geoff Huston, and Roland van Rijswijk for
conversations leading to the problem statement presented in this document. The
title shamelessly riffs off that of Berkeley tech report about IP options
written by Rodrigo Fonseca et al., via a paper at IMC 2017 by Brian Goodchild
et al.

This work is partially supported by the European Commission under Horizon 2020
grant agreement no. 688421 Measurement and Architecture for a Middleboxed
Internet (MAMI), and by the Swiss State Secretariat for Education, Research, and
Innovation under contract no. 15.0268. This support does not imply endorsement.
