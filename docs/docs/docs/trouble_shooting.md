# Trouble Shooting

## Vscode debugger does not work

> related issue: [IsaacLab/issues/3305](https://github.com/isaac-sim/IsaacLab/issues/3305)

When you launch the program using VSCode Python Debugger, you may encounter the following error:

```shell
OSError: libstdc++.so.6: version `GLIBCXX_3.4.30' not found
```

Please try installing the corresponding dependencies in your conda environment:
```shell
conda install -c conda-forge gcc=12 -y
```

## Isaac Sim crashes on startup with newer NVIDIA drivers

> related issue: [isaac-sim/IsaacSim#912](https://github.com/isaac-sim/IsaacSim/issues/912)

Isaac Sim 5.1 (the version LeIsaac pins to) is validated against the NVIDIA **580** driver branch.
With newer branches the app can crash during RTX initialisation, before any stage is loaded, with an
access violation inside `rtx.scenedb`. `import isaacsim` still succeeds, so it does not look like a
packaging problem.

Observed on the same install:

| OS | Driver | Backend | Result |
|---|---|---|---|
| Windows 11 | 610.62 | Vulkan (default) | crash |
| Windows 11 | 610.62 | D3D12 (`--/app/vulkan=false`) | works |
| Ubuntu 22.04 | 615.71.09 | Vulkan (default) | crash |
| Ubuntu 22.04 | 580.126.09 | Vulkan (default) | works |

### Workaround on Windows: force the Direct3D 12 backend

Pass the Kit argument through `--kit_args`:

```shell
python scripts/environments/teleoperation/teleop_se3_agent.py \
  --task=LeIsaac-SO101-PickOrange-v0 \
  --teleop_device=keyboard \
  --num_envs=1 \
  --device=cuda \
  --enable_cameras \
  --kit_args="--/app/vulkan=false"
```

A quick check before installing the full stack, to see whether your machine is affected:

```python
from isaacsim import SimulationApp
app = SimulationApp({"headless": True})
print("OK")
app.close()
```

If this prints `OK` the GPU/driver combination is fine; if the process dies silently, try adding
`--/app/vulkan=false`.

### Alternative: stay on the 580 driver branch

Isaac Sim 5.1 documents `580.65.06` on Linux and `580.88` on Windows. Staying on the 580 branch
avoids the issue entirely, at the cost of not running the newest driver.