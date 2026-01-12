> [!IMPORTANT]  
> This repo has been superseded by [expo/claude-code-plugins](https://github.com/expo/claude-code-plugins).


# Expo Agent Skills

This repository contains a collection of AI agent skills for Expo developers.

## Skills

- [Update Expo Skills](./update-expo-skills/): updates the Expo skills installed on your computer.
- [Expo CI/CD Workflows](./expo-cicd-workflows/): write YAML configuration files to define CI/CD workflows that run on EAS.

## Installation

Your system's default version of Node must be version 22.18.0 or newer.

Clone the `stable` branch of this repository and run `./install`. The installation script creates symlinks in `~/.claude/skills` to your clone of this repository.

```sh
git clone --branch stable https://github.com/expo/skills.git && cd skills && ./install
```

> [!NOTE]
> Maintainers of this repository should track the `main` branch instead.

## Updating

To update your Expo skills, ask your AI agent to "update Expo skills." This runs the "Update Expo Skills" skill, which pulls the latest changes and reinstalls the skills.

You can also just run `git pull` and `./install` again.
