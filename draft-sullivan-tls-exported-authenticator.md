---
title: Exported Authenticators in TLS
abbrev: TLS Exported Authenticator
docname: draft-sullivan-tls-exported-authenticator
date: 2016-10
category: std

ipr:
area: Security
workgroup: TLS
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -  ins: N. Sullivan
    name: Nick Sullivan
    organization: CloudFlare Inc.
    email: nick@cloudflare.com

normative:
  I-D.ietf-tls-tls13:

informative:



--- abstract

This document describes a mechanism in Transport Layer Security (TLS) to provide an exportable proof of ownership of a certificate that can be transmitted out of band and verified by the other party.

--- middle

# Introduction

This document provides a way to authenticate one party of a Transport Layer
Security (TLS) communication to another using a certificate after the session
has been established. This allows both the client and server to prove ownership
of additional identities at any time after the handshake has completed. This
proof of authentication can be exported and transmitted out of band from one
party then validated by the other party.

This mechanism is useful in the following situations:
* servers that have the ability to serve requests from multiple domains over
the same connection but do not have a certificate that is simultaneously
authoritative over all of them
* servers that have resources that require client authentication to access
and need to request client authentication after the connection has started
* clients that want to assert their identity to a server after a connection
has been established
* clients that wish to ask a server to authenticate for a new domain not
covered by the initial connection certificate 

This document intends to replace much of the functionality of renegotiation
in previous versions of TLS. It has the advantages over renegotiation of not
requiring additional on-the-wire changes during a connection.

# Authenticator

Given an established TLS connection, a certificate and a corresponding private
key, an authenticator message can be constructed. This authenticator is
uses the standard data structures from the TLS Authentication Messages,
but with different encryption keys.

Certificate
: The certificate to be used for authentication and any
supporting certificates in the chain. It should contain a unique
certificate_request_context defined by the application.

CertificateVerify
: A signature over the value Hash(Handshake Context + Certificate)

Finished
: A MAC over the value Hash(Handshake Context + Certificate + CertificateVerify)
using a MAC key derived from the base key.
{:br}

The following table defines the Handshake Context and MAC Base Key
for each scenario:

| Mode | Handshake Context | Base Key |
|------|-------------------|----------|
| Server | ClientHello ... later of EncryptedExtensions/CertificateRequest | [sender]_authenticator_secret |
| Client | ClientHello ... ServerFinished     | [sender]_authenticator_secret |

The exported authenticator message is the sequence:
Certificate, CertificateVerify, Finished

This message is protected with keys derived from the [sender]_authenticator_secret.

The [sender] in this table denotes the sending side.

The [sender]_authenticator_secret is derived during the key schedule as follows:
~~~~
            Master Secret
                 |
                 +---------> Derive-Secret(., "client authentictor secret",
                 |                         ClientHello...ServerFinished)
                 |                         = client_authenticator_secret
                 |
                 +---------> Derive-Secret(., "server authenticator secret",
                                          ClientHello...ClientFinished)
                                          = server_handshake_traffic_secret
~~~~


# Acknowledgements {#ack}
--- back
