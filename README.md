## Project structure
To mimic our repository structure, `py` and `rpkg` are submodules in one project. There is a top level `WORKSPACE` for the whole project/repository.
Under each submodule, there is `BUILD`.

## Bazel command syntax
- `bazel help` for CLI help
- Bazel has its own target syntax: `@workspace_name//package_name:target_name`. If you are at the current workspace, `@workspace_name//` can be omitted.
- Example is `bazel build py:poetry_build`

## Lock file for Python
Bazel does not do dependency management. To generate lock file, you can either use `pip-compile requirements.ini` or `poetry export --format=requirements.txt > requirements_lock.txt`. Since we have poetry, I used the latter. Bazel will download packages from pip using its macro `pip-parse`.

## Sandboxing
When Bazel builds, executes tests, and runs, it does so in a "sandbox:" it is basically a clean room filesystem with only the things you've declared in your BUILD file in it. Basically it is file sandboxing, not process sandboxing. Bazel does this for repeatable builds so that it can build in parallel (https://bazel.build/basics/hermeticity).

## Bazel command
- Make sure you're running gradle commands from project root.
- To build wheel of the python project, run `./gradlew pybuild`.
- To install dependencies of the python project, run `./gradlew pyinstall`.
- `pybuild` and `pyinstall` are independent, it doesn't matter which one to run first.
- To install R packages, run `./gradlew rinstall`.
- To build R packages, run `./gradlew rbuild`.


## Test the wheel file

We already tested `pip install *.whl` in `poc-gradle`. Skipped this step as it is same no matter which build tool we choose.

## Test R-installed pkg

Start a R session and import `jhe`

