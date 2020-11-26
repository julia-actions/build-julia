# build-julia
An action to build Julia from source for a given commit.

## Usage

See [`action.yml`](action.yml).

```yaml
name: Example

on: [push, pull_request, workflow_dispatch]

jobs:
  example:
    runs-on: ubuntu-20.04
    strategy:
      matrix:
        # Even if you are only running one build, you may want to use a strategy matrix,
        # as it allows you to refer to the ref using ${{ matrix.julia-ref }} in your workflow.
        # This makes it easier to edit the version that you want to build without having to update
        # several steps.
        julia-ref: [v1.5.3]

    steps:
      - name: Cache
        id: cache-julia
        uses: actions/cache@v2
        with:
          # If you use a different target dir, you may want to update the cache key to include that info.
          # actions/cache does not provide an option to specify the target dir for restoring the cache,
          # so you may run into issues where the target dir of the build-julia action mismatches the cached directory.
          path: ~/julia

          # Note that this cache key will not work with branches
          # because there could be new commits after the cache has been created.
          key: ${{ runner.os }}-${{ matrix.julia-ref }}

      - name: Build Julia
        # Only rebuild Julia if there is no cache hit.
        if: steps.cache-julia.outputs.cache-hit != 'true'
        uses: julia-actions/build-julia@v1
        with:
          # The value that is passed to git checkout, e.g. a tag, branch or a commit.
          #
          # Default: master
          ref: ${{ matrix.julia-ref }}

          # The value that is passed to git clone. If you want to build Julia from a fork of yours,
          # you can change this value to https://github.com/YOUR_GITHUB_NAME/julia.git.
          #
          # Default: https://github.com/JuliaLang/julia.git
          source-repo: ''

          # The directory that Julia will be installed in. Changing this may be useful when building several
          # versions of Julia within the same environment.
          #
          # Default: $HOME/julia
          target-dir: ''

      - name: Show versioninfo()
        run: ~/julia/julia -e "using InteractiveUtils; versioninfo()"
```
