---
title: Portworx Metrics for monitoring
keywords: portworx, learning, lightboard
description: See reference tables for all Portworx metrics available for monitoring applications.
weight: 3
noicon: true
---

## backup_stats stats
| Name | Description |
| :--- | :--- |
| px_backup_stats_status | Status for this backup (0=InProgress,1=Done) |
| px_backup_stats_size | Size in bytes for this backup |
| px_backup_stats_duration_seconds | Duration in seconds for this backup |

## cluster stats
| Name | Description |
| :--- | :--- |
| px_cluster_cpu_percent | Percentage of CPU Used |
| px_cluster_memory_utilized_percent | Percentage of memory utilization |
| px_cluster_disk_total_bytes | Total storage space in bytes for this node |
| px_cluster_disk_available_bytes | Available storage space in bytes for this node |
| px_cluster_disk_utilized_bytes | Utilized storage space in bytes for this node |
| px_cluster_pendingio | Number of read and write operations currently in progress for this node |

## cluster_status stats
| Name | Description |
| :--- | :--- |
| px_cluster_status_cluster_size | Node count for your portworx cluster. **Deprecated**, use cluster_size. |
| px_cluster_status_size | Node count for your portworx cluster |
| px_cluster_status_cluster_quorum | Indicates if the cluster is in quorum. **Deprecated**, use cluster_quorum. |
| px_cluster_status_quorum | Indicates if the cluster is in quorum |
| px_cluster_status_nodes_online | Number of online nodes in the cluster (includes storage and storageless) |
| px_cluster_status_nodes_offline | Number of offline nodes in the cluster (includes storage and storageless) |
| px_cluster_status_nodes_storage_down | Number of storage nodes where the storage that is full or down |
| px_cluster_status_storage_nodes_online | Number of nodes with storage that are marked online |
| px_cluster_status_storage_nodes_offline | Number of nodes with storage that are marked offline |
| px_cluster_status_storage_nodes_decommissioned | Number of nodes with storage that are marked as decommissioned |

## disk_stats stats
| Name | Description |
| :--- | :--- |
| px_disk_stats_used_bytes | Total storage in bytes for this disk |
| px_disk_stats_interval_seconds | interval_seconds |
| px_disk_stats_io_seconds | Time spent doing IO in seconds for this disk |
| px_disk_stats_progress_io | IO's currently in progress for this disk |
| px_disk_stats_disk_read_bytes | Total bytes read for this disk. **Deprecated**, use disk_stats_read_bytes. |
| px_disk_stats_read_bytes | Total bytes read for this disk |
| px_disk_stats_write_bytes_seconds | Total written bytes for this disk. **Deprecated**, use disk_stats_written_bytes. |
| px_disk_stats_written_bytes | Total written bytes for this disk |
| px_disk_stats_read_seconds | Total time spend reading in seconds for this disk |
| px_disk_stats_write_seconds | Total time spend writing in seconds for this disk |
| px_disk_stats_read_latency_seconds | Average time spent per read operation in seconds for this disk |
| px_disk_stats_write_latency | Average time spent per write operation in seconds for this disk. **Deprecated**, use disk_stats_write_latency_seconds. |
| px_disk_stats_write_latency_seconds | Average time spent per write operation in seconds |
| px_disk_stats_disk_num_reads | Total number of read operations completed successfully for this disk. **Deprecated**, use disk_stats_num_reads. |
| px_disk_stats_disk_num_writes | Total number of write operations completed successfully for this disk. **Deprecated**, use disk_stats_num_writes. |
| px_disk_stats_num_reads | Total number of read operations completed successfully for this disk |
| px_disk_stats_num_writes | Total number of write operations completed successfully for this disk |
| px_disk_stats_num_reads_total | Total number of read operations completed successfully for this disk |
| px_disk_stats_num_writes_total | Total number of write operations completed successfully for this disk |
| px_disk_stats_written_bytes_total | Total bytes written for this disk |
| px_disk_stats_read_bytes_total | Total bytes read for this disk |
| px_disk_stats_read_seconds_total | Total time spend reading in seconds for this disk |
| px_disk_stats_write_seconds_total | Total time spend writing in seconds for this disk |

## network_io stats
| Name | Description |
| :--- | :--- |
| px_network_io_bytessent | Number of bytes sent during this interval |
| px_network_io_received_bytes | Number of bytes received during this interval |
| px_network_io_sent_bytes_total | Total number of bytes sent |
| px_network_io_received_bytes_total | Total number of bytes received |

## node_stats stats
| Name | Description |
| :--- | :--- |
| px_node_stats_used_mem | Used memory in bytes |
| px_node_stats_cpu_usage | Percent of CPU consumption |

## node_status stats
| Name | Description |
| :--- | :--- |
| px_node_status_node_status | Status of this node (https://libopenstorage.github.io/w/master.generated-api.html#status) |
| px_node_status_license_expiry | Number of days until License (or License lease) expires (<0 means Expired) |

## none_stats stats
| Name | Description |
| :--- | :--- |
| px_none_stats_free_mem | Available memory in bytes |
| px_none_stats_total_mem | Total memory in bytes |

## pool_stats stats
| Name | Description |
| :--- | :--- |
| px_pool_stats_pool_written_bytes | Bytes written since last interval for this pool. **Deprecated**, use pool_stats_written_bytes. |
| px_pool_stats_pool_write_latency_seconds | Average time spent per write operation for this pool. **Deprecated**, use pool_stats_write_latency_seconds. |
| px_pool_stats_pool_writethroughput | Average number of bytes written per second for this pool. **Deprecated**, use pool_stats_writethroughput. |
| px_pool_stats_pool_flushed_bytes | Number of flushed bytes since last interval for this pool. **Deprecated**, use pool_stats_flushed_bytes. |
| px_pool_stats_pool_num_flushes | Number of flush(sync) operations since last interval for this pool. **Deprecated**, use pool_stats_num_flushes. |
| px_pool_stats_pool_flushms | Latency for flush for this pool. **Deprecated**, use pool_stats_flushms. |
| px_pool_stats_pool_provisioned_bytes | Provisioned storage space in bytes for this pool. **Deprecated**, use pool_stats_provisioned_bytes. |
| px_pool_stats_pool_status | Status of this Pool (0=Offline,1=Online). **Deprecated**, use pool_stats_status. |
| px_pool_stats_written_bytes | Bytes written since last interval for this pool |
| px_pool_stats_write_latency_seconds | Average time spent per write operation for this pool |
| px_pool_stats_writethroughput | Average number of bytes written per second for this pool |
| px_pool_stats_flushed_bytes | Number of flushed bytes since last interval for this pool |
| px_pool_stats_num_flushes | Number of flush(sync) operations since last interval for this pool |
| px_pool_stats_flushms | Latency for flush for this pool |
| px_pool_stats_provisioned_bytes | Provisioned storage space in bytes for this pool |
| px_pool_stats_status | Status of this Pool (0=Offline,1=Online) |
| px_pool_stats_available_bytes | Available storage space in bytes for this pool |
| px_pool_stats_total_bytes | Total storage space in bytes for this pool |
| px_pool_stats_written_bytes_total | Total bytes written for this pool |
| px_pool_stats_flushed_bytes_total | Total number of flushed bytes |
| px_pool_stats_num_flushes_total | Total number of flush(sync) operations |
| px_pool_stats_flushms_total | Total time spent in flush |

## proc_stats stats
| Name | Description |
| :--- | :--- |
| px_proc_stats_virt | Virtual memory in bytes |
| px_proc_stats_res | Resident set size memory in bytes |
| px_proc_stats_cputime | Amount of time that this process has been scheduled in user and kernel mode measured in clock ticks |

## px_cache stats
| Name | Description |
| :--- | :--- |
| px_px_cache_status | Cache enabled (0=No,1=Yes) |
| px_px_cache_total_blocks | Number of total blocks in the cache |
| px_px_cache_used_blocks | Number of used blocks in the cache |
| px_px_cache_dirty_blocks | Number of dirty blocks in the cache |
| px_px_cache_read_hits | Number of read hits for the cache |
| px_px_cache_read_miss | Number of read misses for the cache |
| px_px_cache_write_hits | Number of write hits for the cache |
| px_px_cache_write_miss | Number of write misses for the cache |
| px_px_cache_block_size | Block size for the cache |
| px_px_cache_mode | Mode of the cache |
| px_px_cache_migrate_promote | Number of blocks promoted to the cache |
| px_px_cache_migrate_demote | Number of block demoted from the cache |

## volume stats
| Name | Description |
| :--- | :--- |
| px_volume_usage_bytes | Used storage space in bytes for this volume |
| px_volume_capacity_bytes | Configured size in bytes for this volume |
| px_volume_halevel | Configured HA level for this volume |
| px_volume_currhalevel | Current HA level for this volume |
| px_volume_iopriority | Configured IO priority for this volume |
| px_volume_attached | Attached state for this volume (0=detached,1=attached) |
| px_volume_status | Status for this volume (https://libopenstorage.github.io/w/master.generated-api.html#volumestatus) |
| px_volume_fs_usage_bytes | Used storage space in bytes as reported by the filesystem for this volume |
| px_volume_fs_capacity_bytes | Total storage space in bytes as reported by the filesystem for this volume |
| px_volume_vol_read_bytes | Number of successfully read bytes during this interval for this volume. **Deprecated**, use volume_read_bytes. |
| px_volume_vol_written_bytes | Number of successfully written bytes during this interval for this volume. **Deprecated**, use volume_read_bytes. |
| px_volume_vol_reads | Number of successfully completed read operations during this interval for this volume. **Deprecated**, use volume_read_bytes. |
| px_volume_vol_writes | Number of successfully completed write operations during this interval for this volume. **Deprecated**, use volume_read_bytes. |
| px_volume_read_bytes | Number of successfully read bytes during this interval for this volume |
| px_volume_written_bytes | Number of successfully written bytes during this interval for this volume |
| px_volume_reads | Number of successfully completed read operations during this interval for this volume |
| px_volume_writes | Number of successfully completed write operations during this interval for this volume |
| px_volume_read_bytes_total | Total number of successfully read bytes for this volume |
| px_volume_written_bytes_total | Total number of successfully written bytes for this volume |
| px_volume_reads_total | Total number of successfully completed read operations for this volume |
| px_volume_writes_total | Total number of successfully completed write operations for this volume |
| px_volume_iops | Number of successful completed I/O operations per second during this interval for this volume |
| px_volume_depth_io | Number of I/O operations currently in progress for this volume |
| px_volume_readthroughput | Number of bytes read per second during this interval for this volume |
| px_volume_writethroughput | Number of bytes written per second during this interval for this volume |
| px_volume_vol_read_latency_seconds | Average time spent per successfully completed read operation in seconds during this interval for this volume. **Deprecated**, use volume_stats_read_latency_seconds. |
| px_volume_vol_write_latency_seconds | Average time spent per successfully completed write operation in seconds during this interval for this volume. **Deprecated**, use volume_write_latency_seconds. |
| px_volume_read_latency_seconds | Average time spent per successfully completed read operation in seconds for this volume |
| px_volume_write_latency_seconds | Average time spent per successfully completed write operation in seconds for this volume |
| px_volume_num_long_reads | Number of long reads for this volume |
| px_volume_num_long_writes | Number of long writes for this volume |
| px_volume_num_long_flushes | Number of long flushes for this volume |
| px_volume_dev_depth_io | Number of I/O operations currently in progress as reported by the kernel pxd device for this volume |
| px_volume_dev_writethroughput | Number of successfully written bytes per second as reported by the kernel pxd device for this volume |
| px_volume_dev_readthroughput | Number of successfully read bytes per second as reported by the kernel pxd device for this volume |
| px_volume_dev_read_latency_seconds | Average time spent per successfully completed read in seconds as reported by the kernel pxd device for this volume |
| px_volume_dev_write_latency_seconds | Average time spent per successfully completed write in seconds as reported by the kernel pxd device for this volume |
| px_volume_dev_read_bytes_total | Total number of successfully read bytes as reported by the kernel pxd device for this volume |
| px_volume_dev_written_bytes_total | Number of successfully written bytes as reported by the kernel pxd device for this volume |
| px_volume_dev_reads_total | Total number of successfully completed read operations as reported by the kernel pxd device for this volume |
| px_volume_dev_writes_total | Total number of successfully completed write operations as reported by the kernel pxd device for this volume |
| px_volume_dev_read_seconds_total | Total time spend reading in seconds for this disk as reported by the kernel pxd device for this volume |
| px_volume_dev_write_seconds_total | Total time spend writing in seconds for this disk as reported by the kernel pxd device for this volume |
| px_volume_unique_blocks | Size(in bytes) of unique blocks for this volume |
