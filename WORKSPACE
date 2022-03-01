load("@bazel_tools//tools/build_defs/repo:git.bzl", "new_git_repository", "git_repository")
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

PY_VERSION = '3.8.3'
BUILD_DIR = '/tmp/bazel-python-{0}'.format(PY_VERSION)

# Special logic for building python interpreter with OpenSSL from homebrew.
# See https://devguide.python.org/setup/#macos-and-os-x
_py_configure = """
if [[ "$OSTYPE" == "darwin"* ]]; then
    cd {0} && ./configure --prefix={0}/bazel_install --with-openssl=$(brew --prefix openssl)
else
    cd {0} && ./configure --prefix={0}/bazel_install
fi
""".format(BUILD_DIR)

# Produce deterministic binary by using a fixed build timestamp and
# running `ar` in deterministic mode. See #7
#
# The 'D' modifier is known to be not available on macos. For linux
# distributions, we check for its existence. Note that it should be the default
# on most distributions since binutils is commonly compiled with
# --enable-deterministic-archives. See #9
_ar_flags = """
ar 2>&1 >/dev/null | grep '\\[D\\]'
if [ "$?" -eq "0" ]; then
  cd {0} && echo -n 'rvD' > arflags.f527268b.txt
else
  cd {0} && echo -n 'rv' > arflags.f527268b.txt
fi
""".format(BUILD_DIR)

http_archive(
    name = "python_interpreter",
    urls = [
        "https://www.python.org/ftp/python/{0}/Python-{0}.tar.xz".format(PY_VERSION),
    ],
    sha256 = "dfab5ec723c218082fe3d5d7ae17ecbdebffa9a1aea4d64aa3a2ecdd2e795864",
    strip_prefix = "Python-{0}".format(PY_VERSION),
    patch_cmds = [
        # Create a build directory outside of bazel so we get consistent path in
        # the generated files. See #8
        "mkdir -p {0}".format(BUILD_DIR),
        "cp -r * {0}".format(BUILD_DIR),
        # Build python.
        _py_configure,
        _ar_flags,
        "cd {0} && SOURCE_DATE_EPOCH=0 make -j $(nproc) ARFLAGS=$(cat arflags.f527268b.txt)".format(BUILD_DIR),
        "cd {0} && make install".format(BUILD_DIR),
        # Copy the contents of the build directory back into bazel.
        "rm -rf * && mv {0}/* .".format(BUILD_DIR),
        "ln -s bazel_install/bin/python3 python_bin",
    ],
    build_file_content = """
exports_files(["python_bin"])
filegroup(
    name = "files",
    srcs = glob(["bazel_install/**"], exclude = ["**/* *"]),
    visibility = ["//visibility:public"],
)
""",
)

git_repository(
  name = "pybind11_bazel",
  remote = "https://github.com/pybind/pybind11_bazel.git",
  commit = "72cbbf1fbc830e487e3012862b7b720001b70672",
  shallow_since = "1638580149 -0800",
)

new_git_repository(
  name = "pybind11",
  remote = "https://github.com/pybind/pybind11.git",
  commit = "ffa346860b306c9bbfb341aed9c14c067751feb8",
  shallow_since = "1643841255 -0500",
  build_file = "@pybind11_bazel//:pybind11.BUILD",
)


register_toolchains("//:py_toolchain")

load("@pybind11_bazel//:python_configure.bzl", "python_configure")
python_configure(
  name = "local_config_python",
  python_interpreter_target = "@python_interpreter//:python_bin",
)