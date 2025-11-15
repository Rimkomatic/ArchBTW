# TensorFlow GPU Setup (CUDA 12.x + New GPU Architectures)

A detailed walkthrough of **what to do**, **why it must be done**, and **what each step fixes**.

---

## Background

New NVIDIA GPUs (like **RTX 50‑series**) use **Compute Capability 12.0**, which TensorFlow doesn’t fully support yet. CUDA 12.x also reorganized the internal file `libdevice.10.bc` — a critical bitcode library XLA needs for GPU kernel compilation.

Because of this mismatch, TensorFlow often crashes with:

```
InternalError: libdevice not found at ./libdevice.10.bc
```

This guide ensures TensorFlow finds the correct `libdevice` file and initializes the GPU correctly.

---

# 1. Check GPU Visibility

Before anything else, confirm TensorFlow can **see** the GPU.

```python
import tensorflow as tf
print("GPUs:", tf.config.list_physical_devices('GPU'))
```

### **Why?**

This verifies:

* your GPU driver is working,
* CUDA is installed,
* TensorFlow’s runtime can communicate with the GPU.

If this step fails, nothing else will work.

---

# 2. Warm Up the GPU (Trigger Kernel Compilation)

```python
@tf.function
def gpu_warmup():
    x = tf.random.normal((1024, 1024))
    for _ in range(10):
        x = tf.matmul(x, x)
    return tf.reduce_sum(x)

_ = gpu_warmup()
print("GPU warm-up done.")
```

### **Why?**

The *first* GPU operation forces TensorFlow to:

* load CUDA kernels,
* allocate device memory,
* run XLA’s device-specific compiler,
* try to locate `libdevice.10.bc`.

If it succeeds: your GPU environment is fundamentally working.
If it fails: the `libdevice` fix is required.

---

# 3. Fix Missing `libdevice.10.bc` Automatically

Place this **at the very top** of your script, **before importing TensorFlow**.

```python
import os, glob, warnings

os.environ["TF_CPP_MIN_LOG_LEVEL"] = "2"
warnings.filterwarnings("ignore")

candidates = [
    "/usr/local/cuda-12.4",
    "/usr/local/cuda",
    "/opt/cuda",
    "/usr/local/cuda-12.3",
    "/usr/local/cuda-12.5",
    "/usr/local/cuda-11.8"
]

found = None
libdevice_file = None

for root in candidates:
    nvvm_dir = os.path.join(root, "nvvm", "libdevice")
    if os.path.isdir(nvvm_dir):
        matches = sorted(glob.glob(os.path.join(nvvm_dir, "libdevice*.bc")))
        if matches:
            found = root
            libdevice_file = matches[-1]
            break

if libdevice_file is None:
    for prefix in ("/usr", "/opt", "/home"):
        matches = sorted(glob.glob(os.path.join(prefix, "**", "libdevice*.bc"), recursive=True))
        if matches:
            libdevice_file = matches[-1]
            found = os.path.abspath(os.path.join(libdevice_file, "..", ".."))
            break

if libdevice_file is None:
    print("Warning: libdevice not found.")
else:
    os.environ["CUDA_ROOT"] = found
    os.environ["XLA_FLAGS"] = f"--xla_gpu_cuda_data_dir={found}"

    expected = os.path.join(os.getcwd(), "libdevice.10.bc")
    if not os.path.exists(expected):
        os.symlink(libdevice_file, expected)
        print("Created symlink:", expected, "->", libdevice_file)
    else:
        print("Symlink already exists.")
```

### **Why?**

TensorFlow incorrectly looks for:

```
./libdevice.10.bc
```

But CUDA 12.x stores it in:

```
/opt/cuda/nvvm/libdevice/
```

This block:

1. Searches for the **real** libdevice file.
2. Points XLA to the **correct CUDA installation**.
3. Creates a symlink so TensorFlow finds the file instantly.

This fully resolves the `libdevice not found` crash.

---

# 4. Import TensorFlow After Setting Environment Variables

After the fix is in place:

```python
import tensorflow as tf
```

This ensures TensorFlow initializes using the corrected CUDA configs.

---

# Why We Must Do This (Until TF Updates)

The short version:

* Your GPU uses **Compute Capability 12.0**.
* TensorFlow releases lag behind new GPU architectures.
* CUDA 12 reorganized the `libdevice` directory.
* TensorFlow still looks in **old paths**.

So we fix the path manually.

### This workaround is temporary.

Once TensorFlow officially supports **compute 12.0** and CUDA **12.4+**, this will no longer be required.

You simply remove the block and everything will "just work".

---

# Summary

You learned how to:

* verify GPU visibility,
* warm up the GPU to test real CUDA execution,
* auto-detect your CUDA install,
* guide TensorFlow to the correct libdevice location,
* prevent the `libdevice.10.bc` crash.

This is the proper way to run TensorFlow on brand‑new GPU architectures.

---

If you want, I can add a shorter quick‑start cheat sheet below or include a troubleshooting section.
