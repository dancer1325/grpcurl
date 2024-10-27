# gRPCurl
[![Build Status](https://circleci.com/gh/fullstorydev/grpcurl/tree/master.svg?style=svg)](https://circleci.com/gh/fullstorydev/grpcurl/tree/master)
[![Go Report Card](https://goreportcard.com/badge/github.com/fullstorydev/grpcurl)](https://goreportcard.com/report/github.com/fullstorydev/grpcurl)

* == CL tool / -- lets interact with -- gRPC servers
  * == `curl` -- for -- gRPC servers
  * goal
    * invoke RPC methods | gRPC server -- from the -- CL
  * available inputs
    * 👀[messages](https://protobuf.dev/overview/) -- via -- JSON encoding 👀
  * allows ALSO
    * browsing the schema -- for -- gRPC services
      * via 
        * querying a server / supports [server reflection](https://github.com/grpc/grpc/blob/master/src/proto/grpc/reflection/v1/reflection.proto)
        * reading ".proto"
        * loading | compiled "protoset" files
          * := files / contain encoded file [descriptor protos](https://github.com/google/protobuf/blob/master/src/google/protobuf/descriptor.proto) 
* gRPC servers -- use a -- binary encoding | wire ([protocol buffers](https://developers.google.com/protocol-buffers/))
  * -> impossible to interact -- via -- `curl`
* `github.com/fullstorydev/grpcurl`
  * == library package / 
    * functions -- simplify the -- construction of other CL tools / -- dynamically invoke -- gRPC endpoints
* check [`grpcurl` talk at GopherCon 2018](https://www.youtube.com/watch?v=dDr-8kbMnaw)
  * TODO: Create a file to take notes

## Features

* support
  * ALL kinds of RPC methods
    * _Examples:_
      * streaming methods
      * operate bi-directional streaming methods interactively -- via -- 
        * running `grpcurl` | interactive terminal &
        * using stdin -- as the -- request body
  * secure/TLS servers / 
    * several options for TLS configuration
  * plain-text servers (== NO TLS)
  * mutual TLS
    * requirements
      * client -- is required to -- present a client certificate

## Installation

### Binaries

* Download the binary | [releases](https://github.com/fullstorydev/grpcurl/releases)

### Homebrew (macOS)

  ```shell
  brew install grpcurl
  ```

### Docker

* 
  ```shell
  # Download image
  docker pull fullstorydev/grpcurl:latest
  # Run the tool
  docker run fullstorydev/grpcurl api.grpc.me:443 list
  ```
  * notes
    * if you need to interact with a server / listens | host's loopback network -> specify the host as
      * | Mac or Windows, `host.docker.internal` _OR_ 
      * | Linux, `-network="host"`
    * if you need to provide ".proto" OR descriptor sets -> 
      * mount the folder / contains the files -- as a -- volume (`-v $(pwd):/protos`) &
      * adjust the import paths -- to -- container paths
    * if you want to provide the request message -- via -- stdin (`-d @` option) -> use the `-i` flag | docker command

### Other Packages

* TODO:
There are numerous other ways to install `grpcurl`, thanks to support from third parties that
have created recipes/packages for it. These include other ways to install `grpcurl` on a variety
of environments, including Windows and myriad Linux distributions.

You can see more details and the full list of other packages for `grpcurl` at _repology.org_:
https://repology.org/project/grpcurl/information

### From Source

If you already have the [Go SDK](https://golang.org/doc/install) installed, you can use the `go`
tool to install `grpcurl`:
```shell
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
```

This installs the command into the `bin` sub-folder of wherever your `$GOPATH`
environment variable points. (If you have no `GOPATH` environment variable set,
the default install location is `$HOME/go/bin`). If this directory is already in
your `$PATH`, then you should be good to go.

If you have already pulled down this repo to a location that is not in your
`$GOPATH` and want to build from the sources, you can `cd` into the repo and then
run `make install`.

If you encounter compile errors and are using a version of the Go SDK older than 1.13,
you could have out-dated versions of `grpcurl`'s dependencies. You can update the
dependencies by running `make updatedeps`. Or, if you are using Go 1.11 or 1.12, you
can add `GO111MODULE=on` as a prefix to the commands above, which will also build using
the right versions of dependencies (vs. whatever you may already have in your `GOPATH`).

---

## Usage

```shell
grpcurl -help
```

* arguments
  * MUST be passed before
    * server address &
    * method name

### Invoking RPCs

* | trusted server / supports server reflection
  * trusted server
    * == TLS / NO
      * self-signed key or
      * custom CA
  *  SIMPLEST thing
  * _Examples:_
    * _Example1:_ invocation / sends an empty request body

      ```shell
      grpcurl grpc.server.com:443 my.custom.server.Service/Method
  
      # no TLS
      grpcurl -plaintext grpc.server.com:80 my.custom.server.Service/Method
      ```

    * _Example2:_ invocation / sends a NON-empty request body
    
      ```shell
      # `-d` argument -- to pass -- request body in JSON format
      grpcurl -d '{"id": 1234, "tags": ["foo","bar"]}' \
          grpc.server.com:443 my.custom.server.Service/Method
      ```
* if you want to 
  * pass arguments -> use `-d`
    * _Example:_ check previous request body example
  * include `grpcurl` | command pipeline -> use `-d @` -> read the actual request body | stdin
    * _Example:_ 

        ```shell
        grpcurl -d @ grpc.server.com:443 my.custom.server.Service/Method <<EOM
        {
        "id": 1234,
        "tags": [
            "foor",
            "bar"
        ]
        }
        EOM
        ```

### Adding Headers/Metadata to Request

* TODO:
Adding of headers / metadata to a rpc request is possible via the `-H name:value` command line option. Multiple headers can be added in a similar fashion.
Example :
```shell
grpcurl -H header1:value1 -H header2:value2 -d '{"id": 1234, "tags": ["foo","bar"]}' grpc.server.com:443 my.custom.server.Service/Method
```
For more usage guide, check out the help docs via `grpcurl -help`

### Listing Services

* allows
  * listing ALL services / 
    * if you use
      * server reflection -> -- exposed by a -- server
      * `.proto` source -> defined | here 
      * protoset files -> defined | here
  * seeing ALL methods | particular service

  ```shell
  # Server supports reflection
  grpcurl localhost:8787 list
  
  # Using compiled protoset files
  grpcurl -protoset my-protos.bin list
  
  # Using proto sources
  grpcurl -import-path ../protos -proto my-stuff.proto list
  
  # Export proto files (use -proto-out-dir to specify the output directory)
  grpcurl -plaintext -proto-out-dir "out_protos" "localhost:8787" describe my.custom.server.Service
  
  # Export protoset file (use -protoset-out to specify the output file)
  grpcurl -plaintext -protoset-out "out.protoset" "localhost:8787" describe my.custom.server.Service
  
  # list methods | particular service
  grpcurl localhost:8787 list my.custom.server.Service
  ```

### Describing Elements

* TODO:
The "describe" verb will print the type of any symbol that the server knows about
or that is found in a given protoset file. It also prints a description of that
symbol, in the form of snippets of proto source. It won't necessarily be the
original source that defined the element, but it will be equivalent.

```shell
# Server supports reflection
grpcurl localhost:8787 describe my.custom.server.Service.MethodOne

# Using compiled protoset files
grpcurl -protoset my-protos.bin describe my.custom.server.Service.MethodOne

# Using proto sources
grpcurl -import-path ../protos -proto my-stuff.proto describe my.custom.server.Service.MethodOne
```

## Descriptor Sources
The `grpcurl` tool can operate on a variety of sources for descriptors. The descriptors
are required, in order for `grpcurl` to understand the RPC schema, translate inputs
into the protobuf binary format as well as translate responses from the binary format
into text. The sections below document the supported sources and what command-line flags
are needed to use them.

### Server Reflection

* used by default by `grpcurl`
  * == NO additional command-line flags
* [server reflection](https://github.com/grpc/grpc/blob/master/src/proto/grpc/reflection/v1/reflection.proto)
* [how to set up](https://github.com/grpc/grpc/blob/master/doc/server-reflection.md#known-implementations)

When using reflection, the server address (host:port or path to Unix socket) is required
even for "list" and "describe" operations, so that `grpcurl` can connect to the server
and ask it for its descriptors.

### Proto Source Files
To use `grpcurl` on servers that do not support reflection, you can use `.proto` source
files.

In addition to using `-proto` flags to point `grpcurl` at the relevant proto source file(s),
you may also need to supply `-import-path` flags to tell `grpcurl` the folders from which
dependencies can be imported.

Just like when compiling with `protoc`, you do *not* need to provide an import path for the
location of the standard protos included with `protoc` (which contain various "well-known
types" with a package definition of `google.protobuf`). These files are "known" by `grpcurl`
as a snapshot of their descriptors is built into the `grpcurl` binary.

When using proto sources, you can omit the server address (host:port or path to Unix socket)
when using the "list" and "describe" operations since they only need to consult the proto
source files.

### Protoset Files
You can also use compiled protoset files with `grpcurl`. If you are scripting `grpcurl` and
need to re-use the same proto sources for many invocations, you will see better performance
by using protoset files (since it skips the parsing and compilation steps with each
invocation).

Protoset files contain binary encoded `google.protobuf.FileDescriptorSet` protos. To create
a protoset file, invoke `protoc` with the `*.proto` files that define the service:
```shell
protoc --proto_path=. \
    --descriptor_set_out=myservice.protoset \
    --include_imports \
    my/custom/server/service.proto
```

The `--descriptor_set_out` argument is what tells `protoc` to produce a protoset,
and the `--include_imports` argument is necessary for the protoset to contain
everything that `grpcurl` needs to process and understand the schema.

When using protosets, you can omit the server address (host:port or path to Unix socket)
when using the "list" and "describe" operations since they only need to consult the
protoset files.

