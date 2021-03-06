---
author: nikmd23
ms.author: nimolnar
ms.service: visual-studio-online
title: Configuring Visual Studio Online
ms.topic: overview
ms.date: 02/25/2020
---

# Configuring Visual Studio Environments

Visual Studio Online's [environments](../overview/what-is-vsonline.md#environments) are fully customizable on a per project basis. This is accomplished by including a `devcontainer.json` file in the project's repository.

Example customizations include:

- Setting which Linux based operating system to use
- Automatically installing various tools, runtimes and frameworks
- Forwarding commonly used ports
- Setting environment variables
- Configuring editor settings and installing preferred extensions

The `devcontainer.json` file can be placed in one of two places in a repository:

1. `{repository-root}/.devcontainer.json`
2. `{repository-root}/.devcontainer/devcontainer.json`

> [!WARNING]
> `devcontainer.json` files are also used to support Visual Studio Code Remote Development, and has [additional properties not covered in this document](https://code.visualstudio.com/docs/remote/containers#_devcontainerjson-reference). These additional properties are safe to add to the file, but will be ignored by VS Online.

A general container will be provided when a `devcontainer.json` file isn't provided, or when the `image` or `dockerFile` properties are left blank. This container has the following tools and SDKs:

- Python
- Node.js
- .NET Core
- PHP
- Java
- sudo
- fish
- zsh
- Powershell
- Azure CLI
- nvm
- nvs

> [!NOTE]
> The versions of Python, Node.js, .NET Core, and PHP in the container are based on the latest [Oryx](https://github.com/Microsoft/Oryx) image.

## Containers requirements

Containers should be able to fullfil requirements of both [VSCode Remote Server](https://code.visualstudio.com/docs/remote/containers#_system-requirements) and [Live Share](/visualstudio/liveshare/reference/linux#install-linux-prerequisites).

## Container provisioning

When a `devcontainer.json` file is included in the project's repository, the process to create the specified container starts. This may take some time depending on the complexity of the Dockerfile. A terminal containing the details of the build process will appear, and can be used to aid in diagnosing creation failures.

## Visual Studio Online configuration reference

The following tables lists the configuration properties supported by VS Online. All properties are optional, unless otherwise noted.

### General properties

| Property | Type | Description |
|----------|------|-------------|
| `extensions` | array | An array of [VS Code Extension Marketplace](https://marketplace.visualstudio.com/vscode) IDs that specify the extensions that should be installed in the environment when it is created. By default, VS Online installs the recommended extensions for the an environment's most prominent language. |
| `settings` | object | Adds [VS Code `settings.json`](https://code.visualstudio.com/docs/getstarted/settings) values into the environment.  |
| `forwardPorts`| integer or array | A port or array of ports that should be automatically forwarded locally when the environment is running. By default, no ports are forwarded. This is the preferred method of port forwarding. |
| `appPort` | integer or array | A port or array of ports that should be automatically forwarded locally when the environment is running. By default, no ports are forwarded. |
| `postCreateCommand` | string or array | A command string or list of command arguments to run after the environment is created. Use `&&` in a string to execute multiple commands. For example, `"yarn install"`, `["yarn", "install"]`, or `"apt-get update && apt-get install -y git"`. It fires after your source code has been cloned, so you can also run shell scripts from your source repo. For example: `bash .devcontainer/install-dev-tools.sh`. By default, `oryx build` is run. To disable [Oryx](https://github.com/microsoft/Oryx) behavior, set the value to empty string (`""`) or empty array (`[]`). |

### Docker properties

| Property | Type | Description |
|----------|------|-------------|
| `image` | string | The name of an image in a public container registry (e.g. [DockerHub](https://hub.docker.com), [Azure Container Registry](https://azure.microsoft.com/services/container-registry/), [GitHub](https://github.com/features/package-registry)) that VS Online should use to create the environment. |
| `dockerFile` | string | The location of a [Dockerfile](https://docs.docker.com/engine/reference/builder/) that VS Online will `docker build` and use the resulting Docker image for the environment. The path is relative to the `devcontainer.json` file. You can find a number of sample Dockerfiles for different configurations in the [vscode-dev-containers repository](https://github.com/Microsoft/vscode-dev-containers/tree/master/containers). |
| `context` | string | Path that the VS Online's `docker build` command should be run from, relative to `devcontainer.json`. For example, a value of `".."` would allow you to reference content in sibling directories. Defaults to `"."`. |
| `runArgs` | string | Arguments to pass to docker when creating the container. Only `-e`, `-u`, `--cap-add=SYS_PTRACE` and `--security-opt seccomp=unconfined` are taken into account. |
| `overrideCommand` | string | Tells VS Code whether it should run `/bin/sh -c "while sleep 1000; do :; done"` when starting the container, instead of the container's default command. This property defaults to true, since the container can shut down if the default command fails. It should be set to false if the default command must run for the container to function correctly. |
| `containerEnv` | object | A set of name-value pairs that sets or overrides environment variables for the container.  |
| `remoteEnv` | object | A set of name-value pairs that sets or overrides environment variables for VS Code (or sub-processes like terminals), but not the whole container. Updates are applied when the environment is suspended and restarted, or after five minutes have passed after disconnecting. |
| `containerUser` | string or array | The user that will be used when creating the container. If the property is undefined, it defaults to either root or the last USER instruction in the Dockerfile. |
| `remoteUser` | string or array | Overrides the user that VS Code runs as in the container (along with sub-processes like terminals, tasks, or debugging). This property defaults to the `containerUser`. |

## Visual Studio Online configuration sample

> [!TIP]
> `devcontainer.json` files support JSON with Comments (jsonc)!

```js
/* Contents of {repository-root}/.devcontainer/devcontainer.json */

{
  // Open port 3000 by default
  "appPort": 3000,

  // Install ESLint and Peacock extensions
  "extensions": [
    "dbaeumer.vscode-eslint",
    "johnpapa.vscode-peacock"
  ],

  // Set remote color for Peacock
  "settings": {
    "peacock.remoteColor": "#0078D7"
  },

  // Run Bash script in .devcontainer directory
  "postCreateCommand": "/bin/bash ./.devcontainer/post-create.sh > ~/post-create.log",
}
```
