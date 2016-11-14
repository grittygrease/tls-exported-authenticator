---
title: Exported Authenticators in TLS
abbrev: TLS Exported Authenticator
docname: draft-sullivan-tls-exported-authenticator-latest
date: 2016
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
    organization: Cloudflare Inc.
    email: nick@cloudflare.com

normative:
  I-D.ietf-tls-tls13:
  RFC5705:
  RFC7627:

informative:



--- abstract

This document describes a mechanism in Transport Layer Security (TLS) to
provide an exportable proof of ownership of a certificate that can be
transmitted out of band and verified by the other party.

--- middle

# Introduction

This document provides a way to authenticate one party of a Transport Layer
Security (TLS) communication to another using a certificate after the session
has been established. This allows both the client and server to prove ownership
of additional identities at any time after the handshake has completed. This
proof of authentication can be exported and transmitted out of band from one
party then validated by the other party.

This mechanism is useful in the following situations:

* servers that are authoritative for multiple domains the same connection
but do not have a certificate that is simultaneously authoritative for all
of them
* servers that have resources that require client authentication to access
and need to request client authentication after the connection has started
* clients that want to assert ownership over an identity to a server after
a connection has been established

This document intends to replace much of the functionality of renegotiation
in previous versions of TLS. It has the advantages over renegotiation of not
requiring additional on-the-wire changes during a connection.

# Authenticator

Given an established TLS connection, a certificate, and a corresponding private
key, an authenticator message can be constructed by either the client or the
server. This authenticator uses the message structures from section 4.4. of
{{!I-D.ietf-tls-tls13}}, but with a different handshake context and finished key.
Unlike the Certificate and CertificateRequest messages in TLS 1.3, the messages
described in this draft are not encryped with a handshake key.

The Handshake Context is an {{!RFC5705}} (for TLS 1.2 or earlier) or {{!I-D.ietf-tls-tls13}}
exporter value derived using the label "authenticator handshake context" and
length 64 bytes. The Finished MAC Key is an exporter value derived using the label
"server authenticator finished key" or "client authenticator finished key", depending
on the sender, with length corresponding to the length of the handshake hash.

If the connection is TLS 1.2 or earlier, the master secret MUST have been computed
with the extended master secret {{!RFC7627}} to avoid key synchronization attacks.

Certificate
: The certificate to be used for authentication and any
supporting certificates in the chain.

The certificate message contains an opaque string called
certificate_request_context which MUST be unique for a given connection. Its format
should be defined by the application level protocol and MUST be non-zero
length.

CertificateVerify
: A signature over the value Hash(Handshake Context + Certificate)

Finished
: A HMAC over the value Hash(Handshake Context + Certificate + CertificateVerify)
using the hash function from the handshake and the Finished MAC Key as a key.
{:br}

The certificates used in the Certificate message must conform to the requirements
of a Certificate message in the version of TLS that is being negotiated as
described in section 4.2.3. of {{!I-D.ietf-tls-tls13}}.

The exported authenticator message is the sequence:
Certificate, CertificateVerify, Finished

# API considerations

TLS implementations supporting the use of exported authenticators MUST provide
application programming interfaces by which clients and servers may request
and verify exported authenticator messages.

Given an established connection, the application should be able to obtain an
authenticator by providing the following:

 * certificate_request_context (from 1 to 255 bytes)
 * valid certificate chain for the connection and associated extensions (OCSP, SCT, etc.)
 * signer (either the private key associated with the certificate, or interface
to perform private key operation)

Given an established connection and an exported authenticator message, the
application should be able to provide the authenticator to the connection.
If the Finished and CertificateVerify messages verify, the TLS library should
return the following:

 * certificate chain and extensions
 * certificate_request_context

In order for the application layer to communicate which certificates it will
accept, an API should be exposed that returns an array of TLS 1.3 SignatureScheme
objects that corresponds to the signature algorithms that the library is
willing to validate in an exported authenticator message.

# Security Considerations

TBD

# Acknowledgements {#ack}

Comments on this proposal were provided by Martin Thomson.

--- back
