---
title: "KFunc 'bpf_percpu_obj_new_impl'"
description: "This page documents the 'bpf_percpu_obj_new_impl' eBPF kfunc, including its definition, usage, program types that can use it, and examples."
---
# KFunc `bpf_percpu_obj_new_impl`

<!-- [FEATURE_TAG](bpf_percpu_obj_new_impl) -->
[:octicons-tag-24: v6.7](https://github.com/torvalds/linux/commit/36d8bdf75a93190e5669b9d1d95994e13e15ba1d)
<!-- [/FEATURE_TAG] -->

Allocates a pre-CPU object.

## Definition

Allocates a pre-CPU object of the type represented by `local_type_id` in program BTF. User may use the bpf_core_type_id_local macro to pass the type ID of a struct in program BTF.

The `local_type_id` parameter must be a known constant. The 'meta' parameter is rewritten by the verifier, no need for BPF program to set it.

**Returns**

A pointer to a pre-CPU object of the type corresponding to the passed in `local_type_id`, or NULL on failure.

**Signature**

<!-- [KFUNC_DEF] -->
`#!c void *bpf_percpu_obj_new_impl(u64 local_type_id__k, void *meta__ign)`

!!! note
	This kfunc returns a pointer to a refcounted object. The verifier will then ensure that the pointer to the object 
	is eventually released using a release kfunc, or transferred to a map using a referenced kptr 
	(by invoking [`bpf_kptr_xchg`](../helper-function/bpf_kptr_xchg.md)). If not, the verifier fails the 
	loading of the BPF program until no lingering references remain in all possible explored states of the program.

!!! note
	The pointer returned by the kfunc may be NULL. Hence, it forces the user to do a NULL check on the pointer returned 
	from the kfunc before making use of it (dereferencing or passing to another helper).
<!-- [/KFUNC_DEF] -->

## Usage

!!! example "Docs could be improved"
    This part of the docs is incomplete, contributions are very welcome

### Program types

The following program types can make use of this kfunc:

<!-- [KFUNC_PROG_REF] -->
- [`BPF_PROG_TYPE_LSM`](../program-type/BPF_PROG_TYPE_LSM.md)
- [`BPF_PROG_TYPE_PERF_EVENT`](../program-type/BPF_PROG_TYPE_PERF_EVENT.md) [:octicons-tag-24: v6.12](https://github.com/torvalds/linux/commit/bc638d8cb5be813d4eeb9f63cce52caaa18f3960) - 
- [`BPF_PROG_TYPE_SCHED_CLS`](../program-type/BPF_PROG_TYPE_SCHED_CLS.md)
- [`BPF_PROG_TYPE_STRUCT_OPS`](../program-type/BPF_PROG_TYPE_STRUCT_OPS.md)
- [`BPF_PROG_TYPE_TRACEPOINT`](../program-type/BPF_PROG_TYPE_TRACEPOINT.md) [:octicons-tag-24: v6.12](https://github.com/torvalds/linux/commit/bc638d8cb5be813d4eeb9f63cce52caaa18f3960) - 
- [`BPF_PROG_TYPE_TRACING`](../program-type/BPF_PROG_TYPE_TRACING.md)
- [`BPF_PROG_TYPE_XDP`](../program-type/BPF_PROG_TYPE_XDP.md)
<!-- [/KFUNC_PROG_REF] -->

### Example

!!! example "Docs could be improved"
    This part of the docs is incomplete, contributions are very welcome

