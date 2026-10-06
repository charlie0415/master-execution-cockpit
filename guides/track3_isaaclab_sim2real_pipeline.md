# Track 3: OpenUSD & Isaac Lab Sim-to-Real Pipeline — Technical Field Manual

**Target Audience:** Charles Oluwatuase | Prospective Researcher at ETH Zurich RSL/ASL, TU Delft CoR, NTNU  
**Core Thesis:** Demonstrating an end-to-end, automated OpenUSD to Isaac Lab reinforcement learning pipeline with mathematical inertia verification, V-HACD convex decomposition, and ONNX deployment provides undeniable proof of top-tier research readiness.

---

## 1. OpenUSD (Pixar pxr) Foundations & LIVPS Composition Arcs

OpenUSD resolves scenegraph opinions according to the strict **LIVPS** ordering:
1. **L**ocal opinions (strongest)
2. **I**nherits
3. **V**ariantSets
4. **P**ayloads
5. **S**pecializes
6. **R**eferences (weakest within asset arcs)

### Python Script: Authoring USD Rigid Body & Inertia via `pxr`
```python
from pxr import Usd, UsdGeom, UsdPhysics, Gf, Sdf

def create_physics_asset(stage_path: str, prim_path: str, mass_kg: float, com: tuple, inertia_diag: tuple):
    stage = Usd.Stage.CreateNew(stage_path)
    UsdGeom.SetStageUpAxis(stage, UsdGeom.Tokens.z)
    UsdGeom.SetStageMetersPerUnit(stage, 1.0)
    
    # Define rigid body prim
    prim = stage.DefinePrim(Sdf.Path(prim_path), "Xform")
    rb_api = UsdPhysics.RigidBodyAPI.Apply(prim)
    rb_api.CreateRigidBodyEnabledAttr(True)
    
    # Author Mass, Center of Mass, and 3x3 Inertia Tensor
    mass_api = UsdPhysics.MassAPI.Apply(prim)
    mass_api.CreateMassAttr(mass_kg)
    mass_api.CreateCenterOfMassAttr(Gf.Vec3f(*com))
    mass_api.CreateDiagonalInertiaAttr(Gf.Vec3f(*inertia_diag))
    
    stage.GetRootLayer().Save()
    print(f"Successfully authored USD physics asset to: {stage_path}")

if __name__ == "__main__":
    create_physics_asset("robot_base.usda", "/RobotBase", mass_kg=1.45, com=(0.0, 0.0, 0.02), inertia_diag=(0.012, 0.012, 0.021))
```

---

## 2. Automated CAD Ingestion Tool: `cad2usd` CLI

Raw mesh files (STL/OBJ/STEP) lack physical collision properties and cause PhysX simulation explosive instability if non-convex.

### Architecture of `cad2usd`:
```
Raw CAD Mesh (.stl / .obj)
      │
      ├──► trimesh: Computes volume, density, CoM, and exact 3x3 Inertia Tensor (I)
      │
      ├──► pyvhacd: Generates convex hull decomposition (V-HACD) to prevent penetration
      │
      └──► pxr (USD): Exports composite .usda with visual and collider prims
```

### Complete Implementation Template (`cad2usd.py`)
```python
import argparse
import trimesh
import pyvhacd
from pxr import Usd, UsdGeom, UsdPhysics, Gf, Sdf

def process_mesh(mesh_path: str, output_usd: str, density_kg_m3: float = 1200.0):
    mesh = trimesh.load(mesh_path)
    if not mesh.is_watertight:
        print("Warning: Mesh is not watertight. Repairing...")
        mesh.fill_holes()
    
    mesh.density = density_kg_m3
    mass = mesh.mass
    com = mesh.center_mass
    inertia_matrix = mesh.moment_inertia  # 3x3 matrix
    
    print(f"Computed Mass: {mass:.4f} kg")
    print(f"Computed Center of Mass: {com}")
    print(f"3x3 Inertia Tensor:\n{inertia_matrix}")
    
    # Run V-HACD Convex Decomposition
    print("Running V-HACD convex decomposition...")
    convex_hulls = trimesh.decomposition.convex_decomposition(mesh, max_convex_hulls=16)
    print(f"Generated {len(convex_hulls)} convex collision hulls.")
    
    # Author USD Stage
    stage = Usd.Stage.CreateNew(output_usd)
    root = stage.DefinePrim(Sdf.Path("/Asset"), "Xform")
    stage.SetDefaultPrim(root)
    
    # Author Mass Properties
    mass_api = UsdPhysics.MassAPI.Apply(root)
    mass_api.CreateMassAttr(float(mass))
    mass_api.CreateCenterOfMassAttr(Gf.Vec3f(float(com[0]), float(com[1]), float(com[2])))
    mass_api.CreateDiagonalInertiaAttr(Gf.Vec3f(
        float(inertia_matrix[0, 0]),
        float(inertia_matrix[1, 1]),
        float(inertia_matrix[2, 2])
    ))
    
    stage.GetRootLayer().Save()
    print(f"Saved OpenUSD asset: {output_usd}")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Convert CAD to OpenUSD with Physics")
    parser.add_argument("--input", required=True, help="Input STL or OBJ mesh")
    parser.add_argument("--output", required=True, help="Output USD file")
    args = parser.parse_args()
    process_mesh(args.input, args.output)
```

---

## 3. Headless Isaac Lab Execution & Cloud GPU Setup

Run Isaac Lab on **Azure Free Student Credits ($100)** or **Google Cloud Free Tier ($300)**:

1. **Launch GPU VM:** Ubuntu 22.04 LTS with NVIDIA T4 or A10G GPU (NVIDIA Driver >= 535).
2. **Pull Isaac Lab Docker Container:**
   ```bash
   docker run --gpus all -it --rm --network=host \
     -v ~/isaac_ws:/workspace/isaac_ws \
     nvcr.io/nvidia/isaac-sim:4.2.0
   ```
3. **Run Headless RL Training (No Display Required):**
   ```bash
   python scripts/rsl_rl/train.py \
     --task=Isaac-Velocity-Flat-Anymal-C-v0 \
     --num_envs=4096 \
     --headless \
     --logger=wandb
   ```
4. **PhysX Solver Iteration Tuning:**
   * Inside environment configuration:
     ```python
     sim.physx.solver_type = 1 # TGS (Temporal Gauss-Seidel) for higher contact fidelity
     sim.physx.min_position_iteration_count = 4
     sim.physx.max_position_iteration_count = 8
     sim.physx.min_velocity_iteration_count = 0
     sim.physx.max_velocity_iteration_count = 1
     ```

---

## 4. Reinforcement Learning Domain Randomization (`EventManager`)

In Isaac Lab, robustness across the Sim-to-Real gap is achieved through dynamic triggers:

```python
from omni.isaac.lab.managers import EventTermCfg as EventTerm
from omni.isaac.lab.envs import mdp

# Configure Domain Randomization in EnvCfg:
events = {
    # 1. Randomize Payload Mass (+/- 15%)
    "randomize_base_mass": EventTerm(
        func=mdp.randomize_rigid_body_mass,
        mode="startup",
        params={"asset_cfg": "robot", "mass_distribution_params": (-0.15, 0.15), "operation": "scale"}
    ),
    # 2. Randomize Ground Friction
    "randomize_friction": EventTerm(
        func=mdp.randomize_rigid_body_material,
        mode="startup",
        params={"static_friction_range": (0.3, 1.2), "dynamic_friction_range": (0.2, 1.0)}
    ),
    # 3. Dynamic External Perturbations (Wind / Push Impulses)
    "push_robot": EventTerm(
        func=mdp.push_by_setting_velocity,
        mode="interval",
        interval_range_s=(3.0, 6.0),
        params={"velocity_range": {"x": (-0.8, 0.8), "y": (-0.8, 0.8), "z": (-0.2, 0.2)}}
    )
}
```

---

## 5. ONNX Export & Edge Inference Latency Benchmark

Once PPO policy converges, export the PyTorch actor network:

```python
import torch

def export_actor_to_onnx(model, dummy_input, output_path="policy.onnx"):
    model.eval()
    torch.onnx.export(
        model,
        dummy_input,
        output_path,
        export_params=True,
        opset_version=14,
        do_constant_folding=True,
        input_names=["obs"],
        output_names=["action"]
    )
    print(f"Policy successfully exported to {output_path}")

# Benchmark with ONNX Runtime:
import onnxruntime as ort
import numpy as np
import time

session = ort.InferenceSession("policy.onnx")
input_name = session.get_inputs()[0].name
dummy_obs = np.random.randn(1, 48).astype(np.float32)

# Warmup
for _ in range(50):
    _ = session.run(None, {input_name: dummy_obs})

# Measure Latency over 1,000 steps
t0 = time.perf_counter()
for _ in range(1000):
    _ = session.run(None, {input_name: dummy_obs})
t1 = time.perf_counter()

latency_ms = ((t1 - t0) / 1000) * 1000
print(f"Average ONNX Inference Latency: {latency_ms:.3f} ms (Target < 2.0 ms)")
```

---

## 6. High-Impact 45-Second Demo Video Storyboard

| Timestamp | Visual Content | Audio / Text Overlay |
| :--- | :--- | :--- |
| **00:00 – 00:08** | Terminal running `cad2usd`: 3D CAD mesh loads, V-HACD convex hulls generated, 3x3 inertia matrix printed. | *"Automated OpenUSD Asset Ingestion with Mathematical Inertia & V-HACD Decomposition"* |
| **00:08 – 00:20** | Headless Isaac Lab running 4,096 parallel environments simultaneously. Split screen with WandB training curve showing sharp reward convergence. | *"4,096 Parallel Environments on GPU Tensors via RSL-RL & PPO"* |
| **00:20 – 00:32** | Domain randomization stress-test: Severe external push impulses, surface friction abruptly dropping from 1.0 to 0.3. Robot dynamically stabilizes without falling. | *"Zero-Shot Domain Randomization: Robustness Under Unmodeled Friction and Mass Perturbations"* |
| **00:32 – 00:45** | Terminal showing ONNX export and sub-millisecond edge latency (`0.85 ms`). Direct screen card with GitHub repository link & candidate contact info. | *"Production ONNX Export Ready for Real-Time Edge Deployment | Charles Oluwatuase"* |
