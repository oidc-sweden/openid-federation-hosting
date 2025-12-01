![Logo](https://www.oidc.se/img/oidc-logo.png)

# OpenID Federation Hosting 1.0

This repository contains an OpenID Federation specification defining an extension to OpenID Federation that enables Subordinate Statements to include a claim indicating where a subject’s Entity Configuration is located. The extension introduces the `ec_location` claim, which specifies an alternative location for retrieving the subject’s Entity Configuration. This functionality allows Entity Configuration data to be hosted at an Intermediate Entity, which can be essential for enabling legacy systems or non-federation capable entities to register and operate within a federation.

## Builds

The latest released draft of the specification is available at [https://www.oidc.se/specifications/openid-federation-hosting-1_0.html](https://www.oidc.se/specifications/openid-federation-hosting-1_0.html).

Previews for each branch are automatically built and published. You can view the preview of the main branch at [https://www.oidc.se/openid-federation-hosting/main.html](https://www.oidc.se/openid-federation-hosting/main.html).
Other branches are built and deployed as well, using the branch name as the HTML filename, as shown below:

> `https://www.oidc.se/openid-federation-hosting/$BRANCH-NAME.html`

## Build the HTML ##

```docker run -v `pwd`:/data danielfett/markdown2rfc openid-federation-hosting-1_0.md```

## Contact

For further information and to get involved, please contact the authors.
