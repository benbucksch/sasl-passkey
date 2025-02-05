---
###
# Internet-Draft Markdown Template
title: "SASL Passkey"
abbrev: "sasl-passkey"
category: info

docname: draft-bucksch-sasl-passkey-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: AREA
workgroup: kitten
keyword:
 - sasl
 - passkey
 - login
 - authentication
venue:
  group: kitten
  type: Working Group
  mail: kitten@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/kitten/
  github: benbucksch/sasl-passkey
  latest: https://example.com/LATEST

author:
 -
    fullname: Ben Bucksch
    organization: Beonex
    email: ben.bucksch@beonex.com
 -
    fullname: Stephen Farrell
    organization: Trinity College Dublin
    email: stephen.farrell@cs.tcd.ie

normative:

informative:


--- abstract

Introduces a SASL mechanism that allows the user to authenticate using a FIDO2 Passkey.

--- middle

# Introduction

Introduces a SASL mechanism that allows the user to authenticate using a FIDO2 Passkey.

The client/server exchange is a simple challenge-response mechanism,
using the same mechanism that browsers use when they authenticate
using Passkeys to a website.

# Creation of the Passkey

The Passkey has to be created on the user's device via other means.
Signup and passkey creation happens, for example, at the normal
web frontend of the site.

We assume that the browser will use the same authenticator (OS functions) as the authenticating application. The authenticator
is responsible for creating and storing the Passkey. The
authenticator may also sync the Passkey between the user's devices.
The Passkey is never seen by either browser nor the authenticating
application, but managed entirely by the Passkey manager.

## Initial Auth using Passkey

1. The authenticating application has the target server hostname and
   authorization identity (e.g. username or email address) configured.
SASL `PASSKEY` mechanism supports authenticating as one identity ("authentication identity")
and then (optionally) authorizing as another.

If the
target server is an IMAP server, the username is the email address. If the
target server is an XMPP server, the username is the XMPP address of the user.

2. The authenticating application opens (or reuses existing) connection to the
   target server and starts authentication using the SASL `PASSKEY` mechanism.
`PASSKEY` mechanism starts with the client sending the initial client response,
which has the following format defined using ABNF:

~~~~ abnf
passkey-client-step1 = [authorization_id]
authorization_id     = 1*OCTET
~~~~

3. The server generates a Passkey challenge, based on the
  target server hostname,
  and sends the server challenge to the client. The challenge
  doesn't depend on the passkey to be used by the user.

4.
The authenticating application (SASL client) takes the received passkey challenge and passes it
on as-is to the OS authenticator API, which returns the response.
The OS calls are the same that the web browser would do.

As part of this process, the OS authenticator API may require
the end user to complete additional authentication, for example
entering a device unlock code, providing a fingerprint,
face recognition, or similar. This is the responsibility of the
OS authenticator and outside the scope of this protocol.

The authenticating application then passes on the response
as-is to the server.

5.
  a. If the response is invalid, the server responds with a
  SASL error and a human-readable error message for the end user.

  b. If the server accepts the response as valid, it can derive
  the authentication identity from the received response.
  If login as the derived authentication identity is forbidden,
  the server will return a
  SASL error. A human-readable error message for end users
  must be included, with a detailed and helpful description of why
  login is forbidden for that user, and instructions for the user
  how the situation can be remedied.

  c. If the server accepts the login attempt from the derived
  authentication identity, it then checks
  whether or not the provided authorization identity is allowed
  to act as the authentication identity associated with the passkey
  being used by the user. If such authorization is granted
  (in particular if authorization and authentication identities
  refer to the same identity, or authorization identity is omitted)
  it responds with a SASL success response. The user is logged in.
  Otherwise the server responds with a
  SASL error and a human-readable error message for the end user.

~~~~ abnf
server-final-message = server-error "," server-error-message
        ; Only returned on error. Omitted on success.

server-error = "e=" server-error-value

server-error-value = "invalid-encoding" /
                     "unknown-user" /
                     "invalid-username-encoding" /
                       ; invalid username encoding (invalid UTF-8 or
                       ; SASLprep failed)
                     "other-error" /
                     server-error-value-ext
        ; Unrecognized errors should be treated as "other-error".
        ; In order to prevent information disclosure, the server
        ; may substitute the real reason with "other-error".

server-error-value-ext = value
        ; Additional error reasons added by extensions
        ; to this document.

server-error-message = "m=" server-error-message-value

server-error-message-value = 1*OCTET
        ; Human readable error message in UTF-8
~~~~

This SASL mechanims will typically be combined with SASL chain
or SASL2, to allow re-opening a new connection without requiring
the user to go through Passkey authentication again.

# IMAP Example

In IMAP, the exchange would be:

~~~~
S: * OK ACME IMAP Server v1.23 is ready
C: 22 CAPABILITY
S: * CAPABILITY IMAP4rev1 IMAP4rev2 AUTH=PASSKEY AUTH=REMEMBERME
S: 22 CAPABILITY completed
C: 23 AUTHENTICATE PASSKEY eW91QGV4YW1wbGUuY29tCg==
S: AEC6576576557=== (passkey challenge)
C: EAB675757GJvYgB== (passkey response)
S: 23 OK AUTHENTICATE completed
~~~~

Where "eW91QGV4YW1wbGUuY29tCg==" is base64-encoded authorization identity
("you@example.com"), "AEC6576576557===" is base64-encoded passkey challenge,
"EAB675757GJvYgB==" is base64-encoded passkey response.  All challenge and
responses values are base64-encoded according to the IMAP SASL protocol
profile.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

It's all about security.

# IANA Considerations

IANA is requested to add the following entry to the SASL Mechanism registry
established by {{!RFC4422}}:

~~~~
To: iana@iana.org
Subject: Registration of a new SASL mechanism PASSKEY

SASL mechanism name (or prefix for the family): PASSKEY
Security considerations: Section YY of [RFCXXXX]
Published specification (optional, recommended): [RFCXXXX]
Person & email address to contact for further information:
    IETF Kitten WG <kitten@ietf.org>
Intended usage: COMMON
Owner/Change controller: IESG <iesg@ietf.org>
Note:
~~~~

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
