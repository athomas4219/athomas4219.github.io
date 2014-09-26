---
layout: post
title: "OpenID Connect: Single Logout"
date: 2014-09-29 15:04:09 -04:00
comments: true
categories: sso
---

Working with single logout has been one of the last tasks we've addressed, but it has turned out to be one of the easiest. In our case, we're using a ReferenceID adapter, using our own web site to authenticate a user and deliver token information to Ping Federate. As such, our flow may be different than yours if you're using a different adapter. In our case, the flow is the following:

1. Using the SLO endpoint (/idp/startSLO.ping), one of the relying parties initiates a logout
1. Ping Federate receives the logout request, removes its own cookie, and posts to the adapter endpoint with parameters of resumePath and REF.
1. The referenceID adapter removes its own cookie, or whatever steps are necessary to remove a user's session
1. The referendeID adapter redirects to the resumePath with a query string parameter of REF
1. Ping Federate renders a page that contains an iframe for each client that has authenticated with the adapter
1. Javascript in the page performs a GET request to the configured endpoint for each client
1. The client endpoint removes its own user session and returns an image of a 1x1 transparent pixel
1. Once all clients have checked in, the javascript on the page displays a friendly message to the user indicating that logout has completed successfully.

### Ping Federate Configuration

### ReferenceID Adapter Logout Action

### Client Web Site Logout Action
