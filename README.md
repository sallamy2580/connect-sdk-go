# 1Password Connect Go SDK

[![Go Reference](https://pkg.go.dev/badge/github.com/1Password/connect-sdk-go.svg)](https://pkg.go.dev/github.com/1Password/connect-sdk-go)
[![Go Report Card](https://goreportcard.com/badge/github.com/1Password/connect-sdk-go)](https://goreportcard.com/report/github.com/1Password/connect-sdk-go)
[![Version](https://img.shields.io/github/release/1Password/connect-sdk-go.svg)](https://github.com/1Password/connect-sdk-go/releases/)

The 1Password Connect Go SDK provides access to the [1Password Connect](https://support.1password.com/secrets-automation/) API, to facilitate communication with the Connect server hosted on your infrastructure and 1Password. The library is intended to be used by your applications, pipelines, and other automations to simplify accessing items stored in your 1Password vaults.

<details>
  <summary>Table of Contents</summary>

  - [1Password Connect Go SDK](#1password-connect-go-sdk)
  * [Prerequisites](#prerequisites)
  * [Installation](#installation)
  * [Usage](#usage)
    + [Quickstart](#quickstart)
    + [Creating an API Client](#creating-an-api-client)
    + [Model Objects](#model-objects)
    + [Item CRUD](#item-crud)
      - [Retrieving list of vaults that the Connect token has permission to read](#retrieving-list-of-vaults-that-the-connect-token-has-permission-to-read)
      - [Retrieving all items in a vault](#retrieving-all-items-in-a-vault)
      - [Retrieving item by title](#retrieving-item-by-title)
      - [Retrieving items by vault and item UUID](#retrieving-items-by-vault-and-item-uuid)
      - [Creating items in a vault](#creating-items-in-a-vault)
      - [Update and Item](#update-and-item)
      - [Delete an item](#delete-an-item)
      - [Retrieving a file from an item](#retrieving-a-file-from-an-item)
      - [Retrieving the contents of a file from an item](#retrieving-the-contents-of-a-file-from-an-item)
    + [Unmarshalling into a Struct](#unmarshalling-into-a-struct)
      - [Example Struct](#example-struct)
    + [Environment Variables](#environment-variables)
    + [Errors](#errors)
  * [Development](#development)
    + [Building](#building)
    + [Running Tests](#running-tests)
  * [Security](#security)
</details>

## Prerequisites

- [1Password Connect](https://support.1password.com/secrets-automation/#step-2-deploy-a-1password-connect-server) deployed in your infrastructure

## Installation
To download and install the 1Password Connect Go SDK, as well as its dependencies:
```sh
go get github.com/1Password/connect-sdk-go
```

## Usage

Below, you can find a selection of the most used functionality of the Connect Go SDK. For more detailed information about the content of the SDK, please refer to the [GoDocs](https://pkg.go.dev/github.com/1Password/connect-sdk-go). 

Import the 1Password Connect SDK:

```go
import (
    	"github.com/1Password/connect-sdk-go/connect"
	"github.com/1Password/connect-sdk-go/onepassword"
)
```

### Quickstart

Reading a secret:
```go
import "github.com/1Password/connect-sdk-go/connect"

func main () {
	client := connect.NewClient("<your_connect_host>", "<your_connect_token>")
	item, err := client.GetItem("<item-uuid>", "<vault-uuid>")
	if err != nil {
		log.Fatal(err)
	}
}
```
### Creating an API Client

A !password Connect client (`connect.Client`) is required to make requests to the Connect server via the 1Password Go SDK.
The client is configured with a token and a hostname. Three constructor methods are provided by that allow for creating the 1Password Connect client.

* `connect.NewClient` – Accepts a hostname and a token value.
```go
package main

import "github.com/1Password/connect-sdk-go/connect"

func main () {
	client := connect.NewClient("<your_connect_host>", "<your_connect_token>")
}
```

* `connect.NewClientFromEnvironment` – Fetches the hostname and token value from the environment, and expects these to be passed as environment variables (`OP_CONNECT_HOST` and `OP_CONNECT_TOKEN`, respectively).

Assuming that OP_CONNECT_TOKEN and OP_CONNECT_HOST have been set as environment variables, the `connect.NewClientFromEnvironment` can be invoked as such:
```go
package main

import "github.com/1Password/connect-sdk-go/connect"

func main () {
	client, err:= connect.NewClientFromEnvironment()
	if err != nil {
		panic(err)
	}
}
```

* `connect.NewClientWithUserAgent` – Accepts a hostname, a token value, and a custom User-Agent string for identifying the client to the 1Password Connect API:
```go
package main

import "github.com/1Password/connect-sdk-go/connect"

func main () {
	client := connect.NewClientWithUserAgent("<your_connect_host>", "<your_connect_token>", "Mozilla/5.0 (iPhone; CPU iPhone OS 10_3_2 like Mac OS X) AppleWebKit/603.2.4 (KHTML, like Gecko) FxiOS/8.1.1b4948 Mobile/14F89 Safari/603.2.4")
}
```

[comment]: <> (The following subsection will be modified upon the merging of PR #31)

### Model Objects

The `onepassword.Item` model represents items and `onepassword.Vault` represents vaults, in 1Password.

### Item CRUD

The `connect.Client` also supports methods for:

#### Retrieving list of vaults that the Connect token has permission to read
```go
	vaults, err := client.GetVaults()
	if err != nil {
		log.Fatal(err)
	}
```
#### Retrieving all items in a vault
```go
	items, err := client.GetItems("<vault-uuid>")
	if err != nil {
		log.Fatal(err)
	}
```
#### Retrieving item by title
To retrieve all items in a vault with a given title:
```go
	items, err := client.GetItemsByTitle("<items-title>", "<vault-uuid>")
	if err != nil {
		log.Fatal(err)
	}
```
In case the item title is unique for a vault, another function is available as well, returning only one item, instead of a slice:
```go
	item, err := client.GetItemByTitle("<item-title>", "<vault-uuid>")
	if err != nil {
		log.Fatal(err)
	}
```
#### Retrieving items by vault and item UUID
```go
	item, err := client.GetItem("<item-uuid>", "<vault-uuid>")
	if err != nil {
		log.Fatal(err)
	}
```
#### Creating items in a vault
```go
	item := &onepassword.Item{
		Fields: []*onepassword.ItemField{{
			Value: "mysecret",
			Type: "STRING",
		}},
		Tags:     []string{"1password-connect"},
		Category: onepassword.Login,
		Title:    "Secret String",
	}

	postedItem, err := client.CreateItem(item, "<vault-uuid>")
	if err != nil {
		log.Fatal(err)
	}
```
#### Update and Item
```go
	item, err := client.GetItem("<item-uuid>", "<vault-uuid>")
	if err != nil {
		log.Fatal(err)
	}
	item.Title = "new title"
	client.UpdateItem(item, "<vault-uuid>")
```
#### Delete an item
```go
	item, err := client.GetItem("<item-uuid>", "<vault-uuid>")
	if err != nil {
		log.Fatal(err)
	}
	err = client.DeleteItem(item, "vault-uuid")
```
#### Retrieving a file from an item
```go
	file, err := client.GetFile("<file-uuid>", "item-uuid", "vault-uuid")
	if err != nil {
		log.Fatal(err)
	}
```
#### Retrieving the contents of a file from an item
```go
	file, err := client.GetFile("<file-uuid>", "item-uuid", "vault-uuid")
	if err != nil {
		log.Fatal(err)
	}
	content, err := client.GetFileContent(file)
	if err != nil {
		log.Fatal(err)
	}
```

### Unmarshalling into a Struct

Users can define tags on a struct and have the `connect.Client` unmarshall item data directly in them. Supported field tags are:

- `opvault` – The UUID of the vault the item should come from
- `opitem` – The title of the Item
- `opfield` – The item field whose value should be retrieved

#### Example Struct

This example struct will retrieve 3 fields from one Item and a whole Item from another vault

```go
package main

import (
	"github.com/1Password/connect-sdk-go/connect"
	"github.com/1Password/connect-sdk-go/onepassword"
)

type Config struct {
	Database string           `opitem:"Demo TF Database" opfield:".database"`
	Username string           `opitem:"Demo TF Database" opfield:".username"`
	Password string           `opitem:"Demo TF Database" opfield:".password"`
	APIKey   onepassword.Item `opvault:"7vs66j55o6md5btwcph272mva4" opitem:"API Key"`
}

var client connect.Client

func main() {
	client, err := connect.NewClientFromEnvironment()
	if err != nil {
		panic(err)
	}
	
	connect.Load(client, &c)
}

```

### Environment Variables

The Connect Go SDK makes use of the following environment variables:
* `OP_CONNECT_TOKEN`: the API token to be used to authenticate the client to your 1Password Connect instance. Used in order to authenticate via the `connect.NewClientFromEnvironment` function.
* `OP_CONNECT_HOST`: the hostname of your 1Password Connect instance. Used in order to authenticate via the `connect.NewClientFromEnvironment` function.
* `OP_VAULT`: a vault UUID. Used as default vault in the `LoadConfig` function, for all fields where the `.opvault` tag is not set.


### Errors
All errors returned by Connect API are unmarshalled into a `onepassword.Error` struct:
```go
type Error struct {
    StatusCode int    `json:"status"`
    Message    string `json:"message"`
}
```

Details of the errors can be accessed by using `errors.As()`:
```go
_, err := client.GetVaults()
if err != nil{
    var opErr *onepassword.Error
    if errors.As(err, &opErr){
        fmt.Printf("message=%s, status code=%d\n",
            opErr.Message,
            opErr.StatusCode,
        )
    }
}
```
## Development

### Building

To build all packages:

```sh
make build
```

### Running Tests

Run all tests:

```sh
make test
```

Run all tests with code coverage:

```sh
make test/coverage
```

## Security

1Password requests you practice responsible disclosure if you discover a vulnerability.

Please file requests via [**BugCrowd**](https://bugcrowd.com/agilebits).

For information about security practices, please visit our [Security homepage](https://bugcrowd.com/agilebits).
