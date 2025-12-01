![Logo](https://www.oidc.se/img/oidc-logo.png)

# OpenID Federation Hosting 1.0

This repository contains the OpenID Federation specification for how to host subject Entity Configurations at an Intermediate Entity.

> Some more info about the specification


## Builds

The latest released draft of the specification is available at [https://www.oidc.se/specifications/openid-federation-hosting-1_0.html](https://www.oidc.se/specifications/openid-federation-hosting-1_0.html).

Previews for each branch are automatically built and published. You can view the preview of the main branch at [https://www.oidc.se/openid-federation-hosting/main.html](https://www.oidc.se/openid-federation-hosting/main.html).
Other branches are built and deployed as well, using the branch name as the HTML filename, as shown below:

> `https://www.oidc.se/openid-federation-hosting/$BRANCH-NAME.html`

## Build the HTML ##

```docker run -v `pwd`:/data danielfett/markdown2rfc openid-federation-hosting-1_0.md```

## Contact

For further information and to get involved, please contact the authors.
