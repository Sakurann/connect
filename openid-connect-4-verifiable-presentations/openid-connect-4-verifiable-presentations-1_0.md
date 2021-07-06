%%%
title = "OpenID Connect for Verifiable Presentations"
abbrev = "openid-4-vp"
ipr = "none"
workgroup = "connect"
keyword = ["security", "openid", "ssi"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-connect-4-verifiable-presentations-1_0-01"
status = "standard"

[[author]]
initials="O."
surname="Terbu"
fullname="Oliver Terbu"
organization="ConsenSys Mesh"
    [author.address]
    email = "oliver.terbu@mesh.xyz"

[[author]]
initials="T."
surname="Lodderstedt"
fullname="Torsten Lodderstedt"
organization="yes.com"
    [author.address]
    email = "torsten@lodderstedt.net"

[[author]]
initials="K."
surname="Yasuda"
fullname="Kristina Yasuda"
organization="Microsoft"
    [author.address]
    email = "kristina.yasuda@microsoft.com"

[[author]]
initials="A."
surname="Lemmon"
fullname="Adam Lemmon"
organization="Convergence.tech"
    [author.address]
    email = "adam@convergence.tech"
    
[[author]]
initials="T."
surname="Looker"
fullname="Tobias Looker"
organization="Mattr"
    [author.address]
    email = "tobias.looker@mattr.global"

%%%

.# Abstract

This specification defines an extension of OpenID Connect to allow presentation of claims in the form of W3C Verifiable Credentials as part of the protocol flow in addition to claims provided in the `id_token` and/or via Userinfo responses.

{mainmatter}

# Introduction

This specification extends OpenID Connect with support for presentation of claims via W3C Verifiable Credentials. This allows existing OpenID Connect RPs to extends their reach towards claims sources asserting claims in this format. It also allows new applications built using Verifiable Credentials to utilize OpenID Connect as integration and interoperability layer towards credential holders. 

# Use Cases

## Verifier accesses Wallet via OpenID Connect

A Verifier uses OpenID Connect to obtain verifiable presentations. This is a simple and mature way to obtain identity data. From a technical perspective, this also makes integration with OAuth-protected APIs easier as OpenID Connect is based on OAuth.  

## Existing OpenID Connect RP integrates SSI wallets

An application currently utilizing OpenID Connect for accessing various federated identity providers can use the same protocol to also integrate with emerging SSI-based wallets. Thats an convenient transition path leveraging existing expertise and protecting investments made.

## Existing OpenID Connect OP as custodian of End-User Credentials

An existing OpenID Connect may extends its service by maintaining credentials issued by other claims sources on behalf of its customers. Customers can mix claims of the OP and from their credentials to fulfil authentication requests. 

## Federated OpenID Connect OP adds device-local mode

An extisting OpenID Connect OP with a native user experience (PWA or native app) issues Verifiable Credentials and stores it on the user's device linked to a private key residing on this device under the user's control. For every authentication request, the native user experience first checks whether this request can be fulfilled using the locally stored credentials. If so, it generates a presentations signed with the user's keys in order to prevent replay of the credential. 

This approach dramatically reduces latency and reduces load on the OP's servers. Moreover, the user can identity, authenticate, and authorize even in situations with unstable or without internet connectivity. 
# Terminology

Credential

A set of one or more claims made by an issuer. (see https://www.w3.org/TR/vc-data-model/#terminology)

Verifiable Credential (VC)

A verifiable credential is a tamper-evident credential that has authorship that can be cryptographically verified. Verifiable credentials can be used to build verifiable presentations, which can also be cryptographically verified. The claims in a credential can be about different subjects. (see https://www.w3.org/TR/vc-data-model/#terminology)

Presentation

Data derived from one or more verifiable credentials, issued by one or more issuers, that is shared with a specific verifier. (see https://www.w3.org/TR/vc-data-model/#terminology)

Verified Presentation (VP)

A verifiable presentation is a tamper-evident presentation encoded in such a way that authorship of the data can be trusted after a process of cryptographic verification. Certain types of verifiable presentations might contain data that is synthesized from, but do not contain, the original verifiable credentials (for example, zero-knowledge proofs). (see https://www.w3.org/TR/vc-data-model/#terminology)

W3C Verifiable Credential Objects

Both verifiable credentials and verifiable presentations

Proof

One or more cryptographic proofs that can be used to detect tampering and verify the authorship of a credential or presentation. The specific method used for an embedded proof MUST be included using the type property. (see https://www.w3.org/TR/vc-data-model/#proofs-signatures)

# Overview 

This specification defines mechanisms to allow RPs to request and OPs to provide Verifiable Presentations via OpenID Connect. 

Verifiable Presentations are used to present claims along with cryptographic proofs of the link between presenter and subject of the verifiable credentials it contains. A verifiable presentation can contain a subset of claims asserted in a certain credential (selective disclosure) and it can assemble claims from different credentials. 

There are two credential formats to VCs and VPs: JSON or JSON-LD. There are also two proof formats to VCs and VPs: JWT and Linked Data Proofs. Each of those formats has different properties and capabilites and each of them comes with different proof types. Proof formats are agnostic to the credential format chosen. However, the JSON credential format is commonly used with JSON Web Signatures (https://www.w3.org/TR/vc-data-model/#json-web-token). JSON-LD is commonly used with different kinds of Linked Data Proofs and JSON Web Signatures (https://www.w3.org/TR/vc-data-model/#json-ld). Applications can use all beforementioned assertion and proof formats with this specification. 

This specification introduces the following representations to exchange verifiable credentials objectes between OpenID OPs and RPs.

* The JWT claim `verifiable_presentations` used as generic container to embed verifiable presentation objects into ID tokens or userinfo responses.
* The new token types "VP Token" used as generic container for verifiable presentation objects in authentication and token responses in addition to ID Tokens.

All representations share the same container format.
# Container Format

A verifiable presentation container is an array of objects, each of them containing the following fields:

`format`: REQUIRED A JSON string denoting the proof format the presentation was returned in. This specification introduces the values `jwt_vp` and `ldp_vp` to denote credentials in JSON-LD and JWT format, respectively, as defined in [@!DIF.PresentationExchange].  

`presentation` : REQUIRED. A W3C Verifiable Presentation with a cryptographically verifiable proof in the defined proof format. 

Note that OP would first encode VPs using the rules defined in the Verifiable Credential specification either in JWT format or JSON-LD format, before encoded VPs as container objects.

Here is an example: 

```json
[
   {
      "format":"jwt_vp",
      "presentation":
      "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImRpZDpleGFtcGxlOmFiZmUxM2Y3MTIxMjA0
      MzFjMjc2ZTEyZWNhYiNrZXlzLTEifQ.eyJzdWIiOiJkaWQ6ZXhhbXBsZTplYmZlYjFmNzEyZWJjNmYxY
      zI3NmUxMmVjMjEiLCJqdGkiOiJodHRwOi8vZXhhbXBsZS5lZHUvY3JlZGVudGlhbHMvMzczMiIsImlzc
      yI6Imh0dHBzOi8vZXhhbXBsZS5jb20va2V5cy9mb28uandrIiwibmJmIjoxNTQxNDkzNzI0LCJpYXQiO
      jE1NDE0OTM3MjQsImV4cCI6MTU3MzAyOTcyMywibm9uY2UiOiI2NjAhNjM0NUZTZXIiLCJ2YyI6eyJAY
      29udGV4dCI6WyJodHRwczovL3d3dy53My5vcmcvMjAxOC9jcmVkZW50aWFscy92MSIsImh0dHBzOi8vd
      3d3LnczLm9yZy8yMDE4L2NyZWRlbnRpYWxzL2V4YW1wbGVzL3YxIl0sInR5cGUiOlsiVmVyaWZpYWJsZ
      UNyZWRlbnRpYWwiLCJVbml2ZXJzaXR5RGVncmVlQ3JlZGVudGlhbCJdLCJjcmVkZW50aWFsU3ViamVjd
      CI6eyJkZWdyZWUiOnsidHlwZSI6IkJhY2hlbG9yRGVncmVlIiwibmFtZSI6IjxzcGFuIGxhbmc9J2ZyL
      UNBJz5CYWNjYWxhdXLDqWF0IGVuIG11c2lxdWVzIG51bcOpcmlxdWVzPC9zcGFuPiJ9fX19.KLJo5GAy
      BND3LDTn9H7FQokEsUEi8jKwXhGvoN3JtRa51xrNDgXDb0cq1UTYB-rK4Ft9YVmR1NI_ZOF8oGc_7wAp
      8PHbF2HaWodQIoOBxxT-4WNqAxft7ET6lkH-4S6Ux3rSGAmczMohEEf8eCeN-jC8WekdPl6zKZQj0YPB
      1rx6X0-xlFBs7cl6Wt8rfBP_tZ9YgVWrQmUWypSioc0MUyiphmyEbLZagTyPlUyflGlEdqrZAv6eSe6R
      txJy6M1-lD7a5HTzanYTWBPAUHDZGyGKXdJw-W_x0IWChBzI8t3kpG253fg6V3tPgHeKXE94fz_QpYfg
      --7kLsyBAfQGbg"
   },
   {
      "format":"ldp_vp",
      "presentation":{
         "@context":[
            "https://www.w3.org/2018/credentials/v1"
         ],
         "type":[
            "VerifiablePresentation"
         ],
         "verifiableCredential":[
            {
               "@context":[
                  "https://www.w3.org/2018/credentials/v1",
                  "https://www.w3.org/2018/credentials/examples/v1"
               ],
               "id":"https://example.com/credentials/1872",
               "type":[
                  "VerifiableCredential",
                  "IDCardCredential"
               ],
               "issuer":{
                  "id":"did:example:issuer"
               },
               "issuanceDate":"2010-01-01T19:23:24Z",
               "credentialSubject":{
                  "given_name":"Fredrik",
                  "family_name":"Strömberg",
                  "birthdate":"1949-01-22"
               },
               "proof":{
                  "type":"Ed25519Signature2018",
                  "created":"2021-03-19T15:30:15Z",
                  "jws":"eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..PT8yCqVjj5ZHD0W36zsBQ47oc3El07WGPWaLUuBTOT48IgKI5HDoiFUt9idChT_Zh5s8cF_2cSRWELuD8JQdBw",
                  "proofPurpose":"assertionMethod",
                  "verificationMethod":"did:example:issuer#keys-1"
               }
            }
         ],
         "id":"ebc6f1c2",
         "holder":"did:example:holder",
         "proof":{
            "type":"Ed25519Signature2018",
            "created":"2021-03-19T15:30:15Z",
            "challenge":"()&)()0__sdf",
            "jws":"eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..GF5Z6TamgNE8QjE3RbiDOj3n_t25_1K7NVWMUASe_OEzQV63GaKdu235MCS3hIYvepcNdQ_ZOKpGNCf0vIAoDA",
            "proofPurpose":"authentication",
            "verificationMethod":"did:example:holder#key-1"
         }
      }
   }
]
```
# JWT parameters extention

Verifiable credential objects can be exchanged between OP and RP enveloped in JWT claims in ID tokens or userinfo responses.  

This specification introduces the following JWT claim for that purpose:

- `verifiable_presentations`:  A claim whose value is a verifiable presentations container object as defined above.

This claim can be added to ID Tokens, Userinfo responses as well as Access Tokens and Introspection response. It MAY also be included as aggregated or distributed claims (see Section 5.6.2 of the OpenID Connect specification [OpenID]).

Note that above claim has to be distinguished from `vp` or `vc` claims as defined in [JWT proof format](https://www.w3.org/TR/vc-data-model/#json-web-token). `vp` or `vc` claims contain those parts of the standard verifiable credentials and verifiable presentations where no explicit encoding rules for JWT exist. They are used as part of a verifiable credential or presentation in JWT format. They are not meant to include complete verifiable credentials or verifiable presentations objects which is the purpose of the claims defined in this specification.

# New Tokens extention

This specifications introduces the following new token:

* VP Token: a token containing a verifiable presentations container as defined above. Such a token is provided to the RP in addition to an `id_token` in the `vp_token` parameter. 

`vp_token` is provided in the same response as the `id_token`. Depending on the response type, this can be either the authentication response or the token response. Authentication event information is conveyed via the id token while it's up to the RP to determine what (additional) claims are allocated to `id_token` and `vp_token`, respectively, via the `claims` parameter. 

If the `vp_token` is returned in the frontchannel, a hash of the respective token MUST be included in `id_token`.

# Requesting Verifiable Presentations 

This draft extends the existing OpenID Connect `claims` request parameter to allow RPs to request verifiable presentations using the request syntax defined in [@!DIF.PresentationExchange].

## Embedded Verifiable Presentations {#verifiable_presentations}

A Verifiable Presentation is requested using the synatx defined by the `presentation_definition` element as defined in Section 4 of [@!DIF.PresentationExchange]. This draft deviates from the defintion given in [@!DIF.PresentationExchange] as follows in order to better fit the characteristics of the OpenID Connect protocol:

* the element name is `verifiable_presentations` instead of `presentation_definition`
* the `id` element is optional
* the field `id` of the `input_descriptor` sub element is optional

A Verifiable Presentation embedded in an ID Token (or userinfo response) is requested by adding the element `verifiable_presentations` to the `id_token` (or `userinfo`) top level element of the `claims` parameter. 

The `verifiable_presentations` element MUST contain a `input_descriptors` containing at least the `schema` sub element

The `verifiable_presentations` element MAY contain all other elements as defined in [@!DIF.PresentationExchange] except the `format` element.

Note: supported presentation formats, proof types, and algorithms are determined using new RP and OP metadata (see ). 

Here is a non-normative example: 

<{{examples/request/id_token_type_only.json}}

This simple example requests the presentation of a credential of a certain type. 

The following example

<{{examples/request/id_token_type_and_claims.json}}

shows how the RP can request selective dislosure or certain claims from a credential of a particular type. 

RPs can also ask for alternative credentials being presented, which is shown in the next example:

<{{examples/request/id_token_alternative_credentials.json}}

### VP Token

A VP Token is requested by adding a new top level element `vp_token` to the `claims` parameter. This element uses the same syntax as defined by `verifiable_presentations` in [Embedded Verifiable Presentations](#verifiable_presentations). 

This is illustrated in the following example:

<{{examples/request/vp_token_type_only.json}}

# Metadata

This specification introduces additional metadata to enable RP and OP to determine the verifiable presentation formats, proof types and algorithms to be used in a protocol exchange. 

## RP Metadata

RPs indicate the suported formats using the follwoing element.

* `vp_formats`: an object defining the formats, proof types and algorithms a RP supports. The is based on the definition of the `format` elememt in a `presentation_definition` as defined in [@!DIF.PresentationExchange] with the supported formats `jwt_vp` and `ldp_vp`.

Here is an example for a RP registering with a Standard OP via dynamic client registration:

<{{examples/client_metadata/client_code_format.json}}

Here is an example for a SIOP RP to be used as value of the `registration` request parameter:

<{{examples/client_metadata/client_siop_format.json}}

## OP Metadata

The OP publishes the formats it supports using the `vp_formats` metadata parameter as defined above in its "openid-configuration". 

# Security Considerations

To prevent replay attacks, verifiable presentation container objects MUST be linked to `client_id` and if provided `nonce` from the Authentication Request. The `client_id` is used 
to detect presentation of credentials to a different than the intended party. The `nonce` value binds the presentation to a certain authentication transaction and allows
the verifier to detect injection of a presentation in the OpenID Connect flow, which is especially important in flows where the presentation is passed through the front channel. 

The values are passed through unmodified from the Authentication Request to the verifiable presentations. 

Note: These values MAY be represented in different claims according to the selected proof format denated by the format claim in the verifiable presentation container.

Here is a non-normative example for format=`jwt_vp` (only relevant part):

```json
{
  "iss": "did:example:ebfeb1f712ebc6f1c276e12ec21",
  "jti": "urn:uuid:3978344f-8596-4c3a-a978-8fcaba3903c5",
  "aud": "s6BhdRkqt3",
  "nonce": "343s$FSFDa-",
  "nbf": 1541493724,
  "iat": 1541493724,
  "exp": 1573029723,
  "vp": {
    "@context": [
      "<https://www.w3.org/2018/credentials/v1",>
      "<https://www.w3.org/2018/credentials/examples/v1">
    ],
    "type": ["VerifiablePresentation"],

    "verifiableCredential": [""]
  }
}
```

In the example above, `nonce` is included as the `nonce` and `client_id` as the `aud` value in the proof of the verifiable presentation.

Here is a non-normative example for format=`ldp_vp` (only relevant part):

```json
{
  "@context": [ ... ],
  "type": "VerifiablePresentation",
  "verifiableCredential": [ ... ],
  "proof": {
    "type": "RsaSignature2018",
    "created": "2018-09-14T21:19:10Z",
    "proofPurpose": "authentication",
    "verificationMethod": "did:example:ebfeb1f712ebc6f1c276e12ec21#keys-1",    
    "challenge": "n-0S6_WzA2Mj",
    "domain": "s6BhdRkqt3",
    "jws": "eyJhbGciOiJSUzI1NiIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..kTCYt5
      XsITJX1CxPCT8yAV-TVIw5WEuts01mq-pQy7UJiN5mgREEMGlv50aqzpqh4Qq_PbChOMqs
      LfRoPsnsgxD-WUcX16dUOqV0G_zS245-kronKb78cPktb3rk-BuQy72IFLN25DYuNzVBAh
      4vGHSrQyHUGlcTwLtjPAnKb78"
  }
}
```

In the example above, `nonce` is included as the `challenge` and `client_id` as the `domain` value in the proof of the verifiable presentation.

#  Examples 

This section illustrates examples when W3C Verifiable Credentials objects are requested using `claims` parameter and returned inside ID Tokens.

## Self-Issued OpenID Provider with Verifiable Presentation in ID Token 

Below are the examples when W3C Verifiable Credentials are requested and returned inside ID Token as part of Self-Issued OP response. ID Token contains a `verifiable_presentations` claim with the Verifiable Presentation data. It can also contain `verifiable_credentials` element with the Verifiable Credential data. 

### Authentication request

The following is a non-normative example of how an RP would use the `claims` parameter to request the `verifiable_presentations` claim in the `id_token`:

```
  HTTP/1.1 302 Found
  Location: openid://?
    response_type=id_token
    &client_id=https%3A%2F%2Fclient.example.org%2Fcb
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
    &scope=openid
    &claims=%7B%22id_token%22%3A%7B%22vc%22%3A%7B%22types%22%3A%5B%22https%3A%2F%
     2Fdid.itsourweb.org%3A3000%2Fsmart-credential%2FOntario-Health-Insurance-Plan
     %22%5D%7D%7D%7D
    &state=af0ifjsldkj
    &nonce=960848874
    &registration_uri=https%3A%2F%2F
      client.example.org%2Frf.txt%22%7D
      
```
#### `claims` parameter 

Below is a non-normative example of how the `claims` parameter can be used for requesting verified presentations containg a credential of a certain type.

<{{examples/request/id_token_health.json}}

### Authentication Response 

Below is a non-normative example of ID Token that includes `verifiable_presentations` claim.
Note: the RP was setup with the preferred format `jwt_vp`.

```json
{
  "kid": "did:ion:EiC6Y9_aDaCsITlY06HId4seJjJ...b1df31ec42d0",
  "typ": "JWT",
  "alg": "ES256K"
}.{
   "iss":"https://self-issued.me",
   "aud":"https://book.itsourweb.org:3000/client_api/authresp/uhn",
   "iat":1615910538,
   "exp":1615911138,
   "sub":"did:ion:EiC6Y9_aDaCsITlY06HId4seJjJ-9...mS3NBIn19",
   "auth_time":1615910535,
   "nonce":"960848874",
   "verifiable_presentations":[
      {
         "format":"jwt_vp",
         "presentation":"ewogICAgImlzcyI6Imh0dHBzOi8vYm9vay5pdHNvdXJ3ZWIub...IH0="
      }
   ],   
   "sub_jwk":{
      "crv":"P-384",
      "kty":"EC",
      "kid": "c7298a61a6904426a580b1df31ec42d0",
      "x":"jf3a6dquclZ4PJ0JMU8RuucG9T1O3hpU_S_79sHQi7VZBD9e2VKXPts9lUjaytBm",
      "y":"38VlVE3kNiMEjklFe4Wo4DqdTKkFbK6QrmZf77lCMN2x9bENZoGF2EYFiBsOsnq0"
   }
}
```

Below is a non-normative example of a decoded Verifiable Presentation object that was included in `verifiable_presentations`. 
Note that `vp` is used to contain only "those parts of the standard verifiable presentation where no explicit encoding rules for JWT exist" [VC-DATA-MODEL]

```json
  {
    "iss":"did:ion:EiC6Y9_aDaCsITlY06HId4seJjJ...b1df31ec42d0",
    "aud":"https://book.itsourweb.org:3000/ohip",
    "iat":1615910538,
    "exp":1615911138,   
    "nbf":1615910538,
    "nonce":"acIlfiR6AKqGHg",
    "vp":{
        "@context":[
          "https://www.w3.org/2018/credentials/v1",
          "https://ohip.ontario.ca/v1"
        ],
        "type":[
          "VerifiablePresentation"
        ],
        "verifiableCredential":[
          "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6InVybjp1dWlkOjU0ZDk2NjE2LTE1MWUt...OLryT1g"    
        ]
    }   
  }
```
## Self-Issued OpenID Provider with Verifiable Presentation in ID Token (selective disclosure)
### `claims` parameter 

Below is a non-normative example of how the `claims` parameter can be used for requesting verified presentations with selective disclosure.
Note: the RP was setup with the preferred format `ldp_vp`.

<{{examples/request/id_token_type_and_claims.json}}

### Authentication Response 

Below is a non-normative example of ID Token that includes `verifiable_presentations` claim.

```json
{
   "iss":"https://self-issued.me",
   "aud":"https://book.itsourweb.org:3000/client_api/authresp/uhn",
   "iat":1615910538,
   "exp":1615911138,
   "sub":"did:ion:EiC6Y9_aDaCsITlY06HId4seJjJ...b1df31ec42d0",
   "auth_time":1615910535,
   "verifiable_presentations":[
      {
         "format":"jwt_vp",
         "presentation":{
            "@context":[
               "https://www.w3.org/2018/credentials/v1"
            ],
            "type":[
               "VerifiablePresentation"
            ],
            "verifiableCredential":[
               {
                  "@context":[
                     "https://www.w3.org/2018/credentials/v1",
                     "https://www.w3.org/2018/credentials/examples/v1"
                  ],
                  "id":"https://example.com/credentials/1872",
                  "type":[
                     "VerifiableCredential",
                     "IDCardCredential"
                  ],
                  "issuer":{
                     "id":"did:example:issuer"
                  },
                  "issuanceDate":"2010-01-01T19:23:24Z",
                  "credentialSubject":{
                     "given_name":"Fredrik",
                     "family_name":"Strömberg",
                     "birthdate":"1949-01-22"
                  },
                  "proof":{
                     "type":"Ed25519Signature2018",
                     "created":"2021-03-19T15:30:15Z",
                     "jws":"eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..PT8yCqVjj5ZHD0W36zsBQ47oc3El07WGPWaLUuBTOT48IgKI5HDoiFUt9idChT_Zh5s8cF_2cSRWELuD8JQdBw",
                     "proofPurpose":"assertionMethod",
                     "verificationMethod":"did:example:issuer#keys-1"
                  }
               }
            ],
            "id":"ebc6f1c2",
            "holder":"did:example:holder",
            "proof":{
               "type":"Ed25519Signature2018",
               "created":"2021-03-19T15:30:15Z",
               "challenge":"()&)()0__sdf",
               "jws":"eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..GF5Z6TamgNE8QjE3RbiDOj3n_t25_1K7NVWMUASe_OEzQV63GaKdu235MCS3hIYvepcNdQ_ZOKpGNCf0vIAoDA",
               "proofPurpose":"authentication",
               "verificationMethod":"did:example:holder#key-1"
            }
         }
      }
   ],
   "nonce":"960848874",
   "sub_jwk":{
      "crv":"P-384",
      "kty":"EC",
      "x":"jf3a6dquclZ4PJ0JMU8RuucG9T1O3hpU_S_79sHQi7VZBD9e2VKXPts9lUjaytBm",
      "y":"38VlVE3kNiMEjklFe4Wo4DqdTKkFbK6QrmZf77lCMN2x9bENZoGF2EYFiBsOsnq0"
   }
}
```
## Authorization Code Flow with Verifiable Presentation in ID Token

Below are the examples when W3C Verifiable Credentials are requested and returned inside ID Token as part of Authorization Code flow. ID Token contains a `verifiable_presentations` element with the Verifiable Presentations data. 

### Authentication Request

```
  GET /authorize?
    response_type=code
    &client_id=s6BhdRkqt3 
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
    &scope=openid
    &claims=...
    &state=af0ifjsldkj
    &nonce=n-0S6_WzA2Mj HTTP/1.1
  Host: server.example.com
```
#### Claims parameter 

Below is a non-normative example of how the `claims` parameter can be used for requesting a verified presentations in an ID Token.

<{{examples/request/id_token_health.json}}

### Authentication Response

```
HTTP/1.1 302 Found
  Location: https://client.example.org/cb?
    code=SplxlOBeZQQYbYS6WxSbIA
    &state=af0ifjsldkj
```

### Token Request

```
  POST /token HTTP/1.1
  Host: server.example.com
  Content-Type: application/x-www-form-urlencoded
  Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

  grant_type=authorization_code
  &code=SplxlOBeZQQYbYS6WxSbIA
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```
## Authorization Code Flow with Verifiable Presentation returned from the UserInfo endpoint

Below are the examples when verifiable presentation is requested and returned from the UserInfo endpoint as part of OpenID Connect Authorization Code Flow. UserInfo response contains a `verifiable_presentations` element with the Verifiable Presentation data. 

### Authentication Request

```
  GET /authorize?
    response_type=code
    &client_id=s6BhdRkqt3 
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
    &scope=openid
    &claims=...
    &state=af0ifjsldkj
    &nonce=n-0S6_WzA2Mj HTTP/1.1
  Host: server.example.com
```

#### Claims parameter 

Below is a non-normative example of how the `claims` parameter can be used for requesting verified presentations in a userinfo response.

<{{examples/request/userinfo_health.json}}

### Authentication Response

```
HTTP/1.1 302 Found
  Location: https://client.example.org/cb?
    code=SplxlOBeZQQYbYS6WxSbIA
    &state=af0ifjsldkj
```

### Token Request

```
  POST /token HTTP/1.1
  Host: server.example.com
  Content-Type: application/x-www-form-urlencoded
  Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

  grant_type=authorization_code
  &code=SplxlOBeZQQYbYS6WxSbIA
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```
### Token Response

#### id_token

```json
{
  "iss": "http://server.example.com",
  "sub": "248289761001",
  "aud": "s6BhdRkqt3",
  "nonce": "n-0S6_WzA2Mj",
  "exp": 1311281970,
  "iat": 1311280970,
  "auth_time": 1615910535
}
```
### UserInfo Response 

Below is a non-normative example of a UserInfo Response that includes a `verifiable_presentations` claim:

```json
  HTTP/1.1 200 OK
  Content-Type: application/json

  {
   "sub": "248289761001",
   "name": "Jane Doe",
   "given_name": "Jane",
   "family_name": "Doe",
    "verifiable_presentations":[
      {
         "format":"jwt_vp",
         "presentation":"ewogICAgImlzcyI6Imh0dHBzOi8vYm9vay5pdHNvdXJ3ZWIub...IH0="
      }
   ],   
  }
```

JWT inside the `verifiable_presentations` claim when decoded equals to a verifiable presentation in Self-Issued OP with Verifiable Presentation in ID Token, Authentication Response section.

## Authorization Code Flow with Verifiable Presentation returned from the UserInfo endpoint (LDP)
### Claims parameter 

Below is a non-normative example of how the `claims` parameter can be used for requesting verified presentations signed as Linked Data Proofs.

<{{examples/request/userinfo_type_and_claims.json}}

### Token Response

#### id_token

```json
{
  "iss": "http://server.example.com",
  "sub": "248289761001",
  "aud": "s6BhdRkqt3",
  "nonce": "n-0S6_WzA2Mj",
  "exp": 1311281970,
  "iat": 1311280970,
  "auth_time": 1615910535
}
```
### UserInfo Response 

Below is a non-normative example of a UserInfo Response that includes `verifiable_presentations` claim:

```json
  HTTP/1.1 200 OK
  Content-Type: application/json

  {
   "sub":"248289761001",
   "name":"Jane Doe",
   "given_name":"Jane",
   "family_name":"Doe",
   "verifiable_presentations":[
      {
         "format":"jwt_vp",
         "presentation":{
            "@context":[
               "https://www.w3.org/2018/credentials/v1"
            ],
            "type":[
               "VerifiablePresentation"
            ],
            "verifiableCredential":[
               {
                  "@context":[
                     "https://www.w3.org/2018/credentials/v1",
                     "https://www.w3.org/2018/credentials/examples/v1"
                  ],
                  "id":"https://example.com/credentials/1872",
                  "type":[
                     "VerifiableCredential",
                     "IDCardCredential"
                  ],
                  "issuer":{
                     "id":"did:example:issuer"
                  },
                  "issuanceDate":"2010-01-01T19:23:24Z",
                  "credentialSubject":{
                     "given_name":"Fredrik",
                     "family_name":"Strömberg",
                     "birthdate":"1949-01-22"
                  },
                  "proof":{
                     "type":"Ed25519Signature2018",
                     "created":"2021-03-19T15:30:15Z",
                     "jws":"eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..PT8yCqVjj5ZHD0W36zsBQ47oc3El07WGPWaLUuBTOT48IgKI5HDoiFUt9idChT_Zh5s8cF_2cSRWELuD8JQdBw",
                     "proofPurpose":"assertionMethod",
                     "verificationMethod":"did:example:issuer#keys-1"
                  }
               }
            ],
            "id":"ebc6f1c2",
            "holder":"did:example:holder",
            "proof":{
               "type":"Ed25519Signature2018",
               "created":"2021-03-19T15:30:15Z",
               "challenge":"()&)()0__sdf",
               "jws":"eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..GF5Z6TamgNE8QjE3RbiDOj3n_t25_1K7NVWMUASe_OEzQV63GaKdu235MCS3hIYvepcNdQ_ZOKpGNCf0vIAoDA",
               "proofPurpose":"authentication",
               "verificationMethod":"did:example:holder#key-1"
            }
         }
      }
   ]
}
```
## SIOP with vp_token
This section illustrates the protocol flow for the case of communication through the front channel only (like in SIOP).

### Authentication request

The following is a non-normative example of how an RP would use the `claims` parameter to request claims in the `vp_token`:

```
  HTTP/1.1 302 Found
  Location: openid://?
    response_type=id_token
    &client_id=https%3A%2F%2Fclient.example.org%2Fcb
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
    &scope=openid
    &claims=...
    &state=af0ifjsldkj
    &nonce=n-0S6_WzA2Mj
    &registration_uri=https%3A%2F%2F
      client.example.org%2Frf.txt%22%7D
      
```
#### claims parameter

<{{examples/request/vp_token_type_and_claims.json}}

### Authentication Response (including vp_token)

The successful authentication response contains a `vp_token` parameter along with  `id_token` and `state`.
```
  HTTP/1.1 302 Found
  Location: https://client.example.org/cb#
    id_token=eyJ0 ... NiJ9.eyJ1c ... I6IjIifX0.DeWt4Qu ... ZXso
    &vp_token=...
    &state=af0ifjsldkj
      
```
#### id_token

This example shows an ID Token containing a `vp_hash`:

```json
{
   "iss":"https://book.itsourweb.org:3000/wallet/wallet.html",
   "aud":"https://book.itsourweb.org:3000/client_api/authresp/uhn",
   "iat":1615910538,
   "exp":1615911138,
   "sub":"urn:uuid:68f874e2-377c-437f-a447-b304967ca351",
   "auth_time":1615910535,
   "vp_hash":"77QmUPtjPfzWtF2AnpK9RQ",
   "nonce":"960848874",
   "sub_jwk":{
      "crv":"P-384",
      "ext":true,
      "key_ops":[
         "verify"
      ],
      "kty":"EC",
      "x":"jf3a6dquclZ4PJ0JMU8RuucG9T1O3hpU_S_79sHQi7VZBD9e2VKXPts9lUjaytBm",
      "y":"38VlVE3kNiMEjklFe4Wo4DqdTKkFbK6QrmZf77lCMN2x9bENZoGF2EYFiBsOsnq0"
   }
}
```
#### vp_token content

```json
[
   {
      "format":"ldp_vp",
      "presentation":{
         "@context":[
            "https://www.w3.org/2018/credentials/v1"
         ],
         "type":[
            "VerifiablePresentation"
         ],
         "verifiableCredential":[
            {
               "@context":[
                  "https://www.w3.org/2018/credentials/v1",
                  "https://www.w3.org/2018/credentials/examples/v1"
               ],
               "id":"https://example.com/credentials/1872",
               "type":[
                  "VerifiableCredential",
                  "IDCardCredential"
               ],
               "issuer":{
                  "id":"did:example:issuer"
               },
               "issuanceDate":"2010-01-01T19:23:24Z",
               "credentialSubject":{
                  "given_name":"Fredrik",
                  "family_name":"Strömberg",
                  "birthdate":"1949-01-22"
               },
               "proof":{
                  "type":"Ed25519Signature2018",
                  "created":"2021-03-19T15:30:15Z",
                  "jws":"eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..PT8yCqVjj5ZHD0W36zsBQ47oc3El07WGPWaLUuBTOT48IgKI5HDoiFUt9idChT_Zh5s8cF_2cSRWELuD8JQdBw",
                  "proofPurpose":"assertionMethod",
                  "verificationMethod":"did:example:issuer#keys-1"
               }
            }
         ],
         "id":"ebc6f1c2",
         "holder":"did:example:holder",
         "proof":{
            "type":"Ed25519Signature2018",
            "created":"2021-03-19T15:30:15Z",
            "challenge":"()&)()0__sdf",
            "jws":"eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..GF5Z6TamgNE8QjE3RbiDOj3n_t25_1K7NVWMUASe_OEzQV63GaKdu235MCS3hIYvepcNdQ_ZOKpGNCf0vIAoDA",
            "proofPurpose":"authentication",
            "verificationMethod":"did:example:holder#key-1"
         }
      }
   }
]
```
## Authorization Code Flow with vp_token

This section illustrates the protocol flow for the case of communication using frontchannel and backchannel (utilizing the authorization code flow).

### Authentication Request

```
  GET /authorize?
    response_type=code
    &client_id=s6BhdRkqt3 
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
    &scope=openid
    &claims=...
    &state=af0ifjsldkj
    &nonce=n-0S6_WzA2Mj HTTP/1.1
  Host: server.example.com
```

#### Claims parameter

<{{examples/request/vp_token_type_and_claims.json}}

### Authentication Response
```
HTTP/1.1 302 Found
  Location: https://client.example.org/cb?
    code=SplxlOBeZQQYbYS6WxSbIA
    &state=af0ifjsldkj
```

### Token Request
```
  POST /token HTTP/1.1
  Host: server.example.com
  Content-Type: application/x-www-form-urlencoded
  Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

  grant_type=authorization_code
  &code=SplxlOBeZQQYbYS6WxSbIA
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```

### Token Response (including vp_token)

```json
{
   "access_token":"SlAV32hkKG",
   "token_type":"Bearer",
   "refresh_token":"8xLOxBtZp8",
   "expires_in":3600,
   "id_token":"eyJ0 ... NiJ9.eyJ1c ... I6IjIifX0.DeWt4Qu ... ZXso",
   "vp_token":[
      {
         "format":"ldp_vp",
         "presentation":{
            "@context":[
               "https://www.w3.org/2018/credentials/v1"
            ],
            "type":[
               "VerifiablePresentation"
            ],
            "verifiableCredential":[
               {
                  "@context":[
                     "https://www.w3.org/2018/credentials/v1",
                     "https://www.w3.org/2018/credentials/examples/v1"
                  ],
                  "id":"https://example.com/credentials/1872",
                  "type":[
                     "VerifiableCredential",
                     "IDCardCredential"
                  ],
                  "issuer":{
                     "id":"did:example:issuer"
                  },
                  "issuanceDate":"2010-01-01T19:23:24Z",
                  "credentialSubject":{
                     "given_name":"Fredrik",
                     "family_name":"Strömberg",
                     "birthdate":"1949-01-22"
                  },
                  "proof":{
                     "type":"Ed25519Signature2018",
                     "created":"2021-03-19T15:30:15Z",
                     "jws":"eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..PT8yCqVjj5ZHD0W36zsBQ47oc3El07WGPWaLUuBTOT48IgKI5HDoiFUt9idChT_Zh5s8cF_2cSRWELuD8JQdBw",
                     "proofPurpose":"assertionMethod",
                     "verificationMethod":"did:example:issuer#keys-1"
                  }
               }
            ],
            "id":"ebc6f1c2",
            "holder":"did:example:holder",
            "proof":{
               "type":"Ed25519Signature2018",
               "created":"2021-03-19T15:30:15Z",
               "challenge":"()&)()0__sdf",
               "jws":"eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..GF5Z6TamgNE8QjE3RbiDOj3n_t25_1K7NVWMUASe_OEzQV63GaKdu235MCS3hIYvepcNdQ_ZOKpGNCf0vIAoDA",
               "proofPurpose":"authentication",
               "verificationMethod":"did:example:holder#key-1"
            }
         }
      }
   ]
}
```
#### id_token

```json
{
  "iss": "http://server.example.com",
  "sub": "248289761001",
  "aud": "s6BhdRkqt3",
  "nonce": "n-0S6_WzA2Mj",
  "exp": 1311281970,
  "iat": 1311280970,
  "vp_hash": "77QmUPtjPfzWtF2AnpK9RQ"
}
``` 

{backmatter}

<reference anchor="OpenID" target="http://openid.net/specs/openid-connect-core-1_0.html">
  <front>
    <title>OpenID Connect Core 1.0 incorporating errata set 1</title>
    <author initials="N." surname="Sakimura" fullname="Nat Sakimura">
      <organization>NRI</organization>
    </author>
    <author initials="J." surname="Bradley" fullname="John Bradley">
      <organization>Ping Identity</organization>
    </author>
    <author initials="M." surname="Jones" fullname="Mike Jones">
      <organization>Microsoft</organization>
    </author>
    <author initials="B." surname="de Medeiros" fullname="Breno de Medeiros">
      <organization>Google</organization>
    </author>
    <author initials="C." surname="Mortimore" fullname="Chuck Mortimore">
      <organization>Salesforce</organization>
    </author>
   <date day="8" month="Nov" year="2014"/>
  </front>
</reference>

<reference anchor="DIF.PresentationExchange" target="hhttps://identity.foundation/presentation-exchange/">
        <front>
          <title>Presentation Exchange v1.0.0</title>
		  <author fullname="Daniel Buchner">
            <organization>Microsoft</organization>
          </author>
          <author fullname="Brent Zunde">
            <organization>Evernym</organization>
          </author>
          <author fullname="Martin Riedel">
            <organization>Consensys Mesh</organization>
          </author>
         <date day="8" month="Nov" year="2014"/>
        </front>
</reference>

<reference anchor="OpenID.Registration" target="https://openid.net/specs/openid-connect-registration-1_0.html">
        <front>
          <title>OpenID Connect Dynamic Client Registration 1.0 incorporating errata set 1</title>
		  <author fullname="Nat Sakimura">
            <organization>NRI</organization>
          </author>
          <author fullname="John Bradley">
            <organization>Ping Identity</organization>
          </author>
          <author fullname="Mike Jones">
            <organization>Microsoft</organization>
          </author>
          <date day="8" month="Nov" year="2014"/>
        </front>
 </reference>

# JSON-LD Proofs

Verifiable Credential (VC) and Verifiable Presentation (VP) [JSON-LD
proof](https://w3c-ccg.github.io/ld-proofs) contains information about digitally
signed VCs or VPs in JSON-LD format. This section details the signing
processes for digital algorithms that take HMACed content as input.

First, we present JSON-LD canonicalization that transforms a JSON-LD object into
a normalized form suitable for signing. Next, we describe how to create single
and multiple VC/VP proofs. Last, we summarize how to crate [CMS Advanced digital
Electronic Signatures (CAdES)](https://datatracker.ietf.org/doc/html/rfc5126),
[JSON Advanced digital Electronic Signatures
(JAdES)](https://www.etsi.org/deliver/etsi_ts/119100_119199/11918201/01.01.01_60/ts_11918201v010101p.pdf)
and [JSON Web Signatures (JWS)](https://tools.ietf.org/html/rfc7515) signatures.

## VC and VP serialization overview

To sign a VC or VP, we need to transform it into a normalized form that is
independent of the arrangement of the input data. JSON-LD canonicalization
deserializes the JSON-LD to RDF and applies URDNA2015 canonicalization
algorithm. For details see [RDF Dataset
Canonicalization](https://json-ld.github.io/rdf-dataset-canonicalization/spec/index.html)
and [Deserialize JSON-LD to RDF
Algorithm](https://www.w3.org/TR/json-ld11-api/#deserialize-json-ld-to-rdf-algorithm).

Canonicalization ensures that JSON(-LD) object elements are always ordered
regardless of the initial arrangement. JSON standard ensures that the list order
is unaffected by the canonicalization, hence we MUST NOT change the order of any
list in the VC or VP, after the VC/VP is signed. Changing the elements order
would invalidate the signature, hence it is advisable to always store the VC/VP
in the original format.

JSON-LD objects contain links to JSON-LD schemas. Availability and immutability
of the JSON-LD schema is crucial for the canonicalization and proof validation.
Hence we SHOULD:

* store and maintain schemas locally and share them with the JSON-LD, or
* load schemas from trusted sources, or
* embed schemas into the JSON-LD.

## Single proof

This section summarizes how to sign VCs and VPs in JSON-LD format. The approach
is applicable to all signature algorithms that take MAC or HMAC of the message as
input. To create a VC/VP JSON-LF proof proceeds as follows:

1. Create a VC/VP in JSON-LD format without the proof claim. Result is a VC/VP body.
2. Canonicalize the VC/VP body by deserializing the VC/VP body into RDF and applying URDNA2015 RDF canonicalization. Result is canonicalized VC/VP body.
3. Compute SHA256 digest of the canonicalized VC/VP body. Result is VC/VP body digest.
4. Create a proof object in JSON-LD format without the proofValue and jws claims. Result is a VC/VP proof.
5. Canonicalize the VC/VP proof by deserializing the VC/VP proof into RDF and applying URDNA2015 RDF canonicalization. Result is canonicalized VC/VP proof.
6. Compute SHA256 digest of the canonicalized VC/VP proof. Result is VC/VP proof digest.
7. Concatenate the resulting digests as: (VC/VP proof digest) || (VC/VP body digest). Result is a signing payload.
8. Use the signing payload as input to the signing algorithm and sign it.
9. Add the signature to the VC/VP proof object.
10. Construct the VC/VP by inserting the new proof object into the VC/VP body.

### Multiple proofs (WIP)

In some cases VC need to be signed by several issuers or VPs need to be signed with different keys by the same holder. Proofs can be of two types: un-ordered (proof set) or ordered (proof chain). Below we summarize the steps to create proof sets and proof chains.

#### Proof Set

Proof set is crated when the proof order is unimportant. Every entity signs the VC/VP, as described in section TBD, without including any proof in the document.

Open questions:
- Who determines the signees and how is the information included in the VC/VP?
- Can we require N-out-of-M signatures
- Do we have additional requirements for iat, exp, nonce (in case of the VP)

#### Proof Chain

Proof chain is created any time VC or VP needs to be signed several times in a particular order. In this case all the previous proofs need to be included and the new signature must be appended to the proof list.

## Proof examples

In this section, we present CAdES, JWS and JAdES as JWS JSON-LD proofs.

### CAdES proof

To sign a VC or VP in the JSON-LD format with CAdES, follow steps 1-7 as
described in section [Single proof](#single-proof). The signing payload acts as
input to the CAdES algorithm for signing digests.
The resulting CAdES signature is DER, PEM or S/MIME encoded. As the JSON-LD
proof value MUST be a string, the signatures MUST be further transformed as
summarized in the table below:

| CAdES signature format | As included in JSON-LD proof                    |
| :--------------------- | :---------------------------------------------- |
| DER                    | BASE64URL encoded DER octet string.             |
| PEM                    | PEM with replaced newline characters with '\n'. |
| S/MIME                 | BASE64URL encoded S/MIME text (UTF-8) document. |

The CAdES JSON-LD proof data model is:

| Claim              | Description                                                                                                                                          | Mandatory | Type                   | Reference                                                     |
| :----------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------- | :-------- | :--------------------- | :------------------------------------------------------------ |
| type               | Proof object type. MUST be CAdES2021.                                                                                                                | Yes       | string                 | [LD Proofs](https://w3c-ccg.github.io/ld-proofs/#proof-types) |
| alg                | Signature algorithm format. MUST be "cades".                                                                                                         | Yes       | string                 |                                                               |
| enc                | Signature encoding. MUST be one of "PEM", "DER", "S/MIME".                                                                                           | Yes       | string                 |                                                               |
| proofPurpose       | Defines the proof purpose. MUST be one of "authentication", "assertionMethod"                                                                        | Yes       | string                 | [LD Proofs](https://w3c-ccg.github.io/ld-proofs/#proof-types) |
| verificationMethod | A set of parameters required to independently verify the proof, such as an identifier for a public/private key pair that would be used in the proof. | Yes       | string                 | [LD Proofs](https://w3c-ccg.github.io/ld-proofs/#proof-types) |
| created            | Date and time at which the proof has been created.                                                                                                   | Yes       | string (ISO 8601:2004) | [LD Proofs](https://w3c-ccg.github.io/ld-proofs/#proof-types) |
| exp                | Date and time on and after which the proof MUST NOT be accepted for processing. Used in VPs only.                                                    | No        | string (ISO 8691:2004) |                                                               |
| aud                | Audience(s) that this VP is intended for.                                                                                                            | VP only   | List of strings.       |                                                               |
| nonce              | Random string value used to associate a Client session with a VP request, and to mitigate replay attacks.                                            | VP only   | string                 |                                                               |
| proofValue         | Encoded digital signature as specified in the "enc" claim.                                                                                           | VP only   | string                 | [LD Proofs](https://w3c-ccg.github.io/ld-proofs/#proof-types) |

#### CAdES VC example

Example goes here.

#### CAdES VP example

Example goes here.

### JSON Web Signature

This section summarizes how to create a detached JWS for a VC or VP in JSON-LD format.

1. Follow the steps 1-7 described in section [Single Proof](#single-proof).
2. JWT payload is the signing payload as octet string.
3. Construct the JWS header according to [RFC](https://datatracker.ietf.org/doc/html/rfc7515).
4. Construct the JWS signing payload as OCTET STRING(BASE64URL(JCS(header)) .) || payload
5. Sign the payload using the selected JWS signature.
6. Construct the detached JWS according to [RFC](https://datatracker.ietf.org/doc/html/rfc7515).
7. Include resulting JWS into the proof.

The CAdES JSON-LD proof data model is:

| Claim              | Description                                                                                                                                          | Mandatory | Type                   | Reference                                                     |
| :----------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------- | :-------- | :--------------------- | :------------------------------------------------------------ |
| type               | Proof object type. MUST be CAdES2021.                                                                                                                | Yes       | string                 | [LD Proofs](https://w3c-ccg.github.io/ld-proofs/#proof-types) |
| proofPurpose       | Defines the proof purpose. MUST be one of "authentication", "assertionMethod"                                                                        | Yes       | string                 | [LD Proofs](https://w3c-ccg.github.io/ld-proofs/#proof-types) |
| verificationMethod | A set of parameters required to independently verify the proof, such as an identifier for a public/private key pair that would be used in the proof. | Yes       | string                 | [LD Proofs](https://w3c-ccg.github.io/ld-proofs/#proof-types) |
| created            | Date and time at which the proof has been created.                                                                                                   | Yes       | string (ISO 8601:2004) | [LD Proofs](https://w3c-ccg.github.io/ld-proofs/#proof-types) |
| domain             | A string value specifying the restricted domain of the proof.                                                                                        | VP only   | string                 | [LD Proofs](https://w3c-ccg.github.io/ld-proofs/#proof-types) |
| nonce              | Random string value used to associate a Client session with a VP request, and to mitigate replay attacks.                                            | VP only   | string                 | [LD Proofs](https://w3c-ccg.github.io/ld-proofs/#proof-types) |
| enc                | Signature encoding. MUST be "JWS".                                                                                                                   | Yes       | string                 |                                                               |
| proofValue         | Encoded digital signature as specified in the "enc" claim.                                                                                           | VP only   | string                 | [LD Proofs](https://w3c-ccg.github.io/ld-proofs/#proof-types) |

### JSON Advanced digital Electronic Signatures as JWS

JAdES is a JSON format for Advanced digital Electronic Signature (AdES) built on
JWS. For details see [ETSI TS 119
182-1](https://www.etsi.org/deliver/etsi_ts/119100_119199/11918201/01.01.01_60/ts_11918201v010101p.pdf).
JSON-LD can be transformed into JSON by expanding the JSON-LD, that is replacing
all claims with definitions from the JSON-LD schema. JAdES can be applied to VCs
or VPs in JSON-LD formats as follows:

1. Create VC/VP in JSON-LD format.
2. Expand the JSON-LD as described in [Expanded Document Form](https://www.w3.org/TR/json-ld/#expanded-document-form).
3. Use the resulting JSON object as input for JAdES.
4. Sign the JSON.
5. Include the resulting (detached) JWS into the VC/VP proof.

JAdES proof data model is the same as the JWS proof data model.

JAdES example goes here.


# IANA Considerations

TBD

# Acknowledgements {#Acknowledgements}

TBD

# Notices

Copyright (c) 2021 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that may cover technology that may be required to practice this specification.

# Document History

   [[ To be removed from the final specification ]]

   -01

   * adopted DIF Presentation Exchange request syntax
   * added security considerations regarding replay detection for verifiable credentials

   -00 

   *  initial revision