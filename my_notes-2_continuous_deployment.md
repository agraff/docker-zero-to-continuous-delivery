Fork the repository: https://github.com/codurance/simple_rest


TeamCity instructions
=====================

Create first build configuration
--------------------------------
Name: 1. Build
Create.

Type of VCS: Git
VCS root name: workstation7
VCS root ID: 
Fetch URL: <HTTPS address from fork in Github> e.g. https://github.com/username/simple_rest.git
Test connection. Save.

Build Steps -> Add build step.
Runner type: Gradle
Gradle tasks: clean test build
Gradle Wrapper: tick (need to use wrapper because build agent does not have Gradle installed)
Save.

General Settings.
Artifact paths: docker/Dockerfile
                build/distributions/simple_rest.tar

This creates artifacts including Dockerfile and libraries for the application (in a tar file).


Create a second build configuration
-----------------------------------
Name: 2. Release
Create.

Dependencies.
Add new snapshot dependency.
Tick "1. Build"
On failed dependency: Cancel build.
Use defaults for everything else.
Save.
Add new artifact dependency.
Depend on: 1. Build.
Get artifacts from: Build from the same chain.
Artifacts rules:  Dockerfile
                  simple_rest.tar
Clean destination paths before downloading artifacts: tick
Save.

Triggers.
Add new trigger.
Finish Build Trigger
Build configuration: 1. Build
Trigger after successful build only: tick
Save.

Build Steps
Add build step.
Runner type: Command line
Step name: Release
Run: Custom script
Custom script:  #!/bin/bash
                set -e
                docker build --tag registry.training.local/workstation-7:%build.number% .
                docker push registry.training.local/workstation-7:%build.number%
Save.
Add build step.
Runner type: Command line
Step name: Generate release version
Run: Custom script
Custom script:  echo %build.number% > release.version
Save.

General settings
Artifact paths: release.version
Save.


Create a third build configuration
-----------------------------------
Name: 3. Deploy
Create.

Dependencies
Add new snapshot dependency
Tick "2. Release"
On failed dependency: Cancel build.
Use defaults for everything else.
Save.

Add new artifact dependency.
Depend on: 2. Release
Get artifacts from: Build from the same chain.
Artifacts rules:  release.version
Clean destination paths before downloading artifacts: tick
Save.

Triggers
Add new trigger.
Finish Build Trigger
Build configuration: 2. Release
Trigger after successful build only: tick
Save.

Build Steps
Add build step.
Runner type: Command line
Run: Custom script
Custom script:  #!/bin/bash
                set -e
                export IMAGE_VERSION=$(cat release.version)
                export DOCKER_HOST=tcp://workstation-7.training.local:2375
                docker rm -f workstation-7 || echo "Info: application was not already running"
                docker run -d -p 80:4567 --name workstation-7



To use Docker Compose
1. Build : General Settings : Artifact paths : Add "docker/docker-compose.yml"
2. Release : General Settings : Artifact paths : Add "docker-compose.yml"
           : Dependencies : Artifact Dependencies : Artifacts Source : Add "docker-compose.yml"
3. Deploy : General Settings : Artifact paths : Add "docker-compose.yml"
          : Dependencies : Artifact Dependencies : Artifacts Source : Add "docker-compose.yml"
          : Build Step : Command Line : change script to the following:
          #!/bin/bash
          set -e
          export IMAGE_VERSION=$(cat release.version)
          export DOCKER_HOST=tcp://workstation-7.training.local:2375
          docker-compose down
          docker-compose up -d

Note: Need to log in to workstation and manually stop and remove any containers and images from previous deploy approach, because docker compose down will not clean up containers created using someother way (not created using docker compose).