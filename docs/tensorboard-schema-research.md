# TensorBoard Protobuf Schema Research

## Question
Does the TensorBoard protobuf schema (event.proto, summary.proto) remain compatible with TensorBoard >= v2.15?

## Findings

### Current Schema Status
- **event.proto**: Stable schema in `tensorboard/compat/proto/event.proto`
- **summary.proto**: Stable schema in `tensorboard/compat/proto/summary.proto`
- Both files use `syntax = "proto3"` and package `tensorboard`

### Schema Analysis (as of TensorBoard 2.20.0)

#### event.proto Structure
- `Event` message with core fields:
  - `wall_time` (double) - timestamp
  - `step` (int64) - global step
  - Oneof `what` containing various event types:
    - `file_version` (string) - version identifier
    - `summary` (Summary) - scalar/histogram/image data
    - `graph_def` (bytes) - TensorFlow graph
    - Other fields for session management, metadata

#### summary.proto Structure
- `Summary` message containing repeated `Value` messages
- `Value` message with:
  - `tag` (string) - data identifier
  - `metadata` (SummaryMetadata) - plugin information
  - Oneof `value` containing:
    - `simple_value` (float) - scalar data
    - `image` (Image) - image data
    - `histo` (HistogramProto) - histogram data
    - `audio` (Audio) - audio data
    - `tensor` (TensorProto) - generic tensor data

### Version Compatibility
- **TensorBoard 2.15.2**: No breaking schema changes
- **TensorBoard 2.16.0**: No breaking schema changes  
- **TensorBoard 2.17.0**: No breaking schema changes
- **TensorBoard 2.18.0**: No breaking schema changes
- **TensorBoard 2.19.0**: No breaking schema changes
- **TensorBoard 2.20.0**: No breaking schema changes

### Breaking Changes Analysis
- Only protobuf dependency restrictions changed (avoiding bad versions)
- Core event/summary schema remains backward compatible
- New fields added but not required (protobuf forward compatibility)

## Conclusion
**SAFE TO PROCEED** - The TensorBoard protobuf schema is stable and compatible with TensorBoard >= v2.15.

## Target Version Range
- **Minimum**: TensorBoard 2.15.0
- **Tested**: Up to TensorBoard 2.20.0
- **Expected**: Future 2.x versions (backward compatible)

## Implementation Notes
1. Use `tensorboard/compat/proto/` files (not TensorFlow core)
2. Target protobuf schema from TensorBoard 2.15.0+
3. Schema is forward-compatible due to protobuf design
4. Focus on core `Event` and `Summary` messages for v0.1.0

## References
- [TensorBoard event.proto](https://github.com/tensorflow/tensorboard/blob/master/tensorboard/compat/proto/event.proto)
- [TensorBoard summary.proto](https://github.com/tensorflow/tensorboard/blob/master/tensorboard/compat/proto/summary.proto)
- [TensorBoard Release Notes](https://github.com/tensorflow/tensorboard/releases)
