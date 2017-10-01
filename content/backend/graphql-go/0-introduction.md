---
title: Introduction
pageTitle: "Building a GraphQL Server with Go Backend Tutorial"
description: "Learn how to build a GraphQL server with graphql-go and best practices for filters, authentication, pagination and subscriptions."
---

### Motivation

[Go](https://golang.org//) is an open source programming language that makes it easy to build simple, reliable, and efficient software.

With high performance and efficient concurrency handling while embracing simplicity it's easy to see why some refer to Go as the server language of the future. Also, the gopher logo is pretty adorable.

### What is a GraphQL server?

A GraphQL server should be able to:

* Parse, validate, and execute queries and mutations based on a schema definition
* Connect to any databases or services responsible for storing or fetching data
* Respond based on the specific data requested

You can read more details of the features of GraphQL servers in the [official specification](https://facebook.github.io/graphql/).

### Schema-driven development

An important thing to note about building a GraphQL server is that the main development process will revolve around the schema definition. You'll see in this chapter that the main steps we'll follow will be something like this:

1. Define your types and the appropriate queries and mutations for them.
2. Implement functions called **resolvers** to handle these types and their fields.
3. As new requirements arrive, go back to step 1 to update the schema and continue through the other steps.

The schema is a *contract* agreed on between the frontend and backend, so keeping it at the center allows both sides of the development to evolve without going off the spec. This also makes it easier to parallelize the work, since the frontend can move on with full knowledge of the API from the start, using a simple mocking service (or even a full backend such as [Graphcool](https://www.graph.cool/)) which can later be easily replaced with the final server.
