---
description: Quickstart guide to start using Konduit-Serving
---

# Quickstart

Konduit Serving is a framework-agnostic model serving solution focused on deploying machine learning pipelines to production. The Python SDK allows data scientists to quickly test machine learning deployment scenarios, bridging the gap between data science teams and DevOps.

Before running these commands, set up Konduit Serving according to the installation instructions on the [Building From Source](building-from-source.md) and [Installation](installation.md) page:

## Configuration Files

Konduit Serving configuration files consist of `serving`, `steps` and `client` components. Save the configuration below as a text file named `hello-world.yaml` in your current directory:

```yaml
serving:
  http_port: 1337
  input_data_format: NUMPY
  output_data_format: NUMPY
steps:
  python_step:
    type: PYTHON
    python_code: |
      first += 2
      second = first
    python_inputs:
      first: NDARRAY
    python_outputs:
      second: NDARRAY
client:
    port: 1337
```

The pages in this section show you how to start and interact with a Konduit Serving instance. For these examples, the Konduit Serving instance and client are on the same machine.

For quick experimentation, check out the quickstart for the command line interface \(CLI\):

To access additional options, you will want to configure Konduit Serving instances with the Python SDK. Start with the Python quickstart:

