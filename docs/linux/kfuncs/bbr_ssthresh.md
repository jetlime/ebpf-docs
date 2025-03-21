---
title: "KFunc 'bbr_ssthresh'"
description: "This page documents the 'bbr_ssthresh' eBPF kfunc, including its definition, usage, program types that can use it, and examples."
---
# KFunc `bbr_ssthresh`

<!-- [FEATURE_TAG](bbr_ssthresh) -->
[:octicons-tag-24: v5.13](https://github.com/torvalds/linux/commit/e78aea8b2170be1b88c96a4d138422986a737336)
<!-- [/FEATURE_TAG] -->

Returns slow start threshold

## Definition

Entering loss recovery, so save `cwnd` for when we exit or undo recovery.

**Signature**

<!-- [KFUNC_DEF] -->
`#!c u32 bbr_ssthresh(struct sock *sk)`
<!-- [/KFUNC_DEF] -->

## Usage

!!! example "Docs could be improved"
    This part of the docs is incomplete, contributions are very welcome

### Program types

The following program types can make use of this kfunc:

<!-- [KFUNC_PROG_REF] -->
- [`BPF_PROG_TYPE_STRUCT_OPS`](../program-type/BPF_PROG_TYPE_STRUCT_OPS.md)
<!-- [/KFUNC_PROG_REF] -->

### Example

!!! example "Docs could be improved"
    This part of the docs is incomplete, contributions are very welcome

