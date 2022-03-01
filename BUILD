load("@pybind11_bazel//:build_defs.bzl", "pybind_extension")
load("@bazel_tools//tools/python:toolchain.bzl", "py_runtime_pair")

py_runtime(
  name = "python3_runtime",
  files = ["@python_interpreter//:files"],
  interpreter = "@python_interpreter//:python_bin",
  python_version = "PY3",
  visibility = ["//visibility:public"],
)

py_runtime_pair(
  name = "sourced_py_runtime_pair",
  py2_runtime = None,
  py3_runtime = ":python3_runtime",
)

toolchain(
  name = "py_toolchain",
  toolchain = ":sourced_py_runtime_pair",
  toolchain_type = "@bazel_tools//tools/python:toolchain_type",
)

pybind_extension(
  name = "my_pb_mod",  # This name is not actually created!
  srcs = ["my_pb_mod.cc"],
)

py_library(
  name = "my_pb_mod",
  data = [":my_pb_mod.so"],
)

py_binary(
  name = "example_test",
  srcs = ["example_test.py"],
  data = [":my_pb_mod.so"],
  # deps = [
  #     ":my_pb_mod"
  # ],
)
