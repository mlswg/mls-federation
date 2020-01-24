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
    organization: Wire
    email: raphael@wire.com
 

informative:

  MLSARCH:
       title: "Messaging Layer Security Architecture"
       date: 2018
       author:
         -  ins: E. Omara
            name: Emad Omara
            organization: Google
            email: emadomara@google.com
         -  
            ins: R. Barnes
            name: Richard Barnes
            organization: Cisco
            email: rlb@ipv.sx
         -
	    ins: E. Rescorla 
            name: Eric Rescorla
            organization: Mozilla 
            email: ekr@rtfm.com
         -
            ins: S. Inguva 
            name: Srinivas Inguva 
            organization: Twitter 
            email: singuva@twitter.com
         -
            ins: A. Kwon 
            name: Albert Kwon
            organization: MIT 
            email: kwonal@mit.edu
         -
            ins: A. Duric 
            name: Alan Duric
            organization: Wire 
            email: alan@wire.com 


  MLSPROTO:
       title: "Messaging Layer Security Protocol"
       date: 2018
       author:
         -  ins: R. Barnes
            name: Richard Barnes
            organization: Cisco
            email: rlb@ipv.sx
         -
            ins: J. Millican
            name: Jon Millican
            organization: Facebook
            email: jmillican@fb.com
         -
            ins: E. Omara
            name: Emad Omara
            organization: Google
            email: emadomara@google.com
         -
            ins: K. Cohn-Gordon
            name: Katriel Cohn-Gordon
            organization: University of Oxford
            email: me@katriel.co.uk
         -
            ins: R. Robert
            name: Raphael Robert
            organization: Wire
            email: raphael@wire.com

  KeyTransparency:
       target: https://KeyTransparency.org
       title: Key Transparency
       author:
       -
          ins: Google


--- abstract

This document describes how Messaging Layer Security (MLS) can be used in a federated environment where different MLS
implementations can interoperate by defining the message format for client key retrieval. The document also describes some use
cases where federation could be useful.


--- middle

# Introduction

The MLS Architecture draft {{MLSARCH}} describes the overall MLS system architecture
assuming the client and servers (Delivery Service and Authentication Service) are operated
by the same entity. This document describes the minimum changes needed to allow different
MLS clients operated by the same or different entities to communicate with each other. It also
describes the use cases where federation could be useful.
 

The focus of this document will be the interaction between the client and the Delivery Service,
specifically how the client retrieves the identityKey and InitKeys for another client.
There is no changes needed for the Authentication Service. 

Discovering which Delivery service the client communicates with is out of the scope of this document.


The below diagram shows an MLS group where all clients are operated under the same delivery service:

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

one possible environment is to have different client implementations operated by the same delivery service,
which will look like the diagram above, another environment is to have different or same clients operated
by different delivery services:

~~~~
           +------------------+      +------------------+
          + Delivery Service 1 +    + Delivery Service 2 +
          +                    +    +                    +
           +------------------+      +-------+----------+
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

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all
capitals, as shown here.

Client:
: An agent that uses this protocol to establish shared cryptographic
  state with other clients.  A client is defined by the
  cryptographic keys it holds.  An application or user may use one client
  per device (keeping keys local to each device) or sync keys among
  a user's devices so that each user appears as a single client.

Client Init Key:
: A short-lived HPKE {{!I-D.irtf-cfrg-hpke}} key pair used to introduce a new
  client to a group.  Initialization keys are published for
  each client (ClientInitKey).

Identity Key:
: A long-lived signing key pair used to authenticate the sender of a
  message.

We use the TLS presentation language {{!RFC8446}} to
describe the structure of protocol messages.

# Use cases

## Different Delivery Servers
Different applications operated by different entities can use MLS to exchange end-to-end encrypted messages.
For example in email applications, clients of email1.com can encrypt and decrypt end-to-end encrypted email messages from email2.com. 


## Different client applications
Different client applications operated by the same server can use MLS to exchange end-to-end encrypted handshake and application messages.
For example different browsers can implement the MLS protocol, and web developers write web applications that use the MLS 
implementation in the browser to encrypt and decrypt the messages. This will require a new standard Web API to allow the
client applications to set the address of the delivery service in the browser. A more concrete example is using MLS in the browser
to negotiate SRTP keys for multi-party conference calls.


# Functional Requirements

## Delivery service
In a federated environment, the different members of a group might use different Delivery Services. Each client SHOULD only connect to its
respective Delivery Service, which in turn will connect to other Delivery Services to relay messages.

One Delivery Service MUST be responsible for handshake message ordering at any given point in time, since TreeKEM requires handshake
messages to have a total order. It MUST be clear to all clients and Delivery Services of the group which Delivery Service is responsible.
The protocol between different delivery services is out of the scope of this document.

~~~~
                               +-----------------+          +---------+
                         +--> + Delivery Service B + +---> + Client B1 +
                         |    +                    +        +---------+
                         |     +-----------------+                     
                         |                                             
                     +---+-------------+                    +---------+
 +---------+        + Delivery Service A + +-------------> + Client A2 +
+ Client A1 + +---> +                    +                  +---------+
 +---------+         +------+----------+                                  
                         |                                             
                         |     +-----------------+          +---------+
                         +--> + Delivery Service C + +---> + Client C1 +
                              +                    +        +---------+
                               +-----------------+                     
                                                                                                                                                                                                                                                    
~~~~

OPEN QUESTION: How server assist could be used with multiple servers? How is the server state shared and synced ?

## Authentication Service
There is no change needed for the Authentication Service. However, authentication in a federated environment becomes more important.
The ideal solution would be using a shared transparency log like {{KeyTransparency}}.

# Message format
The encrypted message payload is defined in the MLS protocol document {{MLSPROTO}}, in order to get federation between different systems, 
the identity key and client init key retrieval MUST be defined as well. The identity key can always be included in the client init key response.

~~~~~
enum {
	P256_SHA256_AES128GCM(0x0000),
	X25519_SHA256_AES128GCM(0x0001),
	(0xFFFF)
} CipherSuite;

struct {
   opaque identity<0..2^16-1>;
   CipherSuite supported_suites<0..255>;
}  GetClientInitKeyRequest;

struct {
	opaque client_init_key_id<0..255>;
	CipherSuite cipher_suites<0..255>;
	HPKEPublicKey init_keys<1..2^16-1>;
	Credential credential;
	opaque signature<0..2^16-1>;
} ClientInitKey;

struct {
	opaque identity<0..2^16-1>;
	ClientInitKey client_init_key;
} ClientInitKeyBundle;
~~~~~

The delivery service will return one or more client init key bundles, one for each member.

~~~~~
struct {
   CLientInitKeyBundle client_init_keys<0..2^32-1>;
}  GetClientInitKeyResponse;
~~~~~

OPEN QUESTION: What if different clients have different cipher suites?

# Security Considerations

## Version negotiation
In a federated environment, version negotiation is more critical, to avoid forcing a downgrade attack by
malicious 3rd party delivery services. The negotiation could either be done in the ClientInitKeyBundle or
in a separate handshake message.

# IANA Considerations
This document makes no requests of IANA.
