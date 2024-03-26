# Node API

## Table of Contents

* [Service](node-api.md#service)
  * [Storage Node](node-api.md#storage-node)
* [Data Structure](node-api.md#data-structure)
  * [SegmentWithProof](node-api.md#segmentwithproof)
  * [DataRoot](node-api.md#dataroot)
  * [Segment](node-api.md#segment)
  * [FileInfo](node-api.md#fileinfo)
  * [Transaction](node-api.md#transaction)

[Top](node-api.md#top)

## Service

### Storage Node

<table><thead><tr><th width="225">Method Name</th><th width="138">Request Type</th><th width="165">Response Type</th><th>Description</th></tr></thead><tbody><tr><td>uploadSegment</td><td><a href="node-api.md#segmentwithproof">SegmentWithProof</a></td><td>-</td><td>This uploads segment to storage node</td></tr><tr><td>uploadSegments</td><td>[<a href="node-api.md#segmentwithproof">SegmentWithProof</a>]</td><td>-</td><td>This uploads multiple segments at the same time to the storage node</td></tr><tr><td>downloadSegment</td><td><a href="node-api.md#dataroot">DataRoot</a>, usize, usize</td><td><a href="node-api.md#segment">Segment</a></td><td>This download the segment by locating the data with the merkle root, start&#x26;end index</td></tr><tr><td>downloadSegmentWithProof</td><td><a href="node-api.md#dataroot">DataRoot</a>, usize</td><td><a href="node-api.md#segmentwithproof">SegmentWithProof</a></td><td>This downoads segment with the merkle proof</td></tr><tr><td>getFileInfo</td><td><a href="node-api.md#dataroot">DataRoot</a></td><td><a href="node-api.md#fileinfo">FileInfo</a></td><td>This gets the file information given the data merkle root</td></tr><tr><td>getFileInfoByTxSeq</td><td>u64</td><td><a href="node-api.md#fileinfo">FileInfo</a></td><td>This gets the file information by querying the correponsing sequence index</td></tr></tbody></table>

## Data Structure

### SegmentWithProof

| Field      | Type      | Label | Description                                      |
| ---------- | --------- | ----- | ------------------------------------------------ |
| root       | DataRoot  |       | File merkle root                                 |
| data       | \[u8]     |       | file data                                        |
| index      | usize     |       | segment index                                    |
| proof      | FileProof |       | Merkle proof whose leaf node is the segment root |
| file\_size | usize     |       | file size                                        |

### DataRoot

DataRoot = H256

### Segment

Segment = \[u8]

### FileInfo

<table><thead><tr><th width="211">Field</th><th>Type</th><th width="123">Label</th><th>Description</th></tr></thead><tbody><tr><td>tx</td><td><a href="node-api.md#dataroot">Transaction</a></td><td></td><td>file transaction</td></tr><tr><td>finalized</td><td>bool</td><td></td><td>whether the file is finalized</td></tr><tr><td>is_cached</td><td>bool</td><td></td><td>whether the file is cached</td></tr><tr><td>uploaded_seg_num</td><td>usize</td><td></td><td>number of the segments</td></tr></tbody></table>

### Transaction

<table><thead><tr><th width="211">Field</th><th>Type</th><th width="123">Label</th><th>Description</th></tr></thead><tbody><tr><td>stream_ids</td><td>[u256]</td><td></td><td>file transaction</td></tr><tr><td>data</td><td>[u8]</td><td></td><td>transaction data</td></tr><tr><td>data_merkle_root</td><td>DataRoot</td><td></td><td>merkle root of the data</td></tr><tr><td>merkle_nodes</td><td>[(usize, DataRoot)]</td><td></td><td>[(subtree_depth, subtree_root)]</td></tr><tr><td>start_entry_index</td><td>u64</td><td></td><td>index of the start entry</td></tr><tr><td>size</td><td>u64</td><td></td><td>data size</td></tr><tr><td>seq</td><td>u64</td><td></td><td>sequence number</td></tr></tbody></table>
