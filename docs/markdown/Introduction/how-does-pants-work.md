---
title: "How does Pants work?"
slug: "how-does-pants-work"
hidden: false
createdAt: "2020-07-29T02:58:23.473Z"
updatedAt: "2022-02-10T19:45:06.305Z"
---
The Pants Engine
----------------

Pants is built around the "v2" engine, which is completely new technology, built from the ground up, based on lessons learned from working on the previous, "v1", technology.

The Pants engine is written in [Rust](https://www.rust-lang.org/), for performance. The build rules that it uses are written in typed Python 3, for familiarity and simplicity. 

The engine is designed so that fine-grained invalidation, concurrency, hermeticity, caching, and remote execution happen naturally, without rule authors needing to think about it.

What are the benefits?
----------------------

### Concurrency

The engine can take full advantage of all the cores on your machine because relevant portions are implemented in Rust atop the [Tokio](https://tokio.rs/) framework.

[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/de72295-concurrency.gif",
        "concurrency.gif"
      ],
      "caption": "Pants running multiple linters in parallel."
    }
  ]
}
[/block]

This means, for example, that you can run all of your linters at the same time, and fully utilize your cores to run tests in parallel.

### Caching

The engine caches processes precisely based on their inputs, and sandboxes execution to minimize side-effects and to make builds consistent and repeatable.

[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/603ef44-caching.gif",
        "caching.gif"
      ],
      "caption": "We run both tests, then add a syntax error to one test and rerun; the unmodified test uses the cache and is isolated from the syntax error."
    }
  ]
}
[/block]

### Remote Execution

The engine can delegate work to a remote build cluster so that you are no longer limited by the number of cores on your machine. If you have enough remote workers, you can run your entire test suite in total parallelism.

Remote caching means that your coworkers and your CI can reuse the results of commands you already ran.

### Fine-grained invalidation

Work is broken down into many small units and kept warm in a daemon so that as little work as possible needs to be re-done when files change.

### Hermetic execution

Pants sandboxes all processes that it executes, ensuring that cache keys are always accurate, and builds are always correct.

### Dependency inference

Pants analyzes your code's import statements to determine files' dependencies automatically. Dependency information is required for precise change detection and cache invalidation, but inference means that you don't need to declare dependencies manually (and hermetic execution guarantees that they are always accurate)!

Older build tools like Bazel:

```python BUILD
python_library(
  name="lib"
  deps=[
    "//src/python/project/core",
    "//src/python/project/models:customer",
    "//src/python/project/models:organization",
    "//src/python/project/models:policy",
    "//src/python/project/models:user",
    "//src/python/project/views:dashboard",
    "//src/python/project/util:csrf_util",
    "//src/python/project/util:strutil",
  ],
)

python_tests(
  name="tests",
  deps=[
    ...
  ],
)
```

Pants 2:

```python BUILD
python_sources(name="lib")
python_tests(name="tests")
```

### A powerful plugin system

With the [Pants plugin API](doc:plugins-overview), your custom rules will run with the same concurrency, caching, and remoting semantics as the core rules.

Some example plugins that users have written:

- Cython support
- Building a Docker image, including packages built via `pants package`
- Custom `setup.py` logic to compute the `version` dynamically
- Jupyter support
