"""Calculate affected targets from the BASE commit

Based on
https://github.com/Tinder/bazel-diff/blob/master/bazel-diff-example.sh
"""
def exec(command):
    return command.stdout("piped").spawn().wait_with_output().stdout.strip()


def impl(ctx: task_context):
    out = ctx.std.io.stdout
    out.write("Building bazel-diff executable...\n")
    build = ctx.build(
        ctx.args.bazel_diff_cli or "//tools:bazel-diff",
        events = True,
        bazel_flags = ["--build_runfile_links"],
    )
    (bazel_diff_command, runfiles) = (None, None)
    for event in build.events():
        if event.type == "named_set_of_files":
            for file in event.payload.files:
                if len(file.path_prefix) == 0 or file.file.uri.endswith(".jar"):
                    continue
                runfiles = file.file.uri.removeprefix("file://") + ".runfiles"
                bazel_diff_command = file.file.uri.removeprefix("file://")
    if not bazel_diff_command:
        ctx.std.io.stderr.write("Error finding bazel-diff executable")
        return 1

    bazel_diff = lambda args: ctx.std.process.command(bazel_diff_command) \
        .current_dir(runfiles) \
        .env("RUNFILES_DIR", runfiles) \
        .env("RUNFILES_MANIFEST_FILE", runfiles + "_manifest") \
        .env("BUILD_WORKSPACE_DIRECTORY", ctx.std.env.current_dir()) \
        .env("BUILD_WORKING_DIRECTORY", ctx.std.env.current_dir()) \
        .args(args) \
        .spawn() \
        .wait()

    out.write("Checking out merge base...\n")
    current_revision = exec(ctx.std.process.command("git").args(["rev-parse", "HEAD"]))
    merge_base = exec(ctx.std.process.command("git").args(["merge-base", "HEAD", "origin/main"]))
    checkout = ctx.std.process.command("git").args(["checkout", merge_base, "--quiet"]).spawn().wait()
    if not checkout.success:
        ctx.std.io.err.write("Error checking out merge base: %s" % checkout.stderr)
        return 1

    out.write("Generating Hashes for Base Revision %s\n" % merge_base)
    starting_hashes_json=ctx.std.env.temp_dir() + "/starting_hashes.json"
    final_hashes_json=ctx.std.env.temp_dir() + "/final_hashes.json"
    impacted_targets_path=ctx.std.env.temp_dir() + "/impacted_targets.txt"
    generate_hashes = bazel_diff([
        "generate-hashes",
        "-w", ctx.std.env.current_dir(),
        "-b", "/Users/alexeagle/Downloads/bazel-8.4.1-darwin-arm64",
        starting_hashes_json
    ])
    if not generate_hashes.success:
        ctx.std.io.stderr.write("Error generating hashes: %s" % generate_hashes.code)
        return 1
    
    out.write("Restoring current revision...\n")
    restore = ctx.std.process.command("git").args(["checkout", current_revision, "--quiet"]).spawn().wait()
    if not restore.success:
        ctx.std.io.err.write("Error checking out original commit: %s" % checkout.stderr)
        return 1

    generate_hashes = bazel_diff([
        "generate-hashes", 
        "-w", ctx.std.env.current_dir(),
        "-b", "/Users/alexeagle/Downloads/bazel-8.4.1-darwin-arm64", final_hashes_json
    ])
    if not generate_hashes.success:
        ctx.std.io.stderr.write("Error generating hashes: %s" % generate_hashes.code)
        return 1

    ctx.std.io.stdout.write("Diffing Hashes %s -> %s\n" % (starting_hashes_json, final_hashes_json))
    get_impacted_targets = bazel_diff([
        "get-impacted-targets", 
        "-sh", starting_hashes_json, 
        "-fh", final_hashes_json, 
        "-o", impacted_targets_path
    ])
    if not get_impacted_targets.success:
        ctx.std.io.stderr.write("Error getting impacted targets: %s" % get_impacted_targets.code)
        return 1
    
    ctx.std.io.stdout.write("Impacted Targets for Revision %s: %s" % (current_revision, ctx.std.fs.read_to_string(impacted_targets_path)))
    return 0

affected = task(
    implementation = impl,
    args = {
        "bazel_diff_cli": args.string(),
    }
)
