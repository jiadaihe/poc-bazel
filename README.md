## Project structure
To mimic our repository structure, the poc is structured by language name -> project name -> module name. There is a top level `WORKSPACE` for the whole project/repository.
Under each submodule, there is `BUILD`.

## Bazel command syntax
- `bazel help` for CLI help
- Bazel has its own target syntax: `@workspace_name//package_name:target_name`. If you are at the current workspace, `@workspace_name//` can be omitted.
- Example is `bazel build py:minimal_with_py_package`

## Lock file for Python
Bazel does not do dependency management. To generate lock file, you can either use `pip-compile requirements.ini` or `poetry export --format=requirements.txt > requirements_lock.txt`. Since we have poetry, I used the latter. Bazel will download packages from pip using its macro `pip-parse` (check WORKSPACE.bazel for details).

## Sandboxing
When Bazel builds, executes tests, and runs, it does so in a "sandbox:" it is basically a clean room filesystem with only the things you've declared in your BUILD file in it. Basically it is file sandboxing, not process sandboxing. Bazel does this for repeatable builds so that it can build in parallel (https://bazel.build/basics/hermeticity). For genrule, it is complicated to figure out the sandbox, so I disabled it. But it is built in sandbox when using bazel's native rules. The complexity that sandbox can add is, for example, copy pkg to libPaths as another user, so it doesn't have permission to write.

## Two ways to build py wheel
1. `bazel build py:minimal_with_py_package`. This will put generated dist inside the sandbox `bazel-bin/py/jhe-0.0.1-py3-none-any.whl`. You can verify by simply unpacking it `wheel unpack bazel-bin/py/jhe-0.0.1-py3-none-any.whl`.
2. `bazel build py:pybuild`. This will put generated dist under `py`. After zipping wheel, the command will fail because `declared output 'py/dist/jhe-0.1.0-py3-none-any.whl' was not created by genrule`. I don't fully understand why. Will need more time to figure out if we decide to go this way.

## R install
```bazel build rpkg:rbuild```
It has same problem as py:pybuild. The build command fails but the R pkg is successfully installed. Need more time to figure out if we use `genrule`. The R pkg `jhe` is installed: 
```
jiadaihe@Jiadais-MacBook-Air poc-bazel % ls /Library/Frameworks/R.framework/Versions/4.3-arm64/Resources/library
KernSmooth      cli             foreign         lattice         parallel        stats           translations
MASS            cluster         generics        lifecycle       pillar          stats4          utf8
Matrix          codetools       glue            magrittr        pkgconfig       survival        utils
R6              compiler        grDevices       methods         rlang           tcltk           vctrs
base            datasets        graphics        mgcv            rpart           tibble          withr
boot            dplyr           grid            nlme            spatial         tidyselect
class           fansi           jhe             nnet            splines         tools
```

## Test the wheel file

We already tested `pip install *.whl` in `poc-gradle`. Skipped this step as it is same no matter which build tool we choose.

## Test R-installed pkg

Start a R session and import `jhe`

