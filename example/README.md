# bazel-diff example

    # This is executable Markdown that's tested on CI.
    set -o errexit -o nounset -o xtrace
    alias ~~~=":<<'~~~sh'";:<<'~~~sh'

## Try it out

~~~sh
output="$(aspect impacted)"

# Verify that it produces the expected output
echo "${output}" | grep -q "Impacted Targets" || {
    echo >&2 "Wanted output containing 'Impacted Targets' but got '${output}'"
    exit 1
}
~~~
