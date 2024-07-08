+++
title = 'Creating Protoc Plugin for Go code generation'
date = 2024-07-08T20:55:42+08:00
draft = true
+++

This weekend i've been expoloring new topic for me code generation from protobuf files and how to create custom plugins for `protoc` utility.
I needed to generate in addition to gRPC service implementation, custom RPC protocol via Redis streams and channels.

without futher a do lets gets started.

# Creating first plugin

I didn't know where to start and started to googling in hope to find some how to guide for writing `protoc` plugins, [this blog post](https://rotemtam.com/2021/03/22/creating-a-protoc-plugin-to-gen-go-code/) for creating simple "hello world" plugin really helped me started. Main points that i picked up from this article:

- what need to be installed on my machine to be able generate code from protobuf file:
  - [protoc](https://grpc.io/docs/protoc-installation/)
  - [prtoc-ge-go](https://grpc.io/docs/languages/go/quickstart/)
- You need to name your executable in certain way to make it available for protoc utility. like following `protoc-gen-<name-of-your-plugin>`
- and the most importang it's provided code example for plugin that I was able to start from:

```go
package main

import (
	"google.golang.org/protobuf/compiler/protogen"
)

func main() {
	protogen.Options{}.Run(func(gen *protogen.Plugin) error {
		for _, f := range gen.Files {
			if !f.Generate {
				continue
			}
			generateFile(gen, f)
		}
		return nil
	})
}

// generateFile generates a _ascii.pb.go file containing gRPC service definitions.
func generateFile(gen *protogen.Plugin, file *protogen.File) {
	filename := file.GeneratedFilenamePrefix + "_grpc-redis.pb.go"
	g := gen.NewGeneratedFile(filename, file.GoImportPath)
	g.P("// Code generated by protoc-gen-grp-redis. DO NOT EDIT.")
	g.P()
	g.P("package ", file.GoPackageName)
	g.P()

	return g
}
```
  