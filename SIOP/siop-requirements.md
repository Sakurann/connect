# Requirements for OIDC Self-Issued OpenId Provider

**Introduction**

[Chapter 7 of OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html#SelfIssued) introduces the Self-issued OpenID Provider (SIOP), a personal, self-hosted OP that issues self-signed[ ID Tokens](https://openid.net/specs/openid-connect-core-1_0.html#IDToken) directly to the RP.  SIOP operates on behalf of a human or non-human subject to share information contained in a SIOP store by including it in an ID Token signed using an identifier and keys controlled by the subject.

It was agreed in OpenID Foundation OpenID Connect Working Group to work on a separate specification covering only the SIOP part of OpenID Connect core (OIDC) specification. This document enumerates requirements for SIOP to help define the scope of a new version of SIOP specification - what will and will not be included. To better explain the requirements, this document provides examples of options that could meet the requirements, but it is not the intention of the document to suggest one option over the other.

**Scope** 

*   A. SIOP request
*   B. SIOP response
*   C. Key recovery and key rotation
*   D. Trust model between RP and SIOP
*   E. Issuance of the claims
*   F. Privacy protection


**Terminology**

Avoid using terms that do not have standards based definition (such as SSI)

**Use-Cases (non-exhausive)**

*   Cross-device authentication?
*   Identity (set of data related to the entity) needs to be provable as attested at the time of attestation and cannot be taken away
*   Realizing portability of the identifiers - being able to change the SIOP without losing the ability to identify
*   Just log-in using SIOP vs. transmission of claims
*   RP and SIOP are on the same device

**Requirements**

A.SIOP request

*   1.Not all OpenID Providers must have the capability to issue self-signed id_tokens, and that capability remains specific to Self-Issued OpenID Providers (which can be native apps, PWAs, etc.)
*   2.SIOP can be used for log in and/or requesting claims to transmit identity characteristics. 
*   3.SIOP supports a FLow that does not rely on the backchannel exchange (i.e. Implicit Flow), since SIOP is native apps, PWA, browser apps, etc.
    *   Note: Some people asserted that PWA may result in additional requirements - want to understand more
    
B.SIOP response

*   4.SIOP is able to return Verifiable Credentials and Verifiable Presentations in the response
    
C. Key recovery and key rotation (cryptography itself is out of scope)

*   5.Add layer of indirection in ‘sub’ value to allow several methods of deriving key information derivation: using sub_jwks with URNs (Cryptographic attestation of the claims), DIDs resolved into DID documents, or OpenID Connect Federation entity statements.
*   6.Relationship between old and new identifiers/keys can be attested 
        
D.Trust model between RP and SIOP (Discovery and Registration)

*   7.RP can discover SIOP, and differentiate it from not SIOP-enabled OPs (openid:// custom schema, deeplinks)
*   8.RP and SIOP negotiate configuration information that both parties support
*   9.RP does not have to pre-register with SIOP, but must communicate metadata using the registration parameter iwhen RP is without a webserver or in the client_id(redirect_uri) when RP can host a webserver
    
E.Issuance of the claims (SIOP - Claims Provider)

*   10. [this part could rely on a separate specification] SIOP providers can use Claims provider 
    
F.Privacy protection

*   11.RPs should understand the security/privacy posture of SIOP
*   12.SIOP should support all types of identifiers (pairwise, omnidirectional, and ephemeral)
*   13.RP must be able to receive the claims when the end-user is offline without colluding with the Claims Provider


**Note**
*   Selective Disclosure of the claims is left to the implementations
*   Not sure if there are specific requirements to enable support for PWA-based SIOPs?
*   Is there a need to account for both situations where RP and SIOP and on the same/different devices?
	
	
**Reference **

This requirement list is build upon various discussions and drafts whose non-exhaustive list can be found below:

* [https://nat.sakimura.org/2018/12/11/todo-list-for-self-issued-op/](https://nat.sakimura.org/2018/12/11/todo-list-for-self-issued-op/)
* [https://docs.google.com/document/d/1Q1kTbCjvr0LoD7fsgrtUW0-olorkeebyD7I6mBByMc4/edit#](https://docs.google.com/document/d/1Q1kTbCjvr0LoD7fsgrtUW0-olorkeebyD7I6mBByMc4/edit#)
* [https://bitbucket.org/openid/connect/issues/1180/siop-laundry-list](https://bitbucket.org/openid/connect/issues/1180/siop-laundry-list)
* [https://bitbucket.org/openid/connect/src/master/SIOP/draft-jones-self_issued_identifier.md](https://bitbucket.org/openid/connect/src/master/SIOP/draft-jones-self_issued_identifier.md)
* [https://hackmd.io/YL3juOZrRH65XV51bZR6dQ?view](https://hackmd.io/YL3juOZrRH65XV51bZR6dQ?view)