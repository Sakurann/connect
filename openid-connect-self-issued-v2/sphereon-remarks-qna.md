In the scope chapter it talks about the implicit flow on several occasions. For people not really into OpenID Connect it probably makes sense to link that to the respective spec
**->** addressed in a sphereon-comments PR https://bitbucket.org/openid/connect/pull-requests/61

## Self-Issued OpenID Provider Discovery

> Even if possible, such signals may be susceptible to fingerprinting and passive tracking of the End-user.

This sort of comes out of the blue. It doesn't really explain the problem, which might be confusing to people. Is the sentence really required?

**->** this text is proposed to be deleted in an incoming PR https://bitbucket.org/openid/connect/pull-requests/54

> When the RP wants to support the End-user's choice to select from multiple possible Self-Issued OP applications, it MAY present a static list of the available choices.

I think a bit more elaboration might help here. The assumption is that the RP knows upfront about SIOP applications it seems. How would it look like, since it is just cassually mentioned here. The next paragraph about trust frameworks is more clear about an actual implementation

**->** Elaborated a little more in an incoming PR https://bitbucket.org/openid/connect/pull-requests/54. Let us know if still not clear enough in that PR.

> The trust framework MAY be operated by just one RP, but due to the required maintenance of every application's metadata (which may change frequently) this burden SHOULD be shared across multiple RPs. The same trust framework MAY also be used to host metadata about the supported RPs such that the Self-Issued OP applications can verify the origin of the incoming request as part of the framework as well.

Is that really for this spec to determine/advice on? When looking at it, the first time it felt like a distraction to me

**->** an incoming PR https://bitbucket.org/openid/connect/pull-requests/54 elaborates a little more on Trust Frameworks. I think SIOP spec should never create a trust framework, but I do think that in real life implementations, trust frameworks has a great potential to strengthen trust model between SIOP and RP - for example with SIOP metadata discovery.

> ### Self-Issued OpenID Provider Discovery Metadata
> If the input identifier for the discovery process is the identifier `https://self-issued.me/v2

What is the input identifier? It comes out of the blue and is confusing as it talks about a static configuration later on, which suggests the possibility for dynamic configuration, but doesn't go into details what that entails (probably needs to point to dynamic registration)

**->** will address in an incoming SIOP metadata PR

> * REQUIRED. MUST include `openid:`, could also include additional custom schema.

It is confusing as in the previous chapter it talked about custom protocol schemas with this exact value for SIOPs. Next to that it talks about additional custom schemas. Does that mean you can have multiple endpoints? Does it mean the endpint still needs the string openid in the custom version of it?

**->** will address in an incoming SIOP metadata PR

> `subject_types_supported`
>    * REQUIRED. A JSON array of strings representing supported subject types. Valid values include `pairwise` and `public`.
The pairwise and public come out of the blue. Probably wise to point to the OpenID Connect Core spec
**->** addressed in a sphereon-comments PR.

> ## Relying Party Registration
> Note that in the Self-Issued OP flow no registration response is returned. A successful authentication response implicitly indicates that the registration parameters were accepted.

It might be wise to point out that the RP will typically hold state and that if the OP does not communicate an error to the RP as there is no defined way, this is something to take into account as it is a potential attack/DOS vector

**->** could you describe an attack a little more?

> RP Registration Metadata should preferably be sent by reference using the `registration_uri` parameter, but when RP cannot host a webserver, metadata parameters should be sent by value using the `registration` parameter.

It constantly talks about a URL and even mentions a webserver here. Although the most typical use case use a HTTPS URL as URI, why not make that a recommendation. The param is a URI, so if the RP and SIOP can talk the same schema/protocol it should be okay right?

**->** I'm not sure I understand. Recommendation is to host a file externally with registration parameters and pass them using registration_uri which is HTTPS URL (by reference). but when RP does not have a hosted environment, the only way is to include all paramters by value inside the request. Do you want it to be clarified that registration_uri is HTTPS URL?

> `registration` and `registration_uri` parameters SHOULD NOT be used when the OP is not a Self-Issued OP.

Why is this mentioned here? The whole spec is about SIOP to begin with

**->** Because trust model in SIOP is quite different from that of the rest of the OIDC and we do not want people to start using registration parameter in the SIOP libraries with usual OIDC - you never know what people do, so better to point out.

> * OPTIONAL. A JSON array of strings representing supported DID methods. Valid values are the Method Names listed in Chapter 9 of [@!did-spec-registries], such as `peer` or `web`. RPs can indicate support for any DID method by omitting `did_methods_supported`, while including `did` in `subject_identifier_types_supported`.

I removed the did:<method>:, as the DID method really only is the string 'peer' in did:peer:. It also interferes with implemntation libraries as they will accept the proper method name, meaning people have to strip off the prefix and colons

**->** makes sense. made a comment here: https://bitbucket.org/openid/connect/pull-requests/56#comment-258268298

It would probably be very helpful to be able to negate some did methods. For instance signaling `did` in `subject_identifier_types_supported`, but then for instance not allowing did-key, by using !key in did_methods_supported

**->** very interesting! If there is a requirement like that we should def consider adding this.

> When `request` or `request_uri` parameters are NOT present, `registration` or `registration_uri` parameters MUST be present in the request. When `request` or `request_uri` parameters are present, `registration` or `registration_uri` parameters MUST be included in either of those parameters.

This suggests that there could be multiple interactions (first return the registration info), but it is not really explicit about it.

**->** No, this does not mean multiple interactions are possible.. it simply means that registration should be included in request to prevent confusing SIOP if there are registration parameters both directily in the request and the request property. try clarify the language in sphereon-comments PR.
