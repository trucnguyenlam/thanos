# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](http://keepachangelog.com/en/1.0.0/)
and this project adheres to [Semantic Versioning](http://semver.org/spec/v2.0.0.html).

NOTE: As semantic versioning states all 0.y.z releases can contain breaking changes in API (flags, grpc API, any backward compatibility)

We use *breaking* word for marking changes that are not backward compatible (relates only to v0.y.z releases.)

## Unreleased

## [v0.3.0](https://github.com/improbable-eng/thanos/releases/tag/v0.3.0) - 2019.02.08

### Added

- Support for gzip compressed configuration files before envvar substitution for reloader package.
- `bucket inspect` command for better insights on blocks in object storage.
- Support for [Tencent COS](docs/storage.md#tencent-cos-configuration) object storage.
- Partial Response disable option for StoreAPI and QueryAPI.
- Partial Response disable button on Thanos UI
- We have initial docs for goDoc documentation!
- Flags for Querier and Ruler UIs: `--web.route-prefix`, `--web.external-prefix`, `--web.prefix-header`. Details [here](docs/components/query.md#expose-ui-on-a-sub-path)

### Fixed

- [#649](https://github.com/improbable-eng/thanos/issues/649) - Fixed store label values api to add also external label values.
- [#396](https://github.com/improbable-eng/thanos/issues/396) - Fixed sidecar logic for proxying series that has more than 2^16 samples from Prometheus.
- [#732](https://github.com/improbable-eng/thanos/pull/732) - Fixed S3 authentication sequence. You can see new sequence enumerated [here](https://github.com/improbable-eng/thanos/blob/master/docs/storage.md#aws-s3-configuration)
- [#745](https://github.com/improbable-eng/thanos/pull/745) - Fixed race conditions and edge cases for Thanos Querier fanout logic. 
- [#651](https://github.com/improbable-eng/thanos/issues/651) - Fixed index cache when asked buffer size is bigger than cache max size.

### Changed

- [#529](https://github.com/improbable-eng/thanos/pull/529) Massive improvement for compactor. Downsampling memory consumption was reduce to only store labels and single chunks per each series.
- Qurerier UI: Store page now shows the store APIs per component type.
- Prometheus and TSDB deps are now up to date with ~2.7.0 Prometheus version. Lot's of things has changed. See details [here #704](https://github.com/improbable-eng/thanos/pull/704) Known changes that affects us:    
    - prometheus/prometheus/discovery/file
      - [ENHANCEMENT] Discovery: Improve performance of previously slow updates of changes of targets. #4526
      - [BUGFIX] Wait for service discovery to stop before exiting #4508 ??
    - prometheus/prometheus/promql:
      - **[ENHANCEMENT] Subqueries support. #4831**
      - [BUGFIX] PromQL: Fix a goroutine leak in the lexer/parser. #4858
      - [BUGFIX] Change max/min over_time to handle NaNs properly. #438
      - [BUGFIX] Check label name for `count_values` PromQL function. #4585
      - [BUGFIX] Ensure that vectors and matrices do not contain identical label-sets. #4589
      - [ENHANCEMENT] Optimize PromQL aggregations #4248
      - [BUGFIX] Only add LookbackDelta to vector selectors #4399
      - [BUGFIX] Reduce floating point errors in stddev and related functions #4533
    - prometheus/prometheus/rules:
      - New metrics exposed! (prometheus evaluation!)
      - [ENHANCEMENT] Rules: Error out at load time for invalid templates, rather than at evaluation time. #4537
    - prometheus/tsdb/index: Index reader optimizations.
- Thanos store gateway flag for sync concurrency (`block-sync-concurrency` with `20` default, so no change by default)
- S3 provider:
  - Added `put_user_metadata` option to config.
  - Added `insecure_skip_verify` option to config.
  
### Deprecated
  
- Tests against Prometheus below v2.2.1. This does not mean *lack* of support for those. Only that we don't tests the compatibility anymore. See [#758](https://github.com/improbable-eng/thanos/issues/758) for details.
  
## [v0.2.1](https://github.com/improbable-eng/thanos/releases/tag/v0.2.1) - 2018.12.27

### Added

- Relabel drop for Thanos Ruler to enable replica label drop and alert deduplication on AM side.
- Query: Stores UI page available at `/stores`.

![](./docs/img/query_ui_stores.png)

### Fixed

- Thanos Rule Alertmanager DNS SD bug.
- DNS SD bug when having SRV results with different ports.
- Move handling of HA alertmanagers to be the same as Prometheus.
- Azure iteration implementation flaw.

## [v0.2.0](https://github.com/improbable-eng/thanos/releases/tag/v0.2.0) - 2018.12.10

Next Thanos release adding support to new discovery method, gRPC mTLS and two new object store providers (Swift and Azure).

Note lots of necessary breaking changes in flags that relates to bucket configuration.

### Deprecated

- *breaking*: Removed all bucket specific flags as we moved to config files:
    - --gcs-bucket=\<bucket\>
    - --s3.bucket=\<bucket\>
    - --s3.endpoint=\<api-url\>
    - --s3.access-key=\<key\>
    - --s3.insecure
    - --s3.signature-version2
    - --s3.encrypt-sse
    - --gcs-backup-bucket=\<bucket\>
    - --s3-backup-bucket=\<bucket\>
- *breaking*: Removed support of those environment variables for bucket:
    * S3_BUCKET
    * S3_ENDPOINT
    * S3_ACCESS_KEY
    * S3_INSECURE
    * S3_SIGNATURE_VERSION2
- *breaking*: Removed provider specific bucket metrics e.g `thanos_objstore_gcs_bucket_operations_total` in favor of of generic bucket operation metrics.

### Changed

- *breaking*: Added `thanos_` prefix to memberlist (gossip) metrics. Make sure to update your dashboards and rules.
- S3 provider:
  - Set `"X-Amz-Acl": "bucket-owner-full-control"` metadata for s3 upload operation.

### Added

- Support for heterogeneous secure gRPC on StoreAPI.
- Handling of scalar result in rule node evaluating rules.
- Flag `--objstore.config-file` to reference to the bucket configuration file in yaml format. Detailed information can be found in document [storage](docs/storage.md).
- File service discovery for StoreAPIs:
- In `thanos rule`, static configuration of query nodes via `--query`
- In `thanos rule`, file based discovery of query nodes using `--query.file-sd-config.files`
- In `thanos query`, file based discovery of store nodes using `--store.file-sd-config.files`
- `/-/healthy` endpoint to Querier.
- DNS service discovery to static and file based configurations using the `dns+` and `dnssrv+` prefixes for the respective lookup. Details [here](/docs/service_discovery.md)
- `--cluster.disable` flag to disable gossip functionality completely.
- Hidden flag to configure max compaction level.
- Azure Storage.
- OpenStack Swift support.
- Thanos Ruler `thanos_rule_loaded_rules` metric.
- Option for JSON logger format.

### Fixed

- Issue whereby the Proxy Store could end up in a deadlock if there were more than 9 stores being queried and all returned an error.
- Ruler tracing causing panics.
- GatherIndexStats panics on duplicated chunks check.
- Clean up of old compact blocks on compact restart.
- Sidecar too frequent Prometheus reload.
- `thanos_compactor_retries_total` metric not being registered.

## [v0.1.0](https://github.com/improbable-eng/thanos/releases/tag/v0.1.0) - 2018.09.14

Initial version to have a stable reference before [gossip protocol removal](https://github.com/improbable-eng/thanos/blob/master/docs/proposals/gossip-removal.md).

### Added

- Gossip layer for all components.
- StoreAPI gRPC proto.
- TSDB block upload logic for Sidecar.
- StoreAPI logic for Sidecar.
- Config and rule reloader logic for Sidecar.
- On-the fly result merge and deduplication logic for Querier.
- Custom Thanos UI (based mainly on Prometheus UI) for Querier.
- Optimized object storage fetch logic for Store.
- Index cache and chunk pool for Store for better memory usage.
- Stable support for Google Cloud Storage object storage.
- StoreAPI logic for Querier to support Thanos federation (experimental).
- Support for S3 minio-based AWS object storage (experimental).
- Compaction logic of blocks from multiple sources for Compactor.
- Optional Compaction fixed retention.
- Optional downsampling logic for Compactor (experimental).
- Rule (including alerts) evaluation logic for Ruler.
- Rule UI with hot rules reload.
- StoreAPI logic for Ruler.
- Basic metric orchestration for all components.
- Verify commands with potential fixes (experimental).
- Compact / Downsample offline commands.
- Bucket commands.
- Downsampling support for UI.
- Grafana dashboards for Thanos components.
