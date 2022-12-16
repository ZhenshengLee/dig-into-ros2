# 按功能

# 按版本

## 1.10

fea

- New built-in Shared Memory Transport
- Add new CMake option: SHM_TRANSPORT_DEFAULT. Shared Memory (SHM) Transport disabled by default.

## 2.0

fix

- Fixed SHM issues (#1644, #1895, #2266)
- Fixed Discovery Server 2.0 issues

## 2.1

fea

- Added methods to get qos defined in XML Profile
- SKIP_DEFAULT_XML environment variable

opt

- Internal refactor for intra-process performance boost
- Discovery server improvements

fix

- Fixed some intra-process related segmentation faults and deadlocks
- Fixed SHM data corruption when using both payload and sub-message protection

## 2.2

fea:

- **Data Sharing delivery** (avoids transport encapsulation for localhost communications)
- Extension APIs allowing zero-copy delivery (both intra-process and inter-process)
- Upgrade to Quality Level 1

opt:<br />fix:

- Fixed warnings building on C++20

## 2.3

fea:

- Unique network flows
- Discovery super-client
- Statistics module API
- New flow controller API
- Update Fast CDR to v1.0.21
- Update Fast DDS Gen to v2.0.2
- Support of googletest using colcon

opt

- **Data-sharing delivery internal refactor**
- Discovery server improvements

fix:

## 2.4

fea

- Conditions and Wait-sets implementation.
- Flow controllers.
- Configure Discovery Server locators using names.
- Support for partitions on DataWriterQoS and DataReaderQoS
- Enable memory protection on DataSharing readers
- Set recommended statistics DataReaderQos to PREALLOCATED_WITH_REALLOC

opt

- **Allow setting custom folder for data-sharing files.**

fix

- Avoid a volatile data-sharing reader to block a writer.
- Correctly give priority to intra-process over data-sharing.
- Correctly disable DataReader on destruction

## 2.5

fea

- Added interfaces for content filter APIs
- ContentFilterTopic filtering at the DataReader side.

fix:

- Discovery Server fixes.
- Fix DataSharing sample validation.
- Enable memory protection on DataSharing readers.
- STRICT_REALTIME fix.

## 2.6

fea

- Update Fast-CDR submodule to v1.0.24
- XML support for Fast DDS CLI
- New exchange format to reduce bandwidth in Static Discovery
- Support for writer side content filtering

opt<br />fix

- Fixed acknowledgement in DataSharing.
- Fixed several deadlocks and data races.

# 接口稳定性

- Methods and attributes have been added on several classes of the DDS-PIM high-level API, so indexes of symbols on dynamic libraries may have changed.
- Methods and attributes have been added on several classes of the RTPS low-level API, so indexes of symbols on dynamic libraries may have changed.
- Old Fast-RTPS high-level API remains ABI compatible.
