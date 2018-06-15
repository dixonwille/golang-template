# golang-template

## Installing for local development

1. `go get -d -u github.com/dixonwille/golang-template/cmd/...`
2. `cd $(go env GOPATH)/src/github.com/dixonwille/golang-template`
3. `cp .env.default .env`
4. modify .env for your local environment

### Running development

1. `go run ./cmd/serviceName/*.go`

## File Structure

All directories and files should be named exactly like they are here except for `serviceName` which should be replaced with your actual service name.

### cmd/serviceName/

This keeps the package that generates a binary separate from your business logic. Also the binary will be named the name of the directory.

There are two files in here. One to keep up with configs that are environment variables, `config.go`. The other is the actual main function. Both of these should stay relatively small as their only purpose is to setup a few things then go into another package.

> **NOTE**: Sometimes it makes sense to have multiple binaries for a single project. You would create another folder under `cmd/` and do something very similar to the current structure (set things up but pass execution to a package). Such extra binaries may be testing clients or data migration specific to this service.

### internal/

This folder is special in Go. Only packages that are siblings to this package (as well as cousins) can use the packages in this folder. This means that even though the code is viewable, any external package outside of this repository can't use the package.

The `service/` folder under here holds most of your business logic. The reason we put this folder here is the only logical way to call this business logic is through the service you create, not some other package.

Other folders that may exist here is a store package that deals with data store or a queue package dealing with your queues.

This is great for gRPC and API projects but rules can be broken if it makes sense. If there is a package that you are expecting someone to consume, keep it outside of this folder.

### rpc/serviceName/

This is where your `.proto` file will live and your generated code. The reason for having it in a subdirectory named `serviceName` is so that your generated code live in the proper location for the `go_package` name.

### vendor/

This is maintained by [Dep](#install_dep)

### .env.default

Should have reasonable default values for a service to be ran. This should include the config keys for credentials but supply dummy set of values.

### Gopkg.lock and Gopkg.toml

These files are [Dep](#install_dep) specific. The only file you should manually change if needed is `Gopkg.toml`. Read more about [Dep](#install_dep) below.

### Other Files

A few more files may exist at the root level. This may include linting tool configuration, Continuous Integration service configuration, or docker compose files to standup resources for local development. It is best to keep root files to just a few files, but sometimes they are needed and can make a developers life easier.

## Go Docs

Creating comments above public types/funcs and packages allows Go to generate documentation for you. There is a greate online webpage called [GoDoc](https://godoc.org). It will show documentation for open source packages and tool online in a pretty format. For example https://godoc.org/github.com/dixonwille/skywalker. If you will like to read more about this I would start [here](https://blog.golang.org/godoc-documenting-go-code).

## Generate gRPC Files

1. Run `go generate ./rpc/serviceName`

> **NOTE**: The first line comment in `./rpc/serviceName/doc.go` is what is being ran. Read [here](https://blog.golang.org/generate) for more.

## Vendoring

The reason for vendoring is so that all machines that will run your code will use the same version of all dependent packages. This creates repeatable builds. It also keeps a version of the package in source control if the package was removed for some reason.

### Install Dep

There are a few ways to install Dep. You can either `go get` or use homebrew. For more details: https://github.com/golang/dep#installation

### To initialize a repository without vendoring

1. Run `dep init` in the root of the repository

### Add a package to Dep before consuming

1. Run `dep ensure -add github.com/org/repo`

> **NOTE**: This will get removed if you run the next command and you have not consumed the package yet.

### Add a package after already consuming it

1. Run `dep ensure`

### Read More

- https://golang.github.io/dep/docs/introduction.html

## Linting

Linting is important as it enforces everyone to write code in a similar manner. It can also help reduce bugs and make it easier to maintain because it enforces certain rules.

You should research both and figure out which one you want to use. Keep in mind Continuous Integration and how easy it can be used in CI services such as [Travis-CI](https://travis-ci.com/).

### Gometalinter

https://github.com/alecthomas/gometalinter

Gometalinter is tried and true. But it lacks in speed. It runs all of its linters as separate binaries and most binaries need to be installed on your machine to run them properly. This tool is used by most editors already to do the auto linting on file save.

### GolangCI-Lint

https://github.com/golangci/golangci-lint

I recomend this linter as it is faster and does not require you to install other binaries. But it does not have the huge variety of linters as Gometalinter. But GolangCI-Lint as the major ones most repositories want/need.

## Continuous Integration and Deployment

Continuous Integration and Deployment makes a developers life a lot easier by checking their code by linting and running test. Those two task are usually ran for every Pull Request which is enforced and needed to pass before merging. This helps catch possible bugs from making it to the master branch. Once on the master branch, Continuous Deployment kicks in. It will build your code and deploy it to a server. Keep these things in mind while you are working on your repository to make your life easier.
