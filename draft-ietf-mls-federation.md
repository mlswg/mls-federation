---
title: The Messaging Layer Security (MLS) Federation
abbrev: MLS Federation
docname: draft-ietf-mls-federation-latest
category: info

ipr: trust200902
area: Security
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: E. Omara
    name: Emad Omara
    organization: Google
    email: emadomara@google.com
 -
    ins: R. Robert
    name: Raphael Robert
    organization: 
    email: ietf@raphaelrobert.com

--- abstract

This document describes how the Messaging Layer Security (MLS) protocol can be
used in a federated environment.

--- middle

# Introduction

MLS Architecture draft {{!I-D.ietf-mls-architecture}} describes the overall MLS
system architecture, assuming the client and servers (Delivery Service and
Authentication Service) are operated by the same entity. This is however not a
strict requirement, the MLS protocol does not have an inherent dependency on one
single entity and can instead be used between multiple entities.

This document describes the minimum changes needed to allow different MLS
clients operated by the same or different entities to communicate with each and
explaining the use cases where federation could be useful.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

Client: 
: An agent that uses the MLS protocol to establish shared cryptographic
state with other clients.  A client is defined by the cryptographic keys it
  holds.  An application or user may use one client per device (keeping keys
  local to each device) or sync keys among a user's devices so that each user
  appears as a single client.

Key package: 
: A short-lived HPKE {{!I-D.irtf-cfrg-hpke}} key pair used to
introduce a new client to a group.  Initialization keys are published for each
  client (KeyPackage).

Identity Key:
: A long-lived signing key pair used to authenticate the sender of a
  message.

We use the TLS presentation language {{!RFC8446}} to
describe the structure of protocol messages.

# Federated environments

Federated environments are environments where multiple entities are operating
independent MLS services. In particular, the assumption is that Delivery
Services and Authentication Services are not necessarily operated by the same
entity.

The focus of this document will be on the different components of the MLS
architecture.

## Delivery Services

The below diagram shows an MLS group where all clients are operated under the
same deliver service:

~~~~
                       +------------+
                      + Delivery     +
                      + Service (DS) +
                       +-----+------+
                    /        +        \             Group
*********************************************************
*                 /          +          \               *
*                /           |           \              *
*      +--------+       +----+---+       +--------+     *
*     + Client 0 +     + Client 1 +     + Client 3 +    *
*      +--------+       +--------+       +--------+     *
*     .............................     ............    *
*     User 0                            User 1          *
*                                                       *
*********************************************************


~~~~

One possible environment is to have different client implementations operated by
the same delivery service, which will look like the diagram above, another
environment is to have different or same clients operated By different delivery
services:

~~~~
           +-----------------+      +-----------------+
          + Deliver Service 1 +    + Deliver Service 2 +
          +                   +    +                   +
           +-----------------+      +--------+--------+
               |         |                   |
               |         |                   |      Group
***************|*********|*******************|***********
*              |         |                   |          *
*              |         |                   |          *
*      +--------+       +--------+       +--------+     *
*     + Client 0 +     + Client 1 +     + Client 3 +    *
*      +--------+       +--------+       +--------+     *
*     .............................     ............    *
*     User 0                            User 1          *
*                                                       *
*********************************************************

~~~~

### Key packages

Key packages (as definded in {{!I-D.ietf-mls-protocol}}) are used to add new
clients to a group and can thus be considered a bootstraping mechanism for the
protocol. 

## Authentication Service

In a federated environment, authentication becomes more important. While the
sepcifics of an Authentication Service are out-of-scope for MLS in general, it
is important that strong authentication is accessible to all clients of a
federated environment. As an example, a shared transparency log like
{{KeyTransparency}} could be used.

# Further considerations

The following aspects of federated communication are not considered in this
document:

 - a common format used for the content application messages
 - a network protocol used between different server instances
 - a discovery service to discover clients/users on a specific domain

# Use cases

## Different Delivery Servers

Different applications operated by different entities can use MLS to exchange
end-to-end encrypted messages. For example, in a messaging applications, clients
of messaging1.tld can encrypt and decrypt end-to-end encrypted messages from
messaging2.tld.


## Different client applications

Different client applications operated by the same server can use MLS to
exchange end-to-end encrypted handshake and application messages. For example,
different browsers can implement the MLS protocol, and web developers write web
applications that use the MLS implementation in the browser to encrypt and
decrypt the messages. This will require a new standard Web API to allow the
client applications to set the address of the delivery service in the browser. A
more concrete example is using MLS in the browser to negotiate SRTP keys for
multi-party conference calls.

# Functional Requirements

## Delivery service

In a federated environment, the different members of a group might use different
Delivery Services. Each client SHOULD only connect to its respective Deliver
Service, which in turn will connect to other Delivery Services to relay
messages.

One Delivery Service MUST be responsible for handshake message ordering at any
given point in time, since TreeKEM requires handshake messages to have a total
order. It MUST be clear to all clients and Delivery Services of the group which
Delivery Service is responsible. The different Delivery Services can elect a
dedicated Delivery Service to be responsible for ordering handshake messages for
a certain period of time. The protocol between different delivery services is
out of the scope of this document.

~~~~
                               +-----------------+         +---------+
                         +--> + Deliver Service B + +---> + Client B1 +
                         |    +                   +        +---------+
                         |     +-----------------+
                         |
                     +---+-------------+                   +---------+
 +---------+        + Deliver Service A + +-------------> + Client A2 +
+ Client A1 + +---> +                   +                  +---------+
 +---------+         +------+----------+
                         |
                         |     +-----------------+         +---------+
                         +--> + Deliver Service C + +---> + Client C1 +
                              +                   +        +---------+
                               +-----------------+
                                                                                                                                      
~~~~

## Authentication Service

In a federated environment, authentication becomes more important. While the
sepcifics of an Authentication Service are out-of-scope for MLS in general, it
is important that strong authentication is accessible to all clients of a
federated environment. As an example, a shared transparency log like
{{KeyTransparency}} could be used.

# Security Considerations

## Version  & ciphersuite negotiation

In a federated environment, version & ciphersuite negotiation is more critical,
to avoid forcing a downgrade attack by malicious third party Delivery Services.
The negotiation can be done at the KeyPackage level.

# IANA Considerations

This document makes no requests of IANA.
