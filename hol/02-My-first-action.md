# 🔨 Hands-on: My first Action

In this hands-on lab you will learn how to create a docker action, pass in parameters and return values to your workflow. And you will learn how to test the action locally with a CI build.

This hands on lab consists of the following steps:
- [Creating the action](#creating-the-action)
- [Testing the action](#testing-the-action)
- Optional: [Release and use the action](#optional-release-and-use-the-action)


## Creating the action

1. Create a new file [`hello-world-docker-action/action.yml`](/../../new/main?filename=hello-world-docker-action%2Faction.yml):
<img width="400" alt="image" src="https://user-images.githubusercontent.com/5276337/174234628-14f58066-3188-42a6-9204-99c577558c08.png">

2. Add content to the `action.yml` file (see the [template](https://github.com/actions/hello-world-docker-action) and
  [help](https://github.com/actions/hello-world-docker-action)). Have the action run a `Dockerfile` and pass
  in the parameter `who-to-greet` with the default value `world`. Also give back an output that returns the current time.


<details>
  <summary>Solution</summary>

```YAML
name: 'Hello World Docker Action'
description: 'Say hello to a user or the world.'
inputs:
  who-to-greet:
    description: 'Who to greet'
    required: true
    default: 'world'
outputs:
  time:
    description: 'The time we said hello.'
runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.who-to-greet }}
```

</details>

3. Commit the file (`[skip ci]` to not run a build, yet).
4. Inside the `hello-world-docker-action` folder create the [`Dockerfile`](/../../new/main?filename=hello-world-docker-action%2FDockerfile). The container inherits `FROM alpine:3.22` and should copy and execute a file `entrypoint.sh`. Remember to make the script executable with `RUN chmod +x entrypoint.sh`

<details>
  <summary>Solution</summary>

```dockerfile
FROM alpine:3.22

# Set the working directory inside the container
WORKDIR /usr/src

# Copy any source file(s) required for the action
COPY entrypoint.sh .
RUN chmod +x entrypoint.sh

# Configure the container to be run as an executable
ENTRYPOINT ["/usr/src/entrypoint.sh"]
```

</details>

5. Commit the file (`[skip ci]` to skip running a build, for now).
6. Create the file [`entrypoint.sh`](/../../new/main?filename=hello-world-docker-action%2Fentrypoint.sh) in the folder. It is a simple bash script that writes a message to the console and sets the output parameter.

<details>
  <summary>Solution</summary>

```bash
#!/bin/sh -l

echo "hello $1"

echo "time=$(date)" >> $GITHUB_OUTPUT
```

</details>

7. Commit the file (`[skip ci]` to not run a build, yet).

## Testing the action

1. To test the action we create a new workflow file [`.github/workflows/hello-world-docker-ci.yml`](/../../new/main?filename=.github%2Fworkflows%2Fhello-world-docker-ci.yml&workflow_template=blank).
2. The workflow should run on every push if the action has changed. Also add a manual trigger to start the build manually.
   Checkout the repository to reference the action locally and without a git reference.

<details>
  <summary>Solution</summary>

```YAML
name: CI Build for Docker Action
on:
  push:
    branches: [ main ]
    paths: [ hello-world-docker-action/** ]
  workflow_dispatch:

jobs:
  test-action:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3.3.0

      - name: Run my own container action
        id: hello-action
        uses: ./hello-world-docker-action
        with:
          who-to-greet: '@wulfland'

      - name: Output time set in the container
        run: echo "The time was ${{ steps.hello-action.outputs.time }} when the action said hello"

```

</details>

3. Run the workflow and see how the parameters are passed into the action and back to the workflow.

<img width="600" alt="image" src="https://user-images.githubusercontent.com/5276337/174239255-262a8014-4b66-40df-aa17-6f043f948342.png">

## Summary

In this hands-on lab you've learned how to create a docker action, pass in parameters, return values to your workflow, and to test the action locally with a CI build.

You can continue now with [Staged deployments](03-Staged-deployments.md).
