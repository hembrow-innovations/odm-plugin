# `odm-plugin`

This repository provides a Go library, `odm-plugin`, designed to simplify the creation of plugins for the `odm` CLI tool. It leverages `github.com/hashicorp/go-plugin` to establish a robust and extensible plugin system, allowing you to extend `odm`'s functionality with custom plugins.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
  - [Defining Your Plugin Logic](#defining-your-plugin-logic)
  - [Implementing the Plugin Entrypoint](#implementing-the-plugin-entrypoint)
  - [Building Your Plugin](#building-your-plugin)
- [Contributing](#contributing)
- [License](#license)

---

## Features

- **Simplified Plugin Development**: Provides interfaces and structures to quickly develop `odm` plugins.
- **`go-plugin` Integration**: Built on `hashicorp/go-plugin` for reliable inter-process communication between `odm` and your plugins.
- **Standardized Execution Interface**: Defines a clear `Executer` interface for handling plugin execution requests.
- **Structured Request Body**: Offers a `ExecutionRequestBody` struct for easy parsing of arguments, options, and input from `odm`.

---

## Installation

To use this library, you'll need to have Go installed. Then, you can simply import it into your Go project:

```bash
go get your-repository-path/odm-plugin # Replace with the actual path to this repository
```

**Note**: Since this is a library for creating plugins, you won't directly "run" this repository. Instead, you'll import it into your own plugin projects.

---

## Usage

Creating an `odm` plugin using this library involves two main steps: defining your plugin's execution logic and setting up the plugin's main entrypoint.

### Defining Your Plugin Logic

Your custom plugin logic must implement the `odmplugin.Executer` interface. This interface defines a single method: `Execute`.

```go
package odmplugin

import (
	"context"
)

// Executer is the interface that our plugin will implement.
type Executer interface {
	Execute(ctx context.Context, body string) (string, error)
}
```

The `Execute` method receives a `context.Context` and a `string` representing the request body. This `body` string will contain a JSON-encoded `odmplugin.ExecutionRequestBody` which includes `Args`, `Options`, and `Input`.

Here's an example of how you might implement the `Executer` interface:

```go
package main

import (
	"context"
	"encoding/json"
	"log"

	odmPlugin "odm-plugin" // Replace with odm-plugin library path
)

// ExecuterImpl is the concrete implementation of our Executer interface.
type ExecuterImpl struct{} // No need to embed plugin.Plugin here for the Impl

// Execute implements the odmplugin.Executer interface.
func (g *ExecuterImpl) Execute(ctx context.Context, body string) (string, error) {
	requestBody := &odmPlugin.ExecutionRequestBody{}

	err := json.Unmarshal([]byte(body), requestBody)
	if err != nil {
		return "", err
	}

	log.Printf("Plugin: Execute called with body: \n\tArguments:%+v\n\tInput: %s\n\tOptions: %+v", requestBody.Args, requestBody.Input, requestBody.Options)

	// Your custom plugin logic goes here.
	// You can access requestBody.Args, requestBody.Input, and requestBody.Options.

	return "Success from my custom plugin!", nil
}
```

In this example:

- We define `ExecuterImpl` which will contain our plugin's specific logic.
- The `Execute` method unmarshals the `body` string into an `odmplugin.ExecutionRequestBody` for easy access to the command's arguments, options, and input.
- You would replace the `log.Printf` and hardcoded return with your actual plugin functionality.

### Implementing the Plugin Entrypoint

The `main` function of your plugin needs to set up the `go-plugin` server, telling it how to serve your `Executer` implementation.

```go
package main

import (
	"log"

	odmPlugin "odm-plugin" // Replace with odm-plugin library path

	"github.com/hashicorp/go-plugin"
)

func main() {
	log.Println("Starting Tester plugin...")

	// The plugin must export an implementation of the Executer interface.
	var handshakeConfig = odmPlugin.HandshakeConfig
	var pluginMap = map[string]plugin.Plugin{
		"executer": &odmPlugin.ExecuterPlugin{Impl: &ExecuterImpl{}},
	}

	plugin.Serve(&plugin.ServeConfig{
		HandshakeConfig: handshakeConfig,
		Plugins:         pluginMap,
		// We are using standard RPC for simplicity, so no GRPCServer or GRPCProvider needed.
	})
	log.Println("Greeter plugin finished serving.")
	// This line should never be reached under normal circumstances when the plugin serves correctly.
	log.Println("Plugin: Serve returned (this shouldn't happen)")
}
```

Key points in the entrypoint:

- **`odmPlugin.HandshakeConfig`**: This is a crucial configuration used by `go-plugin` to ensure the `odm` CLI tool and your plugin can successfully establish a connection.
- **`pluginMap`**: This map associates a string name (e.g., `"executer"`) with an instance of your `odmplugin.ExecuterPlugin`. The `Impl` field of `ExecuterPlugin` must be set to an instance of your `Executer` implementation (e.g., `&ExecuterImpl{}`).
- **`plugin.Serve`**: This function from `go-plugin` starts the RPC server that allows the `odm` CLI tool to communicate with your plugin.

### Building Your Plugin

To build your plugin, simply use the standard Go build command:

```bash
go build -o my-odm-plugin .
```

This will create an executable file named `my-odm-plugin` (or `my-odm-plugin.exe` on Windows) in your current directory. This executable is your `odm` plugin.

You would then configure the `odm` CLI tool to discover and utilize this executable as a plugin. Consult the `odm` CLI tool's documentation for specifics on how to register and use external plugins.

---

## Contributing

Contributions to improve this library are welcome\! Please feel free to open issues or submit pull requests.

---

## License

This project is licensed under the [MIT License](https://www.google.com/search?q=LICENSE).

---
