# Polygraphy Command-line Toolkit User Guide

## Table of Contents

- [Introduction](#introduction)
- [Common Use-Cases](#common-use-cases)
    - [Inspecting A Model](#inspecting-a-model)
    - [Converting A Model To TensorRT](#converting-a-model-to-tensorrt)
    - [Sanitizing An ONNX Model](#sanitizing-an-onnx-model)
    - [Comparing A Model Between Frameworks](#comparing-a-model-between-frameworks)
    - [Using Custom Input Data](#using-custom-input-data)
    - [Modifying Input Shapes In An ONNX Model](#modifying-input-shapes-in-an-onnx-model)
- [Advanced Topics](#advanced-topics)
    - [Deterministic Engine Builds](#deterministic-engine-builds)
    - [Defining A Custom TensorRT Network](#defining-a-custom-tensorrt-network)
    - [Defining A Custom TensorRT Builder Configuration](#defining-a-custom-tensorrt-builder-configuration)
    - [Extracting A Subgraph Of An ONNX Model](#extracting-a-subgraph-of-an-onnx-model)
    - [Debugging Intermittent TensorRT Failures](#debugging-intermittent-tensorrt-failures)
    - [Reducing Failing ONNX Models](#reducing-failing-onnx-models)
- [Examples](#examples)
- [Adding New Tools](#adding-new-tools)


## Introduction

The Polygraphy command-line toolkit includes several tools covering a wide variety of prototyping
and debugging use-cases. This guide provides a broad overview of the capabilities included.

For more information about a specific tool, see the README in the corresponding directory here.

Note that some of the tools included are still experimental. Any tool labeled `[EXPERIMENTAL]`
may be subject to backwards-incompatible changes, or even complete removal at any point in time.

All the tools provided by Polygraphy can be invoked using the polygraphy binary: [`bin/polygraphy`](../../bin/polygraphy).
For usage information on a specific tool, you can see the help output with: `polygraphy <subtool> -h`


## Common Use-Cases


### Inspecting A Model

It is often useful to inspect various aspects of a model, such as layer names, attributes,
or overall structure. The `inspect model` subtool aims to provide this functionality in a way
that is conducive to programmatic analysis; that is, it provides a human-readable text representation
of the model (and it's `grep`-friendly!).

For more details, refer to the examples, which, among other things, show how to inspect:
- [TensorRT Networks](../../examples/cli/inspect/01_inspecting_a_tensorrt_network/)
- [TensorRT Engines](../../examples/cli/inspect/02_inspecting_a_tensorrt_engine/)
- [ONNX models](../../examples/cli/inspect/03_inspecting_an_onnx_model/)

You can find the complete listing of `inspect` examples [here](../../examples/cli/inspect).


### Converting A Model To TensorRT

The `convert` tool can be used to convert various types of models to other formats;
for example, converting ONNX models to TensorRT.

For more information, refer to the examples, which, among other things, show how to:
- [Convert models with dynamic shapes to TensorRT](../../examples/cli/convert/03_dynamic_shapes_in_tensorrt/)
- [Run TensorRT INT8 calibration during conversion](../../examples/cli/convert/01_int8_calibration_in_tensorrt)

You can find the complete listing of `convert` examples [here](../../examples/cli/convert/).


### Sanitizing An ONNX Model

Sometimes, TensorRT may be unable to import an ONNX model. In these cases, it often helps to
sanitize the ONNX model, removing excess nodes and folding constants. The `surgeon sanitize`
subtool provides this capability using ONNX-GraphSurgeon and ONNX-Runtime.

For more details, refer to [`surgeon` example 02](../../examples/cli/surgeon/02_folding_constants/).

You can find the complete listing of `surgeon` examples [here](../../examples/cli/surgeon/).


### Comparing A Model Between Frameworks

The `run` tool can run inference and compare outputs generated by one or more frameworks.
This can be used to check the accuracy of a framework for a particular model and debug
when the accuracy is poor.

Moreover, it can be used to generate Python scripts that do the same.

For more information, refer to the examples, which, among other things, show how to:
- [Compare outputs between frameworks](../../examples/cli/run/01_comparing_frameworks/)
- [Save and load outputs to compare across runs](../../examples/cli/run/02_comparing_across_runs)
- [Generate comparison scripts](../../examples/cli/run/03_generating_a_comparison_script/)


You can find the complete listing of `run` examples [here](../../examples/cli/run/).


### Using Custom Input Data

For any tools that use inference input data, such as `run` or `convert`, Polygraphy
provides 2 ways to supply custom input data:

1. `--load-input-data`, which takes a path to a JSON file containing a `List[Dict[str, np.ndarray]]`.
    This will cause Polygraphy to load the entire object into memory.
    *NOTE: This may be impractical or impossible if the data is very large.*

2. `--data-loader-script`, which takes a path to a Python script that defines a `load_data` function
    that returns a data loader. The data loader can be any iterable or generator that yields
    `Dict[str, np.ndarray]`. By using a generator, we can avoid loading all the data
    at once, and instead limit it to just a single input at a time.

    *TIP: If you have an existing script that already defines such a function, you do **not** need to create*
        *a separate script just for the sake of `--data-loader-script`. You can simply use the existing script*
        *and optionally use the `--data-loader-func-name` argument to specify the name of the function if it's not `load_data`*

For more information, refer to [`run` example 05](../../examples/cli/run/05_comparing_with_custom_data/).


### Modifying Input Shapes In An ONNX Model

The best way to modify input shapes in an ONNX model is to re-export the model with
the desired input shapes. When this is not possible, you can use the `surgeon sanitize`
tool to do this.

For more information, refer to [`surgeon` example 03](../../examples/cli/surgeon/03_modifying_input_shapes/).


## Advanced Topics

### Deterministic Engine Builds

Because of the way TensorRT works, the engine building process is non-deterministic.
Even when using the same model with the same builder configuration, engines can vary slightly.

To make engine building reproducible, all Polygraphy CLI tools that involve building TensorRT engines
accept `--save-tactics` and `--load-tactics` options, which allow you to save the tactics selected
for an engine and reload them during subsequent builds.

For more information, refer to [`convert` example 02](../../examples/cli/convert/02_deterministic_engine_builds_in_tensorrt/).


### Defining A Custom TensorRT Network

Many of the command-line tools involve creating TensorRT networks. Most of the time, networks
are created by parsing a model from a framework (generally in ONNX format). However, it
is also possible to define the TensorRT network manually using a Python script.

This is useful if you want to modify the network in some way using the TensorRT Python
API; for example, setting layer precisions, or per-layer device preferences.

For more information, refer to [`run` example 04](../../examples/cli/run/04_defining_a_tensorrt_network_or_config_manually).


### Defining A Custom TensorRT Builder Configuration

Similar to defining custom TensorRT networks, it is possible to provide custom
TensorRT builder configurations on the command-line using a Python script.

This is useful when Polygraphy does not support certain TensorRT builder configuration options.

For more information, refer to [`run` example 04](../../examples/cli/run/04_defining_a_tensorrt_network_or_config_manually).


### Extracting A Subgraph Of An ONNX Model

When debugging ONNX models, it can be useful to extract subgraphs to look at them in isolation.
The `surgeon extract` tool allows you to do so.

For more information, refer to [`surgeon` example 01](../../examples/cli/surgeon/01_isolating_subgraphs/).


### Debugging Intermittent TensorRT Failures

Since the TensorRT optimizer is inherently non-deterministic, rebuilding an engine may
result in different tactics being selected. If a particular tactic is faulty, this may manifest
as an intermittent failure. The `debug build` tools helps find faulty tactics in such cases
and reproduce failures reliably.

For more information, refer to [`debug` example 01](../../examples/cli/debug/01_debugging_flaky_trt_tactics/).


### Reducing Failing ONNX Models

When investigating bugs involving ONNX models, it can be useful to reduce the model to a minimum
faliing subgraph. This helps us pinpoint the issue and makes further debugging much easier.
The `debug reduce` tools helps us automate this process.

For more information, refer to [`debug` example 02](../../examples/cli/debug/02_reducing_failing_onnx_models/).


## Examples

For examples, see the corresponding subdirectory in [examples/cli](../../examples/cli)


## Adding New Tools

You can add a new tool by adding a new file in this directory, creating a
class that extends the [`Tool` base class](./base/tool.py), and adding
the new tool to the [registry](./registry.py).

For details on developing tools, see [this example](../../examples/dev/01_writing_cli_tools/).