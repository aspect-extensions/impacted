# `aspect affected`

This task lets you run only the tests which are "affected" by changes to the sources.

It uses the popular Tinder/bazel-diff utility.
As they document on https://github.com/Tinder/bazel-diff#getting-started:

> simply clone down the repo and then run the example shell script

Instead of an example shell script, this Aspect Extension implements the logic in starlark.
