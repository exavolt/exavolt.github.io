+++
title = 'Containerized development environment'
date = 2022-08-21T14:49:25Z
draft = true
+++

A few days ago I was asked by a team member about which Go version to install to work on the project. Then it came to my mind that I no longer care about setting up my development environments. My development environments are disposable and recreate-able within seconds, or minutes, well, depending on the internet connection.

A few years back, when I was working on various projects, I felt that maintaining various versions of various tech stacks is a real pain. And after embracing Docker for containerized deployment, I wondered if we can have something similar for our development environments.

Imagine something like this: we can create the definition of development environment for every project. Through this specification, we can specify the tools and versions required by that project, which might be different from one to another. And when I open a project, something will set up its environment isolated from other projects and the host system.

It will be very helpful when moving between computers, and for sharing it with developers working on the project. Everyone doesn’t need to care about setting up the environment.

Sure, environment managers provided by various languages, like `pyenv` `nvm` `gvm`, do help. This is what in my mind: 

- Language agnostic with possibility of combining various languages and tools
- Complete environment rather than just around a language
- Machine-readable environment specification so that I don't need to specify it in the README and so that people don't need to keep their 

Fortunately, at least for me, that technology already existed from a few years back, in form of something called VSCode Dev Container. It’s working exactly as described above.

[VSCode Dev Container](https://code.visualstudio.com/docs/devcontainers/containers) utilizes Docker to create a containerized / isolated environment for every project. We define the environment we want as a Dockerfile and through a YAML file.

## Advantages

- Perfectly replicable development environment, almost — making them easy to share
- Isolated from host and from other projects’ environment — not affecting each-other by default
- Self-contained including tools and VSCode extensions
    - Tools’ version are pinned
    - Extensions are nicely set up by default on official base containers
- Destroying an environment is easy, switching between runtime versions, including to experimental versions, is easy
- Great for experimenting with development with various techs
- Reduce cognitive loads and setting up hassles — the definition is the documentation
- Old projects 

## Disadvantages

- Requires Docker, VSCode and [Dev Container extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers). Might now supported by other IDEs.
- Disk usage as there will be copies of compilers, runtimes and other tools across environments. But destroying unused containers is easy, rebuilding an environment is easy, rebuilding an environment is relatively quick.
- Memory usage especially on machines with limited RAM
- Sharing resources with the host machine or other virtualized / simulated environment might require extra steps. But the upside, we will notice well when resource sharing is required

When I want to experiment on new techs, I simply create a folder and then create a container for it. I’d choose based on the base technology: Go, NodeJS, Rust or others.

I’ve used this kind of environment set up for experimenting with various techs including Go, NodeJS, Rust, Python, Solidity
