---
title: Getting Started
pageTitle: "Getting Started with GraphQL & Go Backend Tutorial"
description: "Learn how to setup a GraphQL server with graphql-go and best practices for defining a GraphQL schema."
---

### Setup the project
This tutorial assumes some familiarity with Go and that you have a Go workspace setup. If not, you can quickly get
setup by following the [How to Write Go Code](https://golang.org/doc/code.html) documentation.

<Instruction>

Create a directory inside your workspace in which to keep source code (don't forget to fill in your user name):

```bash(path=".../")
mkdir $GOPATH/src/github.com/{your_user}/gohackernews && cd $GOPATH/src/github.com/{your_user}/gohackernews```
```
</Instruction>

<Instruction>

Next, create a file named main.go inside that directory, containing the following Go code.

```go
package main

import (
	"log"
	"net/http"

	"github.com/neelance/graphql-go"
	"github.com/neelance/graphql-go/relay"
	"github.com/howtographql/graphql-go/resolvers"
	"io/ioutil"
	"fmt"
)

var schema *graphql.Schema

func init() {
	schemaFile, err := ioutil.ReadFile("schema.graphqls")
	if err != nil {
		panic(err)
	}

	schema = graphql.MustParseSchema(string(schemaFile), &resolvers.Resolver{})
}

func main() {
	http.Handle("/query", &relay.Handler{Schema: schema})

	fmt.Println("server is running on port 8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```
</Instruction>

### Define the Schema
You may have noticed we're reading our schema from a file called schema.graphqls. We're using the [Schema Definition Language](http://graphql.org/learn/schema/#type-language) (SDL) - where the schema is generated from a textual language-independent description then wired dynamically.

For this tutorial we'll be using SDL to provide a clear distinction between data and behavior.

If you prefer to have your fields and resolvers collocated you can also define your types programmatically. An example of that can be found [here](https://github.com/neelance/graphql-go/blob/master/example/starwars/starwars.go).

Create a file called schema.graphqls in the root of your project.

<Instruction>
The SDL definition for a simple type representing a link might look like this:

```graphql(path=".../hackernews-graphql-go/schema.graphqls")
type Link {
  url: String!
  description: String!
}
```

The query to fetch all links could be defined as:

```graphql(path=".../hackernews-graphql-go/schema.graphqls")
type Query {
  allLinks: [Link]
}
```

The schema containing this query would be defined as:

```graphql(path=".../hackernews-graphql-go/schema.graphqls")
schema {
  query: Query
}
```

Save each of these definitions in your `schema.graphqls`

</Instruction>

Let's try to build and install the program using the go tool:

```bash(path=".../")
go install github.com/{your_user}/gohackernews
```

You should see:

```bash(path=".../")
./main.go:21:59: undefined: Resolver
```

This won't compile. Which makes a lot of sense since we haven't defined any resolvers. We'll be doing that soon but for now let's do a little more initial project setup.

### Install dependencies
For the rest of this tutorial we're going to use [dep](https://github.com/golang/dep) to manage our dependencies.

If you don't currently use dep you can follow the [setup](https://github.com/golang/dep#setup) to get started.

Assuming you have dep run:

```bash(path=".../")
dep init
```

You can expect to see something like:

```bash(path=".../")
Using master as constraint for direct dep github.com/neelance/graphql-go
Locking in master (10eb949) for direct dep github.com/neelance/graphql-go
Locking in v1.0.2 (1949ddb) for transitive dep github.com/opentracing/opentracing-go
Locking in master (0a93976) for transitive dep golang.org/x/net
```

Gopkg.lock & Gopkg.toml files should be generated as well as a vendor directory.

Now it's time to build our initial resolver so the allLinks query has a way to execute.

