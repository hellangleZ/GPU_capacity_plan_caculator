# ⚡ GenAI Inference Capacity Planner

> Created by Transformer-Zhou

A professional, full-stack estimation tool designed for LLM (Large Language Model) inference infrastructure planning. This calculator provides deep insights into **Latency (TTFT)**, **Generation Speed (Tokens/s)**, and **Capacity (VRAM)**, helping architects and engineers make data-driven decisions for GPU cluster deployment.

<img width="1420" height="1167" alt="image" src="https://github.com/user-attachments/assets/f7d39fa9-5177-4887-9d4b-b1196fc426e6" />


## 🚀 Key Features

* **Full-Stack Estimation**: Calculates Compute-Bound (Prefill) and Bandwidth-Bound (Decode) phases separately.
* **Advanced Hardware Logic**:
  * Built-in specs for **NVIDIA H20, H100, H800, H200**.
  * **Auto-Efficiency Penalty**: Automatically applies discounts for NVLink vs PCIe, and linear scaling penalties for Multi-Node clusters (>8 GPUs).
  * **Precision Modeling**: Supports **BF16, FP8, INT8, INT4** with accurate VRAM calculation and Compute Multipliers (e.g., FP8 = 2x TFLOPS on Hopper).
* **MoE & Architecture Support**:
  * Native support for **Mixture-of-Experts (MoE)** models (e.g., DeepSeek-V3/R1) vs Dense models.
  * **MLA / KV Cache Optimization** toggles.
* **Workload Simulation**:
  * Adjustable Batch Size, Input/Output lengths, and Target RPS.
  * Real-time **Concurrency Estimation** based on Little's Law.
* **Comparison Tool**: "Snapshot" a configuration to perform Apple-to-Apple comparisons (e.g., BF16 vs FP8 on the same hardware).
* **Visualizations**: Interactive Memory Breakdown bars and Latency Composition charts.

---

## 📖 Field Definitions & Reference Guide

### 1. Model Architecture

* **Total Params**: The total size of the model weights.
* **Active Params**:
  * For **Dense** models: Equals Total Params.
  * For **MoE** models: The number of parameters activated per token (e.g., DeepSeek-V3 has 671B total but only ~37B active). This drastically affects inference speed.
* **MLA / KV Cache Opt**: Multi-Head Latent Attention or similar optimizations. Checking this reduces KV Cache VRAM usage by ~50%.

### 2. Infrastructure

* **GPU Select**: Choose the target hardware. Note that H20/H800 are bandwidth-optimized or cut-down versions specific to certain markets.
* **Precision**:
  * **BF16 (2 bytes)**: Standard training/inference precision.
  * **FP8 (1 byte)**: Halves VRAM usage and doubles theoretical TFLOPS on Hopper architecture.
* **Force GPU Count (Optional)**:
  * **Auto (Default)**: Calculates the *minimum* GPUs needed to fit the model (Capacity Bound).
  * **Manual Input**: Force a specific cluster size (e.g., 16 GPUs). **Crucial for fair comparison**: Use this to see the speedup of FP8 vs BF16 on the *same* number of GPUs.

### 3. Workload Configuration

#### **Batch Size**

The number of requests processed simultaneously by one GPU instance.

* **Trade-off**: Higher Batch Size = Higher Throughput (System Efficiency) but Slower Latency (User Experience).
* **Reference Values**:
  * **1 - 8**: Ultra-low latency scenarios (e.g., Real-time voice conversation).
  * **16 - 32**: Balanced (Standard B2B / Enterprise Chatbots).
  * **64 - 128+**: High Throughput (B2C Apps, Offline Batch Processing).

#### **Target RPS (Requests Per Second)**

The traffic load you expect the system to handle.

* **B2B / Enterprise Internal**:
  * **0.5 - 2 RPS**: Typical for internal tools, coding assistants, or departmental RAG.
  * **3 - 5 RPS**: Large-scale group-wide deployment.
* **B2C / Consumer Apps**:
  * **10 - 50 RPS**: Successful consumer applications.
  * **100+ RPS**: Platform-level traffic (requires massive fleets).

#### **≈ Concurrent**

Calculated via **Little's Law**: `Concurrency = RPS * Total Latency`.

* Shows how many active requests are "in-flight" in the system at any given second.
* *Example*: 10 RPS with 3s Latency = 30 Concurrent requests holding VRAM.

---

## 📊 Understanding the Metrics

### **Min GPUs (Capacity)**

The absolute minimum number of GPUs required to fit the Model Weights + KV Cache + Activation Overhead.

* *Note*: If you see a "VRAM Overflow" alert, you need more GPUs or lower precision (FP8).

### **TTFT (Time To First Token)**

**Compute-Bound**. How long the user waits before the first word appears.

* Heavily influenced by **Active Params** and **GPU TFLOPS**.
* *Optimization*: Use FP8 (2x Compute) or more GPUs (Parallelism).

### **Gen Speed (Tokens/s)**

**Bandwidth-Bound**. How fast the text generates after the first token.

* **Formula**: `Total Bandwidth / Active Params`.
* *Optimization*: This is the "User Experience" speed. To make it faster, you need **more Bandwidth** (more GPUs) or **smaller Active Params** (MoE or FP8).

### **User Throughput (Users/sec)**

The system-level throughput target.

* Calculated as `Target RPS * Batch Size` in the current view context, representing the load the fleet is designed to carry.

### **Total Fleet**

The total number of GPUs required to sustain the Target RPS.

* **Formula**: `Replicas * GPUs per Replica`.
* If one instance (e.g., 8 GPUs) can handle 5 RPS, and you need 50 RPS, you need 10 Replicas (80 GPUs total).

---

## 🛠️ How to Use

1. **Clone/Download** the repository.
2. Open `gpp_capacity_plan.html` in any modern web browser (Chrome, Edge, Safari).
3. **No installation required** - it runs entirely in the browser using vanilla JavaScript and TailwindCSS via CDN.

## ⚖️ License

Open Source. Feel free to modify and use for your capacity planning needs.
