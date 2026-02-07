# Notes for upstream PR

## Fix: int64_t vs Tensor dispatch priority (src/codegen.cpp)

PyTorch 2.8.0 removed some mixed-type overloads (e.g.
`randint_like(self, low: int64, high: Tensor)`). This exposed a
pre-existing bug in the runtime dispatch (`cpp_arg_to_torch_type`):
when an argument accepts both `int64_t` and `Tensor`, a plain R
numeric scalar like `500` was dispatched as `Tensor` (via the
atomic-to-Tensor catch-all) instead of `int64_t`.

The fix moves the `int64_t` scalar check above the Tensor catch-all.
Explicit `torch_tensor()` objects still dispatch as Tensor (matched
earlier in the function). Only affects the 3 generated functions where
an argument has `expected_types = c("int64_t", "Tensor")`.

Failing test before fix: `test-creation-ops.R:45` (`torch_randint_like`).

## Fix: pyobj_slot GC race in extract_ignite_state_dict (R/ignite.R)

`optim_ignite_adagrad` state_dict fails to round-trip through
`torch_serialize`/`torch_load` in 2.8.0. Error: "missing states for
parameters NA".

**Root cause:** `extract_ignite_state_dict` uses `xptr_address` to match
optimizer parameters between two independent C++ → R conversions. The
addresses are stable thanks to `operator_sexp_tensor` caching the R
SEXP in each tensor's `TensorImpl::pyobj_slot`. However:

1. `self$param_groups` creates R tensor wrappers, caches them in pyobj_slot
2. `addresses` records their xptr_addresses
3. `param_groups = lapply(...)` replaces tensor refs with integer indices
   — the R tensor wrappers become unreferenced and GC-eligible
4. `rcpp_ignite_optim_parameters_with_state` converts the same C++ tensors
   back to R via `operator_sexp_tensor`, which calls `R_RunPendingFinalizers()`
5. The dead wrappers from step 1 are finalized, clearing pyobj_slot
6. New wrappers are created with different addresses
7. `match(new_addrs, old_addrs)` returns NA

**Why 2.8.0:** PyTorch 2.8.0 changed the Adagrad C++ constructor to
eagerly initialize state for ALL parameters (previously lazy). The extra
tensor allocations increase GC pressure, making the latent race
manifest consistently.

**Fix:** Save the tensor R objects in a local variable (`all_params`)
before overwriting `param_groups`, keeping them alive throughout the
function. This prevents their finalizers from clearing pyobj_slot.

Failing test before fix: `test-ignite.R:148` (`optim_ignite_adagrad`).
