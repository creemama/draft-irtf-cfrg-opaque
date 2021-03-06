---
title: The OPAQUE Asymmetric PAKE Protocol
abbrev: OPAQUE
docname: draft-irtf-cfrg-opaque-latest
date:
category: info

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: H. Krawczyk
    name: Hugo Krawczyk
    organization: Algorand Foundation
    email: hugokraw@gmail.com
 -
    ins: K. Lewi
    name: Kevin Lewi
    organization: Novi Research
    email: lewi.kevin.k@gmail.com
 -
    ins: C. A. Wood
    name: Christopher A. Wood
    organization: Cloudflare
    email: caw@heapingbits.net

informative:

  Boyen09:
    title: "HPAKE: Password authentication secure against cross-site user
impersonation"
    author:
      -
        ins: X. Boyen
        name: Xavier Boyen

    seriesinfo: Cryptology and Network Security (CANS)
    date: 2009

  BG04:
    title: The static Diffie-Hellman problem
    author:
      -
        ins: D. Brown
        name: Daniel R. L. Brown
      -
        ins: R. Galant
        name: Robert P. Galant

    seriesinfo: http://eprint.iacr.org/2004/306
    date: 2004

  Canetti01:
    title: "Universally composable security: A new paradigm for cryptographic
protocols"
    author:
      -
        ins: R. Canetti
        name: Ran Canetti

    seriesinfo: IEEE Symposium on Foundations of Computer Science (FOCS)
    date: 2001

  Cheon06:
    title: Security analysis of the strong Diffie-Hellman problem
    author:
      -
        ins: J. H. Cheon
        name: Jung Hee Cheon

    seriesinfo: Euroctypt 2006
    date: 2006

  FK00:
    title: Server-assisted generation of a strong secret from a password
    author:
      -
        ins: W. Ford
        name: Warwick Ford
      -
        ins: B. S. Kaliski, Jr
        name: Burton S. Kaliski, Jr

    seriesinfo: WETICE
    date: 2000

  GMR06:
    title: "A method for making password-based
key exchange resilient to server compromise"

    author:
      -
        ins: C. Gentry
        name: Craig Gentry
      -
        ins: P. MacKenzie
        name: Phil MacKenzie
      -
        ins: Z, Ramzan
        name: Zulfikar Ramzan

    seriesinfo: CRYPTO
    date: 2006

  AuCPace:
    title: "AuCPace: Efficient verifier-based PAKE protocol tailored for the IIoT"
    author:
      -
        ins: B. Haase
        name: Bjorn Haase
      -
        ins: B. Labrique
        name: Benoit Labrique

    seriesinfo: http://eprint.iacr.org/2018/286
    date: 2018

  I-D.irtf-cfrg-hash-to-curve:
  I-D.irtf-cfrg-voprf:
  I-D.sullivan-tls-opaque:

  keyagreement: DOI.10.6028/NIST.SP.800-56Ar3

  OPAQUE:
    title: "OPAQUE: An Asymmetric PAKE Protocol Secure Against Pre-Computation
    Attacks"
    # see the quotes above? Needed because of the embedded colon.
    author:
      -
        ins: S. Jarecki
        name: Stanislaw Jarecki
      -
        ins: H. Krawczyk
        name: Hugo Krawczyk
      -
        ins: J. Xu
        name: Jiayu Xu
    seriesinfo: Eurocrypt
    date: 2018

  JKKX16:
    title: "Highly-efficient and composable
password-protected secret sharing (or: how to protect your bitcoin wallet
online)"
    # see the quotes above? Needed because of the embedded colon.
    author:
      -
        ins: S. Jarecki
        name: Stanislaw Jarecki
      -
        ins: A. Kiayias
        name: Aggelos Kiayias
      -
        ins: H. Krawczyk
        name: Hugo Krawczyk
      -
        ins: J. Xu
        name: Jiayu Xu
    seriesinfo: IEEE European Symposium on Security and Privacy
    date: 2016

  LGR20:
    title: Partitioning Oracle Attacks
    target: https://eprint.iacr.org/2020/1491.pdf
    author:
      -
        ins: J. Len
        name: Julia Len
      -
        ins: P. Grubbs
        name: Paul Grubbs
      -
        ins: T. Ristenpart
        name: Thomas Ristenpart

  SIGMA:
    title: "SIGMA: The SIGn-and-MAc approach to authenticated Diffie-Hellman and its use in the IKE protocols"
    author:
      -
        ins: H. Krawczyk
        name: Hugo Krawczyk

    seriesinfo: CRYPTO
    date: 2003

  HMQV:
    title: "HMQV: A high-performance secure Diffie-Hellman protocol"
    author:
      -
        ins: H. Krawczyk
        name: Hugo Krawczyk

    seriesinfo: CRYPTO
    date: 2005

  SPAKE2plus:
    title: "Security Analysis of SPAKE2+"
    author:
      -
        ins: V. Shoup
        name: Victor Shoup

    seriesinfo: http://eprint.iacr.org/2020/313
    date: 2020

  SIGNAL:
    title: "Signal recommended cryptographic algorithms"
    seriesinfo: https://signal.org/docs/specifications/doubleratchet/#recommended-cryptographic-algorithms
    date: 2016

  RFC2945:
  RFC5869:
  RFC8125:
  RFC8446:

--- abstract

This document describes the OPAQUE protocol, a secure asymmetric
password-authenticated key exchange (aPAKE) that supports mutual
authentication in a client-server setting without reliance on PKI and
with security against pre-computation attacks upon server compromise.
In addition, the protocol provides forward secrecy and the ability to
hide the password from the server, even during password registration.
This document specifies the core OPAQUE protocol, along with several
instantiations in different authenticated key exchange protocols.

--- middle

# Introduction {#intro}

Password authentication is the prevalent form of authentication in
the web and in many other applications. In the most common
implementation, a user authenticates to a server by sending its user
ID and password to the server over a TLS connection. This makes
the password vulnerable to server mishandling, including accidentally
logging the password or storing it in cleartext in a database. Server
compromise resulting in access to these plaintext passwords is not an
uncommon security incident, even among security-conscious companies.
Moreover, plaintext password authentication over TLS is also vulnerable
to TLS failures, including many forms of PKI attacks, certificate
mishandling, termination outside the security perimeter, visibility
to middle boxes, and more.

Asymmetric (or augmented) Password Authenticated Key Exchange (aPAKE)
protocols are designed to provide password authentication and
mutually authenticated key exchange in a client-server setting without relying on PKI (except
during user/password registration) and without disclosing passwords
to servers or other entities other than the client machine. A secure
aPAKE should provide the best possible security for a password
protocol. Namely, it should only be open to inevitable attacks, such as
online impersonation attempts with guessed user passwords and offline
dictionary attacks upon the compromise of a server and leakage of its
password file. In the latter case, the attacker learns a mapping of
a user's password under a one-way function and uses such a mapping to
validate potential guesses for the password. Crucially important is
for the password protocol to use an unpredictable one-way mapping.
Otherwise, the attacker can pre-compute a deterministic list of mapped
passwords leading to almost instantaneous leakage of passwords upon
server compromise.

Despite the existence of multiple designs for
(PKI-free) aPAKE protocols, none of these protocols are secure against
pre-computation attacks. In particular, none of these protocols can
use the standard technique against pre-computation that combines
_secret_ random values ("salt") into the one-way password mappings.
Either these protocols do not use salt at all or, if they do, they
transmit the salt from server to user in the clear, hence losing the
secrecy of the salt and its defense against pre-computation. Furthermore,
transmitting the salt may require additional protocol messages.

This document describes OPAQUE, a PKI-free secure aPAKE that is secure
against pre-computation attacks and capable of using a secret salt.
OPAQUE provides forward secrecy (essential for
protecting past communications in case of password leakage) and the
ability to hide the password from the server - even during password
registration. Furthermore, OPAQUE enjoys good performance and an array of additional
features including the ability to increase
the difficulty of offline dictionary attacks via iterated hashing
or other hardening schemes, and offloading these operations to the
client (that also helps against online guessing attacks); extensibility of
the protocol to support storage and
retrieval of user's secrets solely based on a password; and being
amenable to a multi-server distributed implementation where offline
dictionary attacks are not possible without breaking into a threshold
of servers (such a distributed solution requires no change or awareness
on the client side relative to a single-server implementation).

OPAQUE is defined and proven as the composition of two functionalities:
an Oblivious PRF (OPRF) and an authenticated key-exchange (AKE) protocol. It can be seen
as a "compiler" for transforming any suitable AKE protocol into a secure
aPAKE protocol. (See {{security-considerations}} for requirements of the
OPRF and AKE protocols.) This document specifies OPAQUE instantiations based
on a variety of AKE protocols, including HMQV {{HMQV}}, 3DH {{SIGNAL}}
and SIGMA {{SIGMA}}. In general, the modularity of OPAQUE's design makes it
easy to integrate with additional AKE protocols, e.g., IKEv2, and with future
ones such as those based on post-quantum techniques.

Currently, the most widely deployed (PKI-free) aPAKE is SRP {{?RFC2945}}, which is
vulnerable to pre-computation attacks, lacks a proof of security, and is less efficient
relative to OPAQUE. Moreover, SRP requires a ring as it mixes addition and
multiplication operations, and thus does not work over plain elliptic curves. OPAQUE
is therefore a suitable replacement for applications that use SRP.

This draft complies with the requirements for PAKE protocols set forth in
{{RFC8125}}.

## Requirements Notation

{::boilerplate bcp14}

## Notation

The following terms are used throughout this document to describe the
operations, roles, and behaviors of OPAQUE:

- Client (U): Entity which has knowledge of a password and wishes to authenticate.
- Server (S): Entity which authenticates clients using passwords.
- (skX, pkX): An AKE key pair used in role X; skX is the private key and pkX is
  the public key. For example, (skU, pkU) refers to U's private and public key.
- kX: An OPRF private key used in role X. For example, kU refers to U's private OPRF
  key.
- I2OSP and OS2IP: Convert a byte string to and from a non-negative integer as
  described in {{?RFC8017}}. Note that these functions operate on byte strings in
  big-endian byte order.
- concat(x0, ..., xN): Concatenation of byte strings.
  `concat(0x01, 0x0203, 0x040506) = 0x010203040506`.
- random(n): Generate a random byte string of length `n` bytes.
- `xor(a,b)`: XOR of byte strings; `xor(0xF0F0, 0x1234) = 0xE2C4`.
  It is an error to call this function with two arguments of unequal
  length.
- `ct_equal(a, b)`: Return `true` if `a` is equal to `b`, and false otherwise.
  This function is constant-time in the length of `a` and `b`, which are assumed
  to be of equal length, irrespective of the values `a` or `b`.

Except if said otherwise, random choices in this specification refer to
drawing with uniform distribution from a given set (i.e., "random" is short
for "uniformly random"). Random choices can be replaced with fresh outputs from
a cryptographically strong pseudorandom generator, according to the requirements
in {{!RFC4086}}, or pseudorandom function.

The name OPAQUE is a homonym of O-PAKE where O is for Oblivious (the name
OPAKE was taken).

# Cryptographic Protocol and Algorithm Dependencies {#dependencies}

OPAQUE relies on the following protocols and primitives:

- Oblivious Pseudorandom Function (OPRF, {{I-D.irtf-cfrg-voprf}}):
  - Blind(x): Convert input `x` into an Element of the OPRF group, randomize it
    by some Scalar `r`, producing `M`, and output (`r`, `M`).
  - Evaluate(k, M): Evaluate input element `M` using private key `k`, yielding
    output element `Z`.
  - Unblind(r, Z): Remove random Scalar `r` from `Z`, yielding output `N`.
  - Finalize(x, N, info): Compute the OPRF output using input `x`, `N`, and domain
    separation tag `info`.

- Cryptographic hash function:
  - Hash(m): Compute the cryptographic hash of input message `m`. The type of the
    hash is determined by the chosen OPRF group.
  - Nh: The output size of the Hash function.

- Memory Hard Function (MHF):
  - Harden(msg, params): Repeatedly apply a memory hard function with parameters
    `params` to strengthen the input `msg` against offline dictionary attacks.
    This function also needs to satisfy collision resistance.

Note that we only need the base mode variant (as opposed to the verifiable mode
variant) of the OPRF described in {{I-D.irtf-cfrg-voprf}}. We also assume the
existence of a function `KeyGen` from {{I-D.irtf-cfrg-voprf}}, which
generates an OPRF private and public key. OPAQUE only requires an OPRF private key.
We write `(kU, _) = KeyGen()` to denote use of this function for generating secret key `kU`
(and discarding the corresponding public key).

# Core Protocol {#protocol}

OPAQUE consists of two stages: registration and authenticated key exchange.
In the first stage, a client registers its password with the server and stores
its encrypted credentials on the server. In the second stage, a client obtains
those credentials, unlocks them using the user's password and subsequently uses
them as input to an authenticated key exchange (AKE) protocol.

Both registration and authenticated key exchange stages require running an OPRF protocol.
The latter stage additionally requires running a mutually-authenticated
key-exchange protocol (AKE) using credentials recovered after the OPRF protocol completes.
(The key-exchange protocol MUST satisfy forward secrecy and the KCI requirement
discussed in {{security-considerations}}.)

We first define the core OPAQUE protocol based on a generic OPRF, hash, and MHF function.
{{instantiations}} describes specific instantiations of OPAQUE using various AKE protocols,
including: HMQV, 3DH, and SIGMA-I. {{I-D.sullivan-tls-opaque}} discusses integration with
TLS 1.3 {{RFC8446}}.

## Credential types and envelope construction {#data-types}

OPAQUE makes use of a structure `Envelope` to store client credentials.
The `Envelope` structure embeds the following types of credentials:

- skU: The encoded user private key for the AKE protocol.
- pkS: The encoded server public key for the AKE protocol.
- idU: The user identity. This is an application-specific value, e.g., an e-mail
  address or normal account name.
- idS: The server identity. This is typically a domain name, e.g., example.com.
  See {{SecIdentities}} for information about this identity.

Each public and private key value is an opaque byte string, specific to the AKE
protocol in which OPAQUE is instantiated. For example, if used as raw public keys
for TLS 1.3 {{?RFC8446}}, they may be RSA or ECDSA keys as per {{?RFC7250}}.

These credentials are incorporated in the `SecretCredentials` and `CleartextCredentials` structs,
depending on the mode set by the value of `EnvelopeMode`:

~~~
enum {
  base(1),
  customIdentifier(2),
  (255)
} EnvelopeMode;
~~~

The `base` mode defines `SecretCredentials` and `CleartextCredentials` as follows:

~~~
struct {
  opaque skU<1..2^16-1>;
} SecretCredentials;

struct {
  opaque pkS<1..2^16-1>;
} CleartextCredentials;
~~~

The `customIdentifier` mode defines `SecretCredentials` and `CleartextCredentials` as follows:

~~~
struct {
  opaque skU<1..2^16-1>;
} SecretCredentials;

struct {
  opaque pkS<1..2^16-1>;
  opaque idU<0..2^16-1>;
  opaque idS<0..2^16-1>;
} CleartextCredentials;
~~~

These credentials are embedded into the following `Envelope` structure with
encryption and authentication.

~~~
struct {
  InnerEnvelopeMode mode;
  opaque nonce[32];
  opaque encrypted_creds<1..2^16-1>;
} InnerEnvelope;

struct {
  InnerEnvelope contents;
  opaque auth_tag[Nh];
} Envelope;
~~~

mode
: The `EnvelopeMode` value. This MUST be one of `base` or `customIdentifier`.

nonce
: A unique 32-byte nonce used to protect this Envelope.

encrypted_creds
: Encoding of encrypted and authenticated `SecretCredentials`.

auth_tag
: Authentication tag protecting the contents of the envelope,
covering `InnerEnvelope` and `CleartextCredentials`.

The full procedure for constructing `Envelope` and `InnerEnvelope` from
`SecretCredentials` and `CleartextCredentials` is described in {{finalize-request}}.

The `EnvelopeMode` value is specified as part of the configuration (see {{configurations}}).

Credential information corresponding to the configuration-specific mode,
along with the user public key `pkU` and private key `skU`, are stored
in a `Credentials` object with the following named fields:

- `skU`, the user's private key
- `pkU`, the user's public key corresponding to `skU`
- `idU`, an optional user identity (present only in the `customIdentifier` mode)
- `idS`, an optional server identity (present only in the `customIdentifier` mode)

## Offline registration stage {#offline-phase}

Registration is executed between a user U (running on a client machine) and a
server S. It is assumed the server can identify the user and the client can
authenticate the server during this registration phase. This is the only part
in OPAQUE that requires an authenticated channel, either physical, out-of-band,
PKI-based, etc. This section describes the registration flow, message encoding,
and helper functions. Moreover, U has a key pair (skU, pkU) for an AKE protocol
which is suitable for use with OPAQUE; See {{online-phase}}. (skU, pkU) may be
randomly generated for the account or provided by the calling client.
Clients MUST NOT use the same key pair (skU, pkU) for two different accounts.

To begin, U chooses password pwdU, and S chooses its own pair of private-public
keys skS and pkS for use with the AKE. S can use the same pair of keys with
multiple users. These steps can happen offline, i.e., before the registration phase.
Once complete, the registration process proceeds as follows:

~~~
      Client (pwdU, creds)                       Server (skS, pkS)
  -----------------------------------------------------------------
 request, blind = CreateRegistrationRequest(pwdU)

                                   request
                              ----------------->

            (response, kU) = CreateRegistrationResponse(request, pkS)

                                   response
                              <-----------------

 record, export_key = FinalizeRequest(pwdU, creds, blind, response)

                                   record
                              ------------------>

~~~

Both client and server MUST validate the other party's public key before use.
See {{validation}} for more details.

### Registration messages

~~~
struct {
    SerializedElement data;
} RegistrationRequest;
~~~

data
: A serialized OPRF group element.

~~~
struct {
    SerializedElement data;
    opaque pkS<1..2^16-1>;
} RegistrationResponse;
~~~

data
: A serialized OPRF group element.

pkS
: An encoded public key that will be used for the online authenticated key exchange stage.

~~~
struct {
    Envelope envU;
    opaque pkU<1..2^16-1>;
} RegistrationUpload;
~~~

envU
: The user's `Envelope` structure

pkU
: An encoded public key, corresponding to the private key `skU`.

### Registration functions

#### CreateRegistrationRequest

~~~
CreateRegistrationRequest(pwdU)

Input:
- pwdU, an opaque byte string containing the user's password

Output:
- request, a RegistrationRequest structure
- r, an OPRF Scalar value

Steps:
1. (r, M) = Blind(pwdU)
2. Create RegistrationRequest request with M
3. Output (request, r)
~~~

#### CreateRegistrationResponse

~~~
CreateRegistrationResponse(request, pkS)

Input:
- request, a RegistrationRequest structure
- pkS, the server's public key

Output:
- response, a RegistrationResponse structure
- kU, Per-user OPRF key

Steps:
1. (kU, _) = KeyGen()
2. Z = Evaluate(kU, request.data)
3. Create RegistrationResponse response with
     (Z, pkS)
4. Output (response, kU)
~~~

#### FinalizeRequest {#finalize-request}

~~~
FinalizeRequest(pwdU, creds, blind, response)

Parameters:
- params, the MHF parameters established out of band
- mode, the InnerEnvelope mode
- Nh, the output size of the Hash function

Input:
- pwdU, an opaque byte string containing the user's password
- creds, a Credentials structure
- blind, an OPRF Scalar value
- response, a RegistrationResponse structure

Output:
- envU, the user's Envelope structure
- export_key, an additional key

Steps:
1. N = Unblind(blind, response.data)
2. y = Finalize(pwdU, N, "OPAQUE01")
3. rwdU = HKDF-Extract("rwdU", Harden(y, params))
4. Create SecretCredentials secret_creds with creds.skU
5. Create CleartextCredentials cleartext_creds with response.pkS
   and custom identifiers creds.idU and creds.idS if mode is customIdentifier
6. nonce = random(32)
7. pseudorandom_pad = HKDF-Expand(rwdU, concat(nonce, "Pad"), len(pt))
8. auth_key = HKDF-Expand(rwdU, concat(nonce, "AuthKey"), Nh)
9. export_key = HKDF-Expand(rwdU, concat(nonce, "ExportKey"), Nh)
10. encrypted_creds = xor(secret_creds, pseudorandom_pad)
11. Create InnerEnvelope contents with (mode, nonce, encrypted_creds)
12. auth_tag = HMAC(auth_key, concat(contents, cleartext_creds))
13. Create Envelope envU with (contents, auth_tag)
14. Create RegistrationUpload record with (envU, creds.pkU)
15. Output (record, export_key)
~~~

[[RFC editor: please change "OPAQUE01" to the correct RFC identifier before publication.]]

The inputs to HKDF-Extract and HKDF-Expand are as specified in {{RFC5869}}. The underlying hash function
is that which is associated with the OPAQUE configuration (see {{configurations}}).

See {{export-usage}} for details about the output export_key usage.

Upon completion of this function, the client MUST send `record` to the server.
The server then stores the tuple `(envU, pkU, kU, pkS, skS)`, where `envU` and `pkU`
are obtained from `record`. If `skS` and `pkS` are used for multiple users, the
server can store these values separately and omit them from the user's record.

## Online authenticated key exchange stage {#online-phase}

After registration, the user (through a client machine) and server run the
authenticated key exchange stage of the OPAQUE protocol. This stage is composed of a concurrent
OPRF and key exchange flow. The key exchange protocol is authenticated using the
client and server private keys established during the offline phase; see {{offline-phase}}.
The type of keys MUST be suitable for the key exchange protocol. For example, if
the key exchange protocol is 3DH, as described in {{SecHmqv}}, then the private and
public keys must be Diffie-Hellman keys. At the end, the client proves the user's
knowledge of the password, and both client and server agree on a mutually authenticated
shared secret key.

This section describes the message flow, encoding, and helper functions used in this stage.

~~~
       Client (pwdU)                    Server (skS, pkS, kU, envU)
  -----------------------------------------------------------------
   request, blind = CreateCredentialRequest(pwdU)

                                   request
                              ----------------->

   response = CreateCredentialResponse(request, pkS, kU, envU)

                                   response
                              <-----------------

  skU, pkS, export_key = RecoverCredentials(pwdU, blind, response)

                            (AKE with credentials)
                              <================>
~~~

The protocol messages below do not include the AKE protocol. Instead, OPAQUE
assumes the client and server run the AKE using the credentials recovered from
the OPRF protocol.

Note also that the authenticated key exchange stage can run the OPRF and AKE protocols
concurrently with interleaved and combined messages (while preserving the internal ordering
of messages in each protocol). In all cases, the client needs to obtain envU and
rwdU, i.e., complete the OPRF protocol, before it can use its own private key
skU and the server's public key pkS in the AKE. See {{instantiations}} for examples
of this integration.

### Authenticated key exchange messages

~~~
struct {
    opaque data<1..2^16-1>;
} CredentialRequest;
~~~

data
: A serialized OPRF group element.

~~~
struct {
    opaque data<1..2^16-1>;
    opaque pkS<1..2^16-1>;
    Envelope envU;
} CredentialResponse;
~~~

data
: A serialized OPRF group element.

pkS
: An encoded public key that will be used for the online authenticated
key exchange stage.

envU
: The user's `Envelope` structure.

### Authenticated key exchange functions

#### CreateCredentialRequest

~~~
CreateCredentialRequest(pwdU)

Input:
- pwdU, an opaque byte string containing the user's password

Output:
- request, an CredentialRequest structure
- r, an OPRF Scalar value

Steps:
1. (r, M) = Blind(pwdU)
2. Create CredentialRequest request with M
3. Output (request, r)
~~~

#### CreateCredentialResponse

~~~
CreateCredentialResponse(request, pkS, kU, envU)

Input:
- request, a CredentialRequest structure
- pkS, public key of the server
- kU, OPRF key associated with idU
- envU, Envelope associated with idU

Output:
- response, a CredentialResponse structure

Steps:
1. Z = Evaluate(kU, request.data)
2. Create CredentialResponse response with (Z, pkS, envU)
3. Output response
~~~

#### RecoverCredentials

~~~
RecoverCredentials(pwdU, blind, response)

Parameters:
- params, the MHF parameters established out of band
- Nh, the output size of the Hash function

Input:
- pwdU, an opaque byte string containing the user's password
- blind, an OPRF Scalar value
- response, a CredentialResponse structure

Output:
- secret_credentials, a `SecretCredentials` structure
- export_key, an additional key

Steps:
1. N = Unblind(blind, response.data)
2. y = Finalize(pwdU, N, "OPAQUE01")
3. contents = response.envU.contents
4. nonce = contents.nonce
5. rwdU = HKDF-Extract("rwdU", Harden(y, params))
6. pseudorandom_pad = HKDF-Expand(rwdU, concat(nonce, "Pad"), len(contents.encrypted_creds))
7. auth_key = HKDF-Expand(rwdU, concat(nonce, "AuthKey"), Nh)
8. export_key = HKDF-Expand(rwdU, concat(nonce, "ExportKey"), Nh)
9. Create CleartextCredentials cleartext_creds with response.pkS
   and custom identifiers creds.idU and creds.idS if mode is customIdentifier
10. expected_tag = HMAC(auth_key, concat(contents, cleartext_creds))
11. If !ct_equal(response.envU.auth_tag, expected_tag), raise DecryptionError
12. secret_creds = xor(contents.encrypted_creds, pseudorandom_pad)
13. Output (secret_creds.skU, response.pkS, export_key)
~~~

[[RFC editor: please change "OPAQUE01" to the correct RFC identifier before publication.]]

## Export Key {#export-usage}

OPAQUE outputs an export_key that may be used for additional
application-specific purposes. For example, one might expand the use of OPAQUE with a
credential-retrieval functionality that is separate from the contents of the `Envelope`
structure.

The exporter_key MUST NOT be used in any way before the HMAC value in the
envelope is validated. See {{envelope-encryption}} for more details about this
requirement.

## AKE Execution and Party Identities {#SecIdentities}

The AKE protocol is run as part of the online authenticated key exchange
flow described above. The AKE MUST authenticate the OPAQUE transcript, which
consists of the encoded `request` and `response` messages exchanged during the
OPRF computation and credential fetch flow.

Also, authenticated key-exchange protocols generate keys that need to be uniquely
and verifiably bound to a pair of identities. In the case of OPAQUE, those identities
correspond to idU and idS. Thus, it is essential for the parties to agree on such
identities, including an agreed bit representation of these identities as needed.

Applications may have different policies about how and when identities are
determined. A natural approach is to tie idU to the identity the server uses
to fetch envU (hence determined during password registration) and to tie idS
to the server identity used by the client to initiate an offline password
registration or online authenticated key exchange session.
idS and idU can also be part of envU or be tied to the
parties' public keys. In principle, it is possible that identities change
across different sessions as long as there is a policy that can establish if
the identity is acceptable or not to the peer. However, we note that the
public keys of both the server and the user must always be those defined at
time of password registration.

# Authenticated Key Exchange Protocol Instantiations {#instantiations}

This section describes several instantiations of OPAQUE using different AKE protocols, all of
which satisfy the forward secrecy and KCI properties discussed in {{security-considerations}}.
For the sake of concreteness it only includes AKE protocols consisting of three messages,
denoted KE1, KE2, KE3, where KE1 and KE2 include key exchange shares (DH values) sent by
client and server, respectively, and KE3 provides explicit client authentication and full
forward security (without it, forward secrecy is only achieved against eavesdroppers
which is insufficient for OPAQUE security).

As shown in {{OPAQUE}}, OPAQUE cannot use less than three messages so the 3-message
instantiations presented here are optimal in terms of number of messages.
On the other hand, there is no impediment of using OPAQUE with protocols with
more than 3 messages as in the case of IKEv2 (or the underlying SIGMA-R
protocol {{SIGMA}}).

The generic outline of OPAQUE with a 3-message AKE protocol is as follows:

- C to S: credential_request, KE1
- S to C: credential_response, KE2
- C to S: KE3

Key derivation and other details of the protocol are specified by the
KE scheme. We note that by the results in {{OPAQUE}}, KE2 and KE3 should
authenticate credential_request and credential_response, respectively,
for binding between the underlying OPRF protocol messages and the KE session.

Next, we present three instantiations of OPAQUE - with HMQV, 3DH and SIGMA-I.
{{I-D.sullivan-tls-opaque}} discusses integration with TLS 1.3 {{RFC8446}}.
Note that these instantiations transmit idU in cleartext. Applications that require
idU privacy should encrypt this appropriately. Mechanisms for doing so are
outside the scope of this document, though may be addressed elsewhere, such as
in {{I-D.sullivan-tls-opaque}}.

OPAQUE may be instantiated with any post-quantum (PQ) AKE protocol that has the message
flow above and security properties (KCI resistance and forward secrecy) outlined
in {{security-considerations}}. This document does not specify such an instantiation.
Note that such an instantiation is not quantum safe unless the OPRF and data encryption schemes
are quantum safe. However, an instantiation where both AKE and data encryption are quantum safe,
but the OPRF is not, would still ensure data security against future quantum attacks since breaking the OPRF
does not retroactively affect the security of data transferred over a quantum-safe secure channel.

## Key Schedule Utility Functions

The key derivation procedures for HMQV, 3DH, and SIGMA-I instantiations
all make use of the functions below, re-purposed from TLS 1.3 {{?RFC8446}}.

~~~
HKDF-Expand-Label(Secret, Label, Context, Length) =
  HKDF-Expand(Secret, HkdfLabel, Length)
~~~

Where HkdfLabel is specified as:

~~~
struct {
   uint16 length = Length;
   opaque label<8..255> = "OPAQUE " + Label;
   opaque context<0..255> = Context;
} HkdfLabel;

Derive-Secret(Secret, Label, Transcript) =
    HKDF-Expand-Label(Secret, Label, Hash(Transcript), Nh)
~~~

HKDF uses Hash as its underlying hash function, which is the same as that
which is indicated by the OPAQUE instantiation.

## Instantiation with HMQV and 3DH {#SecHmqv}

The integration of OPAQUE with HMQV {{HMQV}} leads to the most
efficient instantiation of OPAQUE in terms of exponentiations count.
Performance is close to optimal due to the low cost of authentication in
HMQV: Just 1/6 of an exponentiation for each party over the cost of a regular
DH exchange. However, HMQV is encumbered by an IBM patent, hence we also
present OPAQUE with 3DH which only differs in the key derivation function
at the cost of two additional exponentiations (and less resilience to the compromise
of ephemeral exponents). We note that 3DH serves as a basis for the
key-exchange protocol of {{SIGNAL}}. Importantly, many other protocols
follow a similar format with differences
mainly in the key derivation function. This includes the Noise family of
protocols. Extensions also apply to KEM-based AKE protocols as in many
post-quantum candidates.

### HMQV and 3DH protocol messages {#hmqv-3dh-protocol-messages}

HMQV and 3DH are both implemented using a suitable cyclic group of prime order p.
All operations in the key derivation steps in {{derive-hmqv}} and {{derive-3dh}}
are performed in this group and represented here using multiplicative notation.

OPAQUE with HMQV and OPAQUE with 3DH comprises:

- KE1 = credential_request, nonceU, info1*, epkU
- KE2 = credential_response, nonceS, info2*, epkS, Einfo2*, MAC(Km2; transcript2),
- KE3 = info3*, Einfo3*, MAC(Km3; transcript3)}

where:

- '\*' denotes optional elements;

- The private and public keys of the parties in these examples are
Diffie-Hellman keys, namely, pkU=g^skU and pkS=g^skS.

- credential_request and credential_response denote the online OPAQUE
protocol messages (defined in {{online-phase}}) which carry the client
and server OPRF values, respectively, as well as the envelope.

- nonceU, nonceS are fresh random nonces chosen by client and server,
respectively;

- info1, info2, info3 denote optional application-specific information sent in
the clear (e.g., they can include parameter negotiation, parameters for a
hardening function, etc.);

- Einfo2, Einfo3 denotes optional application-specific information sent
encrypted under keys Ke2, Ke3 defined below;

- epkU, epkS are Diffie-Hellman ephemeral public keys chosen by user and
server, respectively, which MUST be validated to be in the correct group
(see {{validation}});

- transcript2 includes the concatenation of the values
credential_request, nonceU, info1*, epkU, credential_response,
nonceS, info2*, epkS, Einfo2*;

- transcript3 includes the concatenation of all elements in transcript2
followed by info3*, Einfo3*;

Notes:

- The explicit concatenation of elements under transcript2 and transcript3 can be
 replaced with hashed values of these elements, or their combinations, using
 a collision-resistant hash (e.g., as in the transcript-hash of TLS 1.3 {{RFC8446}}).

- The inclusion of the values credential_request and credential_response under
 transcript2 is needed for binding the underlying OPRF execution to that of the
 AKE session. On the other hand, including envU in transcript2 is not mandatory
 for security, though done as part of including credential_response.

### HMQV and 3DH key derivation {#hmqv-key-schedule}

The above protocol requires MAC keys Km2, Km3, and optional encryption keys
Ke2, Ke3, as well as generating a session key SK which is the
AKE output for protecting subsequent traffic (or for generating further key
material). Key derivation uses HKDF {{RFC5869}} with a combination of the parties static
and ephemeral private-public key pairs and the parties' identities idU, idS.
See {{SecIdentities}} for more information about these identities.

HMQV and 3DH use the following key schedule for computing Km2, Km3, Ke2, Ke3, and SK:

~~~
  HKDF-Extract(salt=0, IKM)
      |
      +--> Derive-Secret(., "handshake secret", info) = handshake_secret
      |
      +--> Derive-Secret(., "session secret", info) = SK
~~~

From `handshake_secret`, Km2, Km3, Ke2, and Ke3 are computed as follows:

~~~
Km2 = HKDF-Expand-Label(handshake_secret, "server mac", "", Hash.length)
Km3 = HKDF-Expand-Label(handshake_secret, "client mac", "", Hash.length)
Ke2 = HKDF-Expand-Label(handshake_secret, "server enc", "", key_length)
Ke3 = HKDF-Expand-Label(handshake_secret, "client enc", "", key_length)
~~~

`key_length` is the length of the key required for the AKE handshake encryption algorithm.

Values `IKM` and `info` are defined for each instantiation in the following sections.

#### HMQV key derivation {#derive-hmqv}

The HKDF input parameter `info` is computed as follows:

~~~
info = "HMQV keys" || I2OSP(len(nonceU), 2) || nonceU
                   || I2OSP(len(nonceS), 2) || nonceS
                   || I2OSP(len(idU), 2) || idU
                   || I2OSP(len(idS), 2) || idS
~~~

Here, idU and idS are by default set to be equal to the idU and idS supplied as a
credential for the envelope; however, if no such credential was supplied,
then these values (indepedently) default to pkU and pkS instead.

Also, note that if pkU is not contained in the envelope, then it must be computed
from skU by the client.

The input parameter `IKM` is `Khmqv`, where `Khmqv` is computed by the client as follows:

~~~
1. u' = (eskU + u\*skU) mod p
2. Khmqv = (epkS \* pkS^s)^u'
~~~

Hash is the same hash function used in the main OPAQUE protocol for key derivation.
Its output length must be at least the length of the group order p.

Likewise, servers compute `Khmqv` as follows:

~~~
1. s' = (eskS + s\*skS) mod p
2. Khmqv = (epkU \* pkU^u)^s'
~~~

In both cases, `u` is computed as follows:

~~~
hashInput = I2OSP(len(epkU), 2) || epkU ||
            I2OSP(len(info), 2) || info ||
            I2OSP(len("client"), 2) || "client"
u = Hash(hashInput) mod (len(p)-1)
~~~

Likewise, `s` is computed as follows:

~~~
hashInput = I2OSP(len(epkS), 2) || epkS ||
            I2OSP(len(info), 2) || info ||
            I2OSP(len("server"), 2) || "server"
s = Hash(hashInput) mod (len(p)-1)
~~~

#### 3DH key derivation {#derive-3dh}

The HKDF input parameter `info` is computed as follows:

~~~
info = "3DH keys" || I2OSP(len(nonceU), 2) || nonceU
                  || I2OSP(len(nonceS), 2) || nonceS
                  || I2OSP(len(idU), 2) || idU
                  || I2OSP(len(idS), 2) || idS
~~~

idU and idS are set according to the same rules described in {#derive-hmqv}.

The input parameter `IKM` is `K3dh`, where `K3dh` is the concatenation of
three DH values computed by the client as follows:

~~~
K3dh = epkS^eskU || pkS^eskU || epkS^skU
~~~

Likewise, `K3dh` is computed by the server as follows:

~~~
K3dh = epkU^eskS || epkU^skS || pkU^eskS
~~~

## Instantiation with SIGMA-I {#SecSigma}

We show the integration of OPAQUE with the 3-message SIGMA-I protocol {{SIGMA}}.
This is an example of a signature-based protocol and also serves
as a basis for integration of OPAQUE with TLS 1.3 as specified in {{I-D.sullivan-tls-opaque}}.
This specification can be extended to the 4-message SIGMA-R protocol as used
in IKEv2.

### SIGMA protocol messages

OPAQUE with SIGMA-I comprises:

- KE1 = credential_request, nonceU, info1*, epkU
- KE2 = credential_response, nonceS, info2*, epkS, Einfo2*,
       Sign(skS; transcript2-), MAC(Km2; idS),
- KE3 = info3*, Einfo3*, Sign(skU; transcript3-), MAC(Km3; idU)}

See explanation of fields in {{hmqv-3dh-protocol-messages}}.
In addition, for the signed material,
transcript2- is defined similarly to transcript2, however if transcript2 includes
information that identifies the user, such information can be eliminated in
transcript2- (this is advised if signing user's identification information by
the server is deemed to have adverse privacy consequences).
Similarly, transcript3- is defined as transcript3 with server identification
information removed if so desired.

### SIGMA key derivation

The key schedule for computing Km2, Km3, Ke2, Ke3, and SK is the same as
specified in {{hmqv-key-schedule}}. The HKDF input parameter `info` is
computed as follows:

~~~
info = "SIGMA-I keys" || I2OSP(len(nonceU), 2) || nonceU
                      || I2OSP(len(nonceS), 2) || nonceS
                      || I2OSP(len(idU), 2) || idU
                      || I2OSP(len(idS), 2) || idS
~~~

idU and idS are set according to the same rules described in {#derive-hmqv}.

The input parameter `IKM` is `Ksigma`, where `Ksigma` is computed by clients
as `epkS^eskU` and by servers as `epkU^eskS`.

# Configurations {#configurations}

An OPAQUE configuration is a tuple (OPRF, Hash, MHF, AKE, EnvelopeMode). The OPAQUE OPRF protocol is
drawn from the "base mode" variant of {{I-D.irtf-cfrg-voprf}}. The following OPRF
ciphersuites supports are supported:

- OPRF(ristretto255, SHA-512)
- OPRF(decaf448, SHA-512)
- OPRF(P-256, SHA-256)
- OPRF(P-384, SHA-512)
- OPRF(P-521, SHA-512)

The OPAQUE hash function is that which is associated with the OPRF variant.
For the variants specified here, only SHA-512 and SHA-256 are supported.

The OPAQUE MHFs include Argon2 {{?I-D.irtf-cfrg-argon2}}, scrypt {{?RFC7914}},
and PBKDF2 {{?RFC2898}} with suitable parameter choices. These may be constant
values or set at the time of password registration and stored at the server.
In the latter case, the server communicates these parameters to the client during
login.

The OPAQUE AKE protocols are those which are specified in {{instantiations}}.
Future specifications (such as {{I-D.sullivan-tls-opaque}}) MAY introduce other
AKE instantiations.

The EnvelopeMode value is defined in {{data-types}}. It MUST be one of `base`
or `customIdentifier`.

[[https://github.com/cfrg/draft-irtf-cfrg-opaque/issues/60: Should we have a registry for configurations?]]

# Security Considerations {#security-considerations}

OPAQUE is defined and proven as the composition of two
functionalities: An Oblivious PRF (OPRF) and an authenticated key-exchange (AKE) protocol.
It can be seen as a "compiler" for transforming any AKE
protocol (with KCI security and forward secrecy - see below)
into a secure aPAKE protocol. In OPAQUE, the user stores a secret private key at the
server during password registration and retrieves this key each time
it needs to authenticate to the server. The OPRF security properties
ensure that only the correct password can unlock the private key
while at the same time avoiding potential offline guessing attacks.
This general composability property provides great flexibility and
enables a variety of OPAQUE instantiations, from optimized
performance to integration with TLS. The latter aspect is of prime
importance as the use of OPAQUE with TLS constitutes a major security
improvement relative to the standard password-over-TLS practice.
At the same time, the combination with TLS builds OPAQUE as a fully functional
secure communications protocol and can help provide privacy to
account information sent by the user to the server prior to authentication.

The KCI property required from AKE protocols for use with OPAQUE
states that knowledge of a party's private key does not allow an attacker
to impersonate others to that party. This is an important security
property achieved by most public-key based AKE protocols, including
protocols that use signatures or public key encryption for
authentication. It is also a property of many implicitly
authenticated protocols (e.g., HMQV) but not all of them. We also note that
key exchange protocols based on shared keys do not satisfy the KCI
requirement, hence they are not considered in the OPAQUE setting.
We note that KCI is needed to ensure a crucial property of OPAQUE: even upon
compromise of the server, the attacker cannot impersonate the user to the
server without first running an exhaustive dictionary attack.
Another essential requirement from AKE protocols for use in OPAQUE is to
provide forward secrecy (against active attackers).

## Related analysis

Jarecki et al. {{OPAQUE}} proved the security of OPAQUE
in a strong aPAKE model that ensures security against pre-computation attacks
and is formulated in the Universal Composability (UC) framework {{Canetti01}}
under the random oracle model. This assumes security of the OPRF
function and of the underlying key-exchange protocol. In turn, the
security of the OPRF protocol from {{I-D.irtf-cfrg-voprf}} is proven
in the random oracle model under the One-More Diffie-Hellman assumption {{JKKX16}}.

Very few aPAKE protocols have been proven formally, and those proven were analyzed
in a weak security model that allows for pre-computation attacks (e.g.,
{{GMR06}}). This is not just a formal issue: these protocols are
actually vulnerable to such attacks. This includes protocols that have recent
analyses in the UC model such as AuCPace {{AuCPace}} and SPAKE2+ {{SPAKE2plus}}.
We note that as shown in {{OPAQUE}}, these protocols, and any aPAKE
in the model from {{GMR06}}, can be converted into an aPAKE secure against
pre-computation attacks at the expense of an additional OPRF execution.

OPAQUE's design builds on a line of work initiated in the seminal
paper of Ford and Kaliski {{FK00}} and is based on the HPAKE protocol
of Xavier Boyen {{Boyen09}} and the (1,1)-PPSS protocol from Jarecki
et al. {{JKKX16}}. None of these papers considered security against
pre-computation attacks or presented a proof of aPAKE security
(not even in a weak model).

## Envelope encryption {#envelope-encryption}

The analysis of OPAQUE from {{OPAQUE}} requires the authenticated encryption scheme
used to produce envU to have a special property called random key-robustness
(or key-committing). This specification enforces this property by utilizing
encrypt-then-HMAC in the construction of envU. There is no option to use another
authenticated-encryption scheme with this specification. (Deviating from the
key-robustness requirement may open the protocol to attacks, e.g., {{LGR20}}.)
We remark that export_key for authentication or encryption requires no special
properties from the authentication or encryption schemes as long as export_key
is used only after the envU is validated, i.e., after the HMAC in RecoverCredentials
passes verification.

## Configuration choice

Best practices regarding implementation of cryptographic schemes
apply to OPAQUE. Particular care needs to be given to the
implementation of the OPRF regarding testing group membership and
avoiding timing and other side channel leakage in the hash-to-curve
mapping. Drafts {{I-D.irtf-cfrg-hash-to-curve}} and
{{I-D.irtf-cfrg-voprf}} have detailed instantiation and
implementation guidance.

## Static Diffie-Hellman oracles

While one can expect the practical security of the OPRF function
(namely, the hardness of computing the function without knowing the
key) to be in the order of computing discrete logarithms or solving
Diffie-Hellman, Brown and Gallant [BG04] and Cheon {{Cheon06}} show an
attack that slightly improves on generic attacks. For the case that
q-1 or q+1, where q is the order of the group G, has a t-bit divisor,
they show an attack that calls the OPRF on 2^t chosen inputs and
reduces security by t/2 bits, i.e., it can find the OPRF key in time
2^{q/2-t/2} and 2^{q/2-t/2} memory. For typical curves, the attack
requires an infeasible number of calls and/or results in insignificant
security loss (\*). Moreover, in the OPAQUE application, these
attacks are completely impractical as the number of calls to the function
translates to an equal number of failed authentication attempts by a
_single_ user. For example, one would need a billion impersonation attempts
to reduce security by 15 bits and a trillion to reduce it by 20 bits - and
most curves will not even allow for such attacks in the first place
(note that this theoretical loss of security is with respect to computing
discrete logarithms, not in reducing the password strength).

(\*) Some examples (courtesy of Dan Brown): For P-384, 2^90 calls reduce
security from 192 to 147 bits; for NIST P-256 the options are 6-bit
reduction with 2153 OPRF calls, about 14 bit reduction with 187 million
calls and 20 bits with a trillion calls. For Curve25519, attacks are
completely infeasible (require over 2^100 calls) but its twist form allows
an attack with 25759 calls that reduces security by 7 bits and one with
117223 calls that reduces security by 8.4 bits.

## Input validation {#validation}

Both client and server MUST validate the other party's public key(s) used
for the execution of OPAQUE. This includes the keys shared during the
offline registration phase, as well as any keys shared during the online
key agreement phase. The validation procedure varies depending on the
type of key. For example, for OPAQUE instantiations
using 3DH with P-256, P-384, or P-521 as the underlying group, validation
is as specified in Section 5.6.2.3.4 of {{keyagreement}}. This includes
checking that the coordinates are in the correct range, that the point
is on the curve, and that the point is not the point at infinity.
Additionally, validation MUST ensure the Diffie-Hellman shared secret is
not the point at infinity. For X25519 and X448, validation is as described in
{{?RFC7748}}. In particular, where applicable, endpoints MUST check whether
the Diffie-Hellman shared secret is the all-zero value and abort if so.

## User authentication versus Authenticated Key Exchange

OPAQUE provides PAKE (password-based authenticated key exchange)
functionality in the client-server setting. While in the case of user
identification, wherein the focus is often on authentication, we stress
that the key exchange element is essential. Indeed, in most cases,
user authentication enforces some policy, and the key exchange step
is essential for binding this enforcement to the authentication step.
Skipping the key exchange part is analogous to carefully checking a
visitor's credential at the door and then leaving the door open for
others to enter freely.

## OPRF hardening

Hardening the output of the OPRF greatly increases the cost of an offline
attack upon the compromise of the password file at the server. Applications
SHOULD select parameters that balance cost and complexity.

## User and server identities

The user identity (idU) and server identity (idS) are optional parameters
which are left to the application to designate as monikers for the client
and server. If the application layer does not supply values for these
parameters, then they will be omitted from the creation of the envelope
during the registration stage. Furthermore, they will be substituted with
idU = pkU and idS = pkS during the authenticated key exchange stage.

The advantage to supplying a custom idU and idS (instead of simply relying
on a fallback to pkU and pkS) is that the client can then ensure that any
mappings between idU and pkU (and idS and pkS) are protected by the
authentication from the envelope. Then, the client can verify that the
idU and idS contained in its envelope matches the idU and idS supplied by
the server.

However, if this extra layer of verification is unnecessary for the
application, then simply leaving idU and idS unspecified (and using pkU and
pkS instead) is acceptable.

<!-- TODO(caw): bring this back after updating later -->

<!-- ## Envelope considerations

It is possible to dispense with encryption in the construction of envU to
obtain a shorter envU (resulting in less storage at the server and less
communication from server to client). The idea is to derive skU from rwdU.
However, for cases where skU is not a random string of a given length, we
define a more general procedure. Namely, what is derived from rwdU is a random
seed used as an input to a key generation procedure that generates the pair
(skU, pkU). In this case, secret_credentials is empty and cleartext_credentials
contains pkS. The random key generation seed is defined as
HKDF-Expand(KdKey; info="KG seed", L)
where L is the required seed length. We note that in this encryption-less
scheme, the authentication still needs to be random-key robust which HMAC
satisfies. -->

<!--
Mention advantage of avoidable equivocable encryption? Still needs equivocable
authentication, but that one gets by modeling HMAC as programmable RO - check.
-->

<!-- To further minimize storage space, the server can derive per-user OPRF keys
kU from a single global secret key, and it can use the same pair
(skS,pkS) for all users. In this case, the per-user OPAQUE storage
consists of pkU and HMAC(Khmac; pkS), a total of 64-byte overhead with a
256-bit curve and hash. envU communicated to the user is of the same length,
consisting of pkS and HMAC(Khmac; pkS). -->

<!-- Can provide AuCPace paper (sec 7.7) as reference to importance of small
envU (for settings where storage and/or communication is expensive) -->

## User enumeration {#SecEnumeration}

User enumeration refers to attacks where the attacker tries to learn
whether a given user identity is registered with a server. Preventing
such attack requires the server to act with unknown user identities
in a way that is indistinguishable from its behavior with existing
users. Here we suggest a way to implement such defense, namely, a way for
simulating the values beta and envU for non-existing users.
Note that if the same pair of user identity idU and value alpha is received
twice by the server, the response needs to be the same in both cases (since
this would be the case for real users).
For protection against this attack, one would apply the encryption function in
the construction of envU to all the key material in envU, namely,
cleartext_credentials will be empty.
The server S will have two keys MK, MK' for a PRF f
(this refers to a regular PRF such as HMAC or CMAC).
Upon receiving a pair of user identity idU and value alpha for a non-existing
user idU, S computes kU=f(MK; idU) and kU'=f(MK'; idU) and responds with
values beta=alpha^kU and envU, where the latter is computed as follows.
rwdU is set to kU' and AEenv is set to the all-zero string (of the
length of a regular envU plaintext). Care needs to be taken to avoid side
channel leakage (e.g., timing) from helping differentiate these
operations from a regular server response.
The above requires changes to the server-side implementation but not to the
protocol itself or the client side.

There is one form of leakage that the above allows and whose prevention would
require a change in OPAQUE.
Note that an attacker that tests a idU (and same alpha) twice and receives
different responses can conclude that either the user registered with the
service between these two activations or that the user was registered before
but changed its password in between the activations (assuming the server
changes kU at the time of a password change). In any case, this
indicates that idU is a registered user at the time of the second activation.
To conceal this information, S can implement the derivation of kU
as kU=f(MK; idU) also for registered users. Hiding changes in envU, however,
requires a change in the protocol. Instead of sending envU as is,
S would send an encryption of envU under a key that the user derives from the
OPRF result (similarly to rwdU) and that S stores during password
registration. During the authenticated key exchange stage, the user will derive
this key from the OPRF result, will use it to decrypt envU, and continue with the
regular protocol. If S uses a randomized encryption, the encrypted envU will look
each time as a fresh random string, hence S can simulate the encrypted envU also
for non-existing users.

Note that the first case above does not change the protocol so its
implementation is a server's decision (the client side is not changed).
The second case, requires changes on the client side so it changes OPAQUE
itself.

[[https://github.com/cfrg/draft-irtf-cfrg-opaque/issues/22: Should this variant be documented/standardized?]]

## Password salt and storage implications

In OPAQUE, the OPRF key acts as the secret salt value that ensures the infeasibility
of pre-computation attacks. No extra salt value is needed. Also, clients never
disclose their password to the server, even during registration. Note that a corrupted
server can run an exhaustive offline dictionary attack to validate guesses for the user's
password; this is inevitable in any aPAKE protocol. (OPAQUE enables a defense against such
offline dictionary attacks by distributing the server so that an offline attack is only
possible if all - or a minimal number of - servers are compromised {{OPAQUE}}.)

Some applications may require learning the user's password for enforcing password
rules. Doing so invalidates this important security property of OPAQUE and is
NOT RECOMMENDED. Applications should move such checks to the client. Note that
limited checks at the server are possible to implement, e.g., detecting repeated
passwords.

# IANA Considerations

This document makes no IANA requests.

--- back

# Acknowledgments

The OPAQUE protocol and its analysis is joint work of the author with Stas
Jarecki and Jiayu Xu. We are indebted to the OPAQUE reviewers during CFRG's
aPAKE selection process, particularly Julia Hesse and Bjorn Tackmann.
This draft has benefited from comments by multiple people. Special thanks
to Richard Barnes, Dan Brown, Eric Crockett, Paul Grubbs, Fredrik Kuivinen,
Kevin Lewi, Payman Mohassel, Jason Resch, Greg Rubin, Nick Sullivan.
