# [new task] example

    # This is executable Markdown that's tested on CI.
    set -o errexit -o nounset -o xtrace
    alias ~~~=":<<'~~~sh'";:<<'~~~sh'

## Try it out

Lorem Ipsum

~~~sh
output="$(aspect mytask)"

# Verify that it produces the expected output
echo "${output}" | grep -q "SUCCESS" || {
    echo >&2 "Wanted output containing 'SUCCESS' but got '${output}'"
    exit 1
}
~~~
