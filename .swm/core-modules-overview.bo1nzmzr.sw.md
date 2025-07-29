---
title: Core Modules Overview
---
# Core Modules Overview

Core modules form the foundational components of the library, providing essential functionality that enables data validation, serialization, and model configuration. These modules work together to support the seamless creation and management of data models using Python type hints.

## Integration with pydantic-core

A key core module is the integration with `pydantic-core`, a Rust-based library responsible for the core validation logic. This integration ensures that data validation is both efficient and robust, leveraging Rust's performance advantages while maintaining Python's usability.

## The BaseModel Class

The main module defines the `BaseModel` class, which serves as the primary interface for developers to create data models. `BaseModel` encapsulates validation and serialization logic by interfacing with core schemas and validators provided by `pydantic-core`. It also manages model configuration and field metadata, enabling flexible and powerful model definitions.

## Internal Utilities and Helpers

Supporting the main functionality are internal utilities and helper modules that handle namespace management, decorators, field processing, generics, and model construction. These utilities abstract complex internal mechanisms, allowing developers to focus on defining models without worrying about underlying implementation details.

## How Core Modules Enable Model Functionality

Together, these core modules enable the library to provide a smooth experience for defining, validating, and serializing data models. By combining Python type hints with Rust-powered validation, the library achieves both developer-friendly syntax and high performance.

## Using Core Modules in Practice

Developers primarily interact with core modules by extending the `BaseModel` class to define their data models. The class automatically handles validation and serialization by leveraging the Rust-based core. Additionally, internal utilities support advanced features such as generics and custom field handling, enhancing model expressiveness and flexibility.

## Example: BaseModel Usage

The `BaseModel` class demonstrates core module usage by defining class variables for model configuration, field metadata, and validation handlers. It integrates tightly with `pydantic-core` to perform schema validation and serialization, ensuring that models are type-safe and performant.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBcHlkYW50aWMlM0ElM0FTd2ltbS1EZW1v" repo-name="pydantic"><sup>Powered by [Swimm](/)</sup></SwmMeta>
