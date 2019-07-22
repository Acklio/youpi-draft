---
stand_alone: true
ipr: trust200902
docname: draft-petrov-t2trg-youpi-latest
cat: info
pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
  toc: 'yes'
title: YANG Object Universal Parsing Interface
abbrev: YOUPI
rg: t2trg Research Group
author:
- ins: I. Petrov
  name: Ivaylo Petrov
  org: Acklio
  street: 2bis rue de la Chataigneraie
  city: 35510 Cesson-Sevigne Cedex
  country: France
  email: ivaylo@ackl.io
  role: editor
normative:
  RFC2119:
  RFC8174:
  RFC7950:
informative:

--- abstract

YANG Object Universal Parsing Interface (YOUPI) specification describes generic
way to encode and decode binary data based on a YANG model for use of
constrainted devices. YOUPI is a generic mechanism designed for great
flexibility, so that it can be adapted for any of the constainted devices.

--- middle

# Introduction {#introduction}

A huge number of very constraint IoT devices are expected to be coming to the
market. They are very constraint in terms of the MTU (sometimes as small as 10b
per message). As they are expected to be running for many years without the
need for external energy, energy efficiency which is directly linked to the
size of the payloads that need to be sent, is also very important. For those
devices JSON and even CBOR formats might be too wasteful in terms of payload
size. The reality of the ecosystem is that currently a great number of
applications use proprietary binary formats for exchanging information. A
significant problem exists if those systems are to be interacting in a purely
M2M fashion. While there are a number of possibilities to resolve those issues,
due to the constraints it is mandatory to have a way to extract and encode
information from/to the binary payload and be able to annotate it with semantic
metadata.

While binary formats can be rather complicated to parse and sometimes even
context dependent (some entity needs to keep context in order to parse a
message), for most cases a simple description format could be sufficient.

A good solution should not be bounded to the output format. It should be a data
modeling language like YANG {{RFC7950}} that simply describes the structure of
the obtained data and that allows different serialization formats afterwards.

## Terminology {#terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}}
when, and only when, they appear in all capitals, as shown here.

