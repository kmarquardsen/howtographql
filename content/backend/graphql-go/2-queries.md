---
title: Queries
pageTitle: "Resolving Queries with a Go GraphQL Server Tutorial"
description: "Learn how to define the GraphQL schema, implement query resolvers in Go, and test your queries in a GraphiQL Playground."
question: Is a special client required to query a GraphQL server?
answers: ["Yes, and there is an implementation in each language", "Yes, you need GraphiQL to create and issue queries", "No, GraphQL servers are always exposed over HTTP", "No, a GraphQL server could be exposed over any transport"]
correctAnswer: 3
---

### Query resolvers

To represent GraphQL types in Go we define each type as a Struct, and fields as functions. 
Structs model the domain, and *resolvers*, model the queries and mutations and contain the resolver functions. 
Both are needed to model a single GraphQL type.

The schema so far looks like this:

```graphql(nocopy)
type Link {
  url: String!
  description: String!
}

type Query {
  allLinks: [Link]
}

schema {
  query: Query
}
```

To model it, let's first create a `resolvers` directory containing a `resolvers.go` file.

<Instruction>

`Link` is a simple struct, so create it as follows:
```go(path=".../hackernews-graphql-go/resolvers/resolvers.go")
type Link struct {
	url         string
	description string
}
```

Let's initialize some sample link data

```go(path=".../hackernews-graphql-go/resolvers/resolvers.go")
var allLinks = []*Link{
	{
		url:         "http://howtographql.com",
		description: "The best resource for learning GraphQL",
	},
	{
		url:         "https://golang.org/",
		description: "A language that makes it easy to build simple, reliable, and efficient software.",
	},
}

var linkData = make(map[string]*Link)

func init() {
	for _, l := range allLinks {
		linkData[l.url] = l
	}
}
```

Additionally "a resolver must have one method for each field of the GraphQL type it resolves. The method name has to be exported and match the field's name in a non-case-sensitive way."
See https://github.com/neelance/graphql-go#resolvers for more information.

Let's add the necessary resolvers for allLinks and each of it's fields.

```go(path=".../hackernews-graphql-go/resolvers/resolvers.go")
type Resolver struct{}

func (r *Resolver) AllLinks() *[]*linkResolver {
	var l []*linkResolver
	for _, link := range allLinks {
		l = append(l, &linkResolver{link})
	}
	return &l
}

type linkResolver struct {
	l *Link
}

func (r *linkResolver) URL() string {
	return r.l.url
}

func (r *linkResolver) Description() string {
	return r.l.description
}
```

Next let's add graphiql support to our app to test things out.

### Testing with GraphiQL

[GraphiQL](https://github.com/graphql/graphiql) is an in-browser IDE allowing you to explore the schema, fire queries/mutations and see the results.

<Instruction>

To add GraphiQL, and the following template to the `main.go`:

```go(path=".../hackernews-graphql-go/main.go")
var page = []byte(`
<!DOCTYPE html>
<html>
	<head>
		<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/graphiql/0.10.2/graphiql.css" />
		<script src="https://cdnjs.cloudflare.com/ajax/libs/fetch/1.1.0/fetch.min.js"></script>
		<script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.5.4/react.min.js"></script>
		<script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.5.4/react-dom.min.js"></script>
		<script src="https://cdnjs.cloudflare.com/ajax/libs/graphiql/0.10.2/graphiql.js"></script>
	</head>
	<body style="width: 100%; height: 100%; margin: 0; overflow: hidden;">
		<div id="graphiql" style="height: 100vh;">Loading...</div>
		<script>
			function graphQLFetcher(graphQLParams) {
				return fetch("/query", {
					method: "post",
					body: JSON.stringify(graphQLParams),
					credentials: "include",
				}).then(function (response) {
					return response.text();
				}).then(function (responseBody) {
					try {
						return JSON.parse(responseBody);
					} catch (error) {
						return responseBody;
					}
				});
			}
			ReactDOM.render(
				React.createElement(GraphiQL, {fetcher: graphQLFetcher}),
				document.getElementById("graphiql")
			);
		</script>
	</body>
</html>
`)
```

Then add the supporting handler to the main function:

```go(path=".../hackernews-graphql-go/main.go")
http.Handle("/", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    w.Write(page)
}))
```

</Instruction>

If you now run the application and open http://localhost:8080/ you should see the awesome GraphiQL IDE.

![](http://i.imgur.com/KlnKaZe.png)

If you type the following query into the left panel (notice the auto-completion!) and hit the play icon, you should get the following result:

![](https://i.imgur.com/bHVQroR.png)

Congratulations! We'll continue using GraphiQL to test out queries and mutation throughout the tutorial.
