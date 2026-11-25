# tpt-prism Todo

Source: `spec.txt` (v2, Formal Contracts). Copyright TPT Solutions.

## Decisions

- **License:** dual `MIT OR Apache-2.0`. Supersedes the spec's MIT-only rule.
- **Dependencies:** no Apache-only crates. Every dependency must offer MIT or be permissive. Apache-2.0 is allowed as one option of a dual license. GPL, AGPL, LGPL and MPL are denied.
- **Build:** zero `build.rs`, no bindgen in CI, no naga. Shaders are compiled offline and embedded with `include_bytes!`.
- **Backends:** Vulkan, Metal and DX12. Shaders compile to SPIR-V, MSL and DXIL.
- **Layout:** Cargo workspace with five crates (`tpt-prism-surface`, `-pipeline`, `-command`, `-image`, `-contracts`) and an umbrella `tpt-prism` crate.
- **Platforms:** Windows, Linux (X11 and Wayland) and macOS. Prism accepts raw window handles and never creates windows itself.
- **Memory:** `tpt-matrix` is not built yet. Prism ships a minimal allocator behind a `GpuMemory` trait. `tpt-matrix` implements the trait later.
- **Verification:** Kani, run in CI as dev tooling.

---

## Phase 0: Project Foundation

- [ ] Create Cargo workspace with the five member crates and the `tpt-prism` umbrella crate
- [ ] Set `license = "MIT OR Apache-2.0"` and copyright TPT Solutions in every `Cargo.toml`
- [ ] Add `LICENSE-MIT` and `LICENSE-APACHE` at the repo root
- [ ] Add SPDX header `MIT OR Apache-2.0` to every source file
- [ ] Write `deny.toml`: allow MIT, Apache-2.0, Unicode-3.0, BSD-2/3-Clause, ISC, Zlib. Deny GPL, AGPL, LGPL, MPL
- [ ] Add a `deny.toml` rule or CI check that rejects Apache-only crates (every dependency must offer MIT or be permissive)
- [ ] Add a CI check that fails if any `build.rs` file exists
- [ ] Add a CI check that fails if `bindgen` or `naga` appears in the dependency tree
- [ ] Pin the Rust toolchain (`rust-toolchain.toml`)
- [ ] Set up CI: `cargo build`, `cargo test`, `cargo clippy`, `cargo fmt --check`, `cargo deny check` on Windows, Linux and macOS
- [ ] Install and configure Kani; add a CI job that runs proof harnesses
- [ ] Document the local bindgen workflow: generate FFI on a dev machine, commit the output `.rs` files, delete any `build.rs`
- [ ] Document the offline shader workflow: `glslangValidator` (SPIR-V), `dxc` (DXIL), MSL compiler (Metal)
- [ ] Write `README.md`: purpose, non-goals, licence, and the v1.0 scope list from spec section 4
- [ ] Write `CONTRIBUTING.md` with the dependency policy and the contract style guide

---

## Phase 1: Surface & Swapchain (Weeks 1-4)

**FFI generation (spec 2.2)**
- [ ] Generate and commit Vulkan FFI: surface and swapchain extensions only
- [ ] Generate and commit Metal FFI for `CAMetalLayer`
- [ ] Generate and commit DX12 FFI for swapchain
- [ ] Confirm `cargo build` works with no C toolchain installed on each platform

**Memory (spec 3.5, interim)**
- [ ] Define the `GpuMemory` trait (allocate, free, map, bounds info)
- [ ] Implement a minimal built-in allocator behind `GpuMemory`
- [ ] Add `requires` and `ensures` contracts on allocator methods as `debug_assert!`

**Core types (spec 3.1, 2.4.1)**
- [ ] Define raw window and display handle input (`raw-window-handle`, checked for MIT or permissive licence)
- [ ] Implement device and queue bootstrap for Vulkan, Metal and DX12
- [ ] Implement `Surface` for Windows (HWND), Linux (X11 and Wayland) and macOS (`CAMetalLayer`)
- [ ] Implement `Swapchain` creation and recreation
- [ ] Implement `PresentMode`: `VSync`, `Immediate`, `Mailbox`
- [ ] Define type-state lifecycle: `Acquired` → `Rendered` → `Presented`
- [ ] Implement `Swapchain::acquire_next_image()` returning `ImageIndex`
- [ ] Add a contract that `ImageIndex` is within the swapchain image count
- [ ] Implement `Swapchain::present()`
- [ ] Confirm no hidden frame pacing or automatic resize logic (spec 2.5)

**Milestone**
- [ ] Open a window and clear the screen to a solid colour at 60fps on Windows, Linux and macOS

---

## Phase 2: Pipeline & Command Recording (Weeks 5-8)

**Render passes (spec 3.2)**
- [ ] Implement `RenderPass` with attachment descriptions and subpass dependencies
- [ ] Implement `Framebuffer` binding `ImageView`s to a `RenderPass`
- [ ] Explicitly exclude `VK_KHR_dynamic_rendering` (spec 4)

**Shader pipeline (spec 2.3)**
- [ ] Write a build script or documented command that compiles shaders offline to SPIR-V, MSL and DXIL
- [ ] Commit compiled shader binaries to the repo
- [ ] Embed shader bytes with `include_bytes!`
- [ ] Verify the shader pipeline adds no runtime compiler dependency

**Graphics pipeline (spec 2.4.1, 3.2)**
- [ ] Implement `GraphicsPipeline` bundling vertex input, rasterization, depth/stencil (stub), and colour blend state
- [ ] Bind `GraphicsPipeline` to a specific `RenderPass` at creation (type-state)
- [ ] Support standard alpha blending (SourceOver) only (spec 4)
- [ ] Make viewport and scissor explicit on the pipeline (spec 2.5)

**Command recording (spec 3.3, 2.4.1)**
- [ ] Implement `CommandEncoder<State>` with `Idle`, `InRenderPass` and `InComputePass` states
- [ ] Implement `begin_render_pass` (Idle → InRenderPass)
- [ ] Implement `end` (InRenderPass → Idle)
- [ ] Implement `bind_pipeline`
- [ ] Implement `bind_vertex_buffer`
- [ ] Implement `draw`, callable only in `InRenderPass`
- [ ] Add compile-fail tests (e.g. `trybuild`) showing `draw` outside a render pass does not compile
- [ ] Implement `begin_compute_pass` as a type-state stub; full compute is out of scope for v1.0

**Verification (spec 2.4.3)**
- [ ] Write Kani harnesses for command buffer submission state
- [ ] Add the Kani harness job to CI

**Milestone**
- [ ] Render a static, untextured, coloured triangle to the screen on all three backends

---

## Phase 3: Textures & Samplers (Weeks 9-12)

**Images (spec 3.4)**
- [ ] Implement `Texture` allocation with mipmap support and standard formats
- [ ] Implement `ImageView` for a mip level or array layer range
- [ ] Reject invalid `ImageView` sub-ranges at construction (contract plus `Result`)
- [ ] Implement `Sampler` with linear filtering and clamp-to-edge

**Upload (spec 2.4.2, Phase 3 roadmap)**
- [ ] Implement CPU-to-GPU texture upload through `GpuMemory`
- [ ] Add `requires` and `ensures` contracts to every upload function, as in spec 2.4.2
- [ ] Add a unit test that upload round-trips bytes

**Shader binding**
- [ ] Add descriptor or binding model for sampled textures
- [ ] Update the offline shader pipeline for texture sampling

**Milestone**
- [ ] Render a textured 2D quad from a PNG loaded from disk

---

## Phase 4: Depth, Stencil & Indexing (Weeks 13-16)

**Depth and stencil (spec 3.2)**
- [ ] Complete depth/stencil state on `GraphicsPipeline`
- [ ] Add depth attachment support to `RenderPass` and `Framebuffer`
- [ ] Support depth write on and off per pipeline (spec 2.5)

**Indexing (spec 3.3)**
- [ ] Implement `bind_index_buffer`
- [ ] Implement `draw_indexed`

**Uniforms**
- [ ] Implement uniform buffer creation and updates
- [ ] Support MVP matrix upload

**Verification (spec 2.4.3)**
- [ ] Run the full Kani pass on the VRAM sub-allocator
- [ ] Prove no overlapping regions for allocation requests under the defined bound

**Milestone**
- [ ] Render a rotating, depth-tested, textured 3D cube

---

## Phase 5: tpt-nova-render Migration (Weeks 17-24)

- [ ] Add a feature flag in `tpt-nova-render` to switch between wgpu and tpt-prism plus tpt-matrix
- [ ] Port the PBR pipeline to pre-compiled shaders and tpt-prism APIs
- [ ] Port the Sprite pipeline to pre-compiled shaders and tpt-prism APIs
- [ ] Integrate `tpt-matrix` by implementing `GpuMemory` (replaces the interim allocator)
- [ ] Benchmark startup time against wgpu
- [ ] Benchmark binary size against wgpu
- [ ] Record results and confirm or revise the claimed wins

**Milestone**
- [ ] tpt-nova runs entirely on the pure-MIT, zero-build-script, formally-contracted TPT GPU stack

---

## Phase 6: Hardening & v1.0 Release

- [ ] Run the full cross-platform CI matrix (Windows, Linux X11 and Wayland, macOS) on every backend
- [ ] Audit every `requires` and `ensures` contract against the spec
- [ ] Audit the dependency tree against `deny.toml` and the dual-licence rule
- [ ] Confirm no `build.rs` and no runtime shader compiler remain
- [ ] Confirm the v1.0 exclusion list (spec 4) is not violated: no ray tracing, mesh shaders, tessellation, geometry shaders, dynamic rendering, dual-source blending, or multi-GPU
- [ ] Complete rustdoc for all public items
- [ ] Write the `CHANGELOG.md`
- [ ] Decide crates.io publish order for the five crates and the umbrella crate
- [ ] Publish v1.0 and tag the release
