---
title: "KFunc 'scx_bpf_select_cpu_dfl'"
description: "This page documents the 'scx_bpf_select_cpu_dfl' eBPF kfunc, including its definition, usage, program types that can use it, and examples."
---
# KFunc `scx_bpf_select_cpu_dfl`

<!-- [FEATURE_TAG](scx_bpf_select_cpu_dfl) -->
[:octicons-tag-24: v6.12](https://github.com/torvalds/linux/commit/f0e1a0643a59bf1f922fa209cec86a170b784f3f)
<!-- [/FEATURE_TAG] -->

This function is the default implementation of [`sched_ext_ops.select_cpu`](../program-type/BPF_PROG_TYPE_STRUCT_OPS/sched_ext_ops.md#select_cpu)

## Definition

Can only be called from [`select_cpu`](../program-type/BPF_PROG_TYPE_STRUCT_OPS/sched_ext_ops.md#select_cpu) if the built-in CPU selection is enabled, [`sched_ext_ops.update_idle`](../program-type/BPF_PROG_TYPE_STRUCT_OPS/sched_ext_ops.md#update_idle) is missing, or [`SCX_OPS_KEEP_BUILTIN_IDLE`](../program-type/BPF_PROG_TYPE_STRUCT_OPS/sched_ext_ops.md#scx_ops_keep_builtin_idle) is set. `p`, `prev_cpu` and `wake_flags` match [`sched_ext_ops.select_cpu`](../program-type/BPF_PROG_TYPE_STRUCT_OPS/sched_ext_ops.md#select_cpu).

**Parameters**

`p:` task_struct to select a CPU for

`prev_cpu`: CPU `p` was on previously

`wake_flags`: `SCX_WAKE_*`, possible values are:

* `SCX_WAKE_FORK` (`0x02`) - Wakeup after exec
* `SCX_WAKE_TTWU` (`0x04`) - Wakeup after fork
* `SCX_WAKE_SYNC` (`0x08`) - Wakeup

`is_idle`: out parameter indicating whether the returned CPU is idle

**Returns**

The picked CPU with `is_idle` indicating whether the picked CPU is currently idle and thus a good candidate for direct dispatching.

**Signature**

<!-- [KFUNC_DEF] -->
`#!c s32 scx_bpf_select_cpu_dfl(struct task_struct *p, s32 prev_cpu, u64 wake_flags, bool *is_idle)`
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

