# Containers

## Dockerfiles
Same purpose as YAMLs, but these are for building container images instead of running the containers
Each line in the dockerfile starts with a command in all caps known as the instruction
a simple dockerfile might look like this:
```
FROM <base image>

RUN <command to install a dependency>

COPY <source location> <destination location>
# Source location is relative to the build command

ENTRYPOINT <command to run when the container starts>
```

Instructions:
- FROM: always the first instruction in a dockerfile, specifies the base image to use for the container
- RUN: executes a command inside the container
- COPY: copies code from the host system to the container
- ENTRYPOINT: specifies the command that the container runs on startup

## Kubernetes YAML vs Dockerfile
Images are built using the dockerfile, which are then called to be part of pods via the yaml
commands included in the YAML will be applied as an additional layer on top of the image once the container starts running
that could be a new command, a new command with new arguments, or updated arguments for the last command in the dockerfile

| YAML | Dockerfile |
| command | ENTRYPOINT |
| args | CMD |
### ENTRYPOINT vs CMD
Do essentially the same thing; specify the command to run when the container starts up
ENTRYPOINT sets the "what" for the container - the core command or application
- useful when you want to specify an executable, but don't want to restrict the inputs
CMD sets the "how" - default parameters or arguments for that command
- will get overwritten by parameters provided on the command line
CMD can also be used alone to just define the command to run, but it can still get overwritten by the command line
ENTRYPOINT can get overwritten as well, but still


