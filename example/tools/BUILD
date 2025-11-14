load("@rules_java//java:defs.bzl", "java_binary")

package(default_visibility = ["//visibility:public"])
java_binary(
    name = "bazel-diff",
    main_class = "com.bazel_diff.Main",
    runtime_deps = ["@bazel-diff//jar"],
)
