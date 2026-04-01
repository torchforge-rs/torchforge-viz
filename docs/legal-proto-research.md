# Legal Research: TensorFlow Protobuf Licensing

## Question
What are the licensing obligations when deriving Rust structs from TensorFlow's Apache-2.0 licensed `.proto` files using `prost-build`?

## Findings

### TensorFlow License
- TensorFlow uses Apache License 2.0
- The `.proto` files in `tensorboard/compat/proto/` are Apache-2.0 licensed
- Apache 2.0 is permissive and allows creating derivative works

### Generated Code Status
- According to Apache 2.0 Section 1 ("Object" form definition): "Object form shall mean any form resulting from mechanical transformation or translation of a Source form, including but not limited to compiled object code, generated documentation, and conversions to other media types."
- Protobuf-generated Rust code qualifies as "Object form" under this definition
- This is explicitly permitted under Apache 2.0

### prost-build Licensing
- `prost` itself is Apache-2.0 licensed
- The prost maintainers recommend checking in generated code rather than building at runtime
- Generated code inherits the source license terms

### Attribution Requirements
Apache 2.0 requires:
1. **Preserve copyright notices** - Include in `proto/README.md`
2. **Preserve license text** - Reference Apache-2.0 license
3. **Include NOTICE file if present** - TensorFlow includes a NOTICE file
4. **State changes** - Document that this is derived from TensorFlow protobuf files

## Conclusion
**SAFE TO PROCEED** - Vendoring TensorFlow `.proto` files and generating Rust structs via `prost-build` creates no additional license obligations beyond proper attribution under Apache 2.0.

## Required Attribution
1. Include TensorFlow copyright notice in `proto/README.md`
2. Reference Apache-2.0 license
3. Document source URLs and commit hashes
4. Add attribution in crate-level documentation
5. **Note**: The crate's own code remains Apache-2.0 licensed (not dual-licensed). Only the vendored `.proto` files require attribution to TensorFlow.

## References
- [Apache License 2.0, Section 1](https://www.apache.org/licenses/LICENSE-2.0)
- [TensorFlow License](https://github.com/tensorflow/tensorflow/blob/master/LICENSE)
- [prost License](https://github.com/tokio-rs/prost/blob/master/LICENSE-APACHE)
