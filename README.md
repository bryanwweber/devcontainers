# devcontainers

This repository contains specifications for VS Code Remote Development extension [devcontainers](https://code.visualstudio.com/docs/remote/containers). Please see the documentation from Microsoft regarding the use of these files.

Briefly, you have 2 options:

1. Copy the appropriate `.devcontainer` folder into the root of your project's repository
2. Create a global folder for your devcontainers to live in, and configure the `remote.containers.repositoryConfigurationPaths` User setting to point to that folder. For instance, on my computer, the setting is

   ```json
   "remote.containers.repositoryConfigurationPaths": [
     "/Users/bryan/GitHub/devcontainers"
   ]
   ```
