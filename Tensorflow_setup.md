# TensorFlow-GPU Setup on Barebone Arch Linux (Stable & Reproducible)

## 1. Install Required System Packages

Install essential tools:

```bash
sudo pacman -S base-devel git wget python311 python311-pip python311-setuptools
```

Verify Python:

```bash
python3.11 --version
```

TensorFlow 2.15 requires **Python 3.11**.

---

## 2. Install CUDA & CUDA-Tools (12.4.1-4) from Arch Archive

```bash
sudo pacman -U https://archive.archlinux.org/packages/c/cuda/cuda-12.4.1-4-x86_64.pkg.tar.zst
sudo pacman -U https://archive.archlinux.org/packages/c/cuda-tools/cuda-tools-12.4.1-4-x86_64.pkg.tar.zst
```

Check:

```bash
nvcc --version
```

---

## 3. Install cuDNN (8.9.x)

```bash
sudo pacman -U https://archive.archlinux.org/packages/c/cudnn/cudnn-8.9.7.29-1-x86_64.pkg.tar.zst
```

Verify:

```bash
ls /usr/lib/libcudnn*
```

---

## 4. Create TensorFlow Python Environment

```bash
python3.11 -m venv ~/venvs/tf --system-site-packages
source ~/venvs/tf/bin/activate
pip install --upgrade pip setuptools wheel
```

---

## 5. Install TensorFlow (GPU-enabled)

TensorFlow 2.15 supports:

* CUDA 12.x
* cuDNN 8.x
* Python 3.11

Install:

```bash
pip install tensorflow==2.15.0
```

---

## 6. Register Jupyter Kernel (Optional)

```bash
pip install ipykernel jupyterlab
python -m ipykernel install --user --name tf --display-name "Python 3.11 (TF-GPU)"
```

Start Jupyter:

```bash
jupyter-lab
```

Select **Python 3.11 (TF-GPU)**.

---

## 7. Test GPU Access

```python
import tensorflow as tf
print(tf.config.list_physical_devices("GPU"))
```

Expected:

```
[PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```

---

## 8. Freeze Critical Packages (Prevent Breakage)

Edit pacman.conf:

```bash
sudo nano /etc/pacman.conf
```

Add:

```
IgnorePkg = cuda cuda-tools cudnn nccl tensorflow-opt-cuda python311 python311-pip python311-setuptools
```

This prevents accidental upgrades that break TensorFlow.

---

## 9. Optional: Check Live GPU Usage

```bash
watch -n 0.5 nvidia-smi
```

Run a TensorFlow op to observe GPU activity.

---

## Setup Complete

You now have:

* CUDA 12.4
* cuDNN 8.9
* TensorFlow 2.15 GPU
* Stable Python 3.11 environment
* Pacman-protected ML stack

This configuration is fully stable for long-term deep learning work.
