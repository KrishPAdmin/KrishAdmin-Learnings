# Datacenter GPU LLM Serving Roadmap

This document tracks my learning path for LLM serving, GPU infra, and agentic systems using a Proxmox host with a 4060 Ti GPU and Linux virtual machines. It is written to be pushed as a learning log to GitHub.

---

## 0. Environment Context

- **Host**: Proxmox VE, consumer 4060 Ti GPU
- **Guests**: Linux VMs with GPU passthrough
- **Goal**: Use this as a mini datacenter to learn LLM serving, performance tuning, and agentic AI patterns that transfer to real datacenter GPU work

Key tasks:

- Configure GPU passthrough to a VM
- Install NVIDIA drivers and CUDA inside the VM
- Run one or more LLM servers on the VM
- Build simple RAG and agentic workflows on top

---

## 1. Core Theory: LLM Inference Fundamentals

### 1.1 Concepts to understand

- Tokens and tokenization
- Transformer forward pass at inference time
- Autoregressive decoding
- **KV cache** (key value cache) and how it grows with context length
- **Batching** and continuous batching
- **Quantization** (FP16, INT8, 4 bit and effects on quality and VRAM)
- The relationship between:
  - Model size
  - Context length
  - Batch size
  - Latency, throughput, and VRAM

### 1.2 Suggested resources

- Andrej Karpathy playlist (especially “Intro to Large Language Models” and “Deep Dive into LLMs like ChatGPT”)  
  - https://www.youtube.com/playlist?list=PLAqhIrjkxbuW9U8-vZ_s_cjKPT_FqRStI  
  - Channel: https://www.youtube.com/AndrejKarpathy  

---

## 2. GPU and CUDA Basics inside a VM

### 2.1 Skills

- Set up a Linux VM on Proxmox with PCIe GPU passthrough
- Install and verify NVIDIA drivers
- Install CUDA toolkit
- Run CUDA samples to confirm GPU compute is working
- Use `nvidia-smi` to check utilization, VRAM, processes, power, and temperature

### 2.2 Proxmox GPU passthrough references

- Proxmox forum thread: “2025 Proxmox PCIe / GPU Passthrough with NVIDIA”  
  - https://forum.proxmox.com/threads/2025-proxmox-pcie-gpu-passthrough-with-nvidia.169543/  
- Ultimate beginners guide to GPU passthrough (Proxmox oriented)  
  - https://www.reddit.com/r/homelab/comments/b5xpua/the_ultimate_beginners_guide_to_gpu_passthrough/  
- Gist style guide  
  - https://gist.github.com/KasperSkytte/6a2d4e8c91b7117314bceec84c30016b  

### 2.3 CUDA toolkit and samples

- CUDA toolkit main page  
  - https://developer.nvidia.com/cuda/toolkit  
- CUDA toolkit documentation  
  - https://docs.nvidia.com/cuda/  
- CUDA samples (GitHub)  
  - https://github.com/NVIDIA/cuda-samples  
- CUDA samples docs  
  - https://docs.nvidia.com/cuda/cuda-samples/index.html  

---

## 3. Local LLMs on a Single GPU VM

### 3.1 Goals

- Run local LLMs on the 4060 Ti inside a VM
- Explore VRAM limits with 7B, 8B and similar models
- Compare quantization levels and their impact on tokens per second

### 3.2 Tools

**Ollama**

- Official documentation  
  - https://docs.ollama.com/  
- English documentation  
  - https://ollama.readthedocs.io/en/  
- GitHub repository  
  - https://github.com/ollama/ollama  

Suggested workflow:

- Install Ollama on the GPU VM
- Pull a 7B model (for example a Llama or Mistral variant)
- Measure:
  - VRAM usage
  - Tokens per second
  - Latency for short and long prompts

Optional:

- Pair Ollama with a simple web UI such as Open WebUI or similar frontends

---

## 4. High Performance LLM Serving with vLLM

### 4.1 Concepts

- LLM server as a long running process
- Loading models from Hugging Face or local checkpoints
- Continuous batching and request scheduling
- KV cache management and paged attention
- OpenAI compatible APIs as a standard interface
- Multi model and multi GPU considerations (for later)

### 4.2 vLLM resources

- vLLM documentation  
  - https://docs.vllm.ai/en/latest/  
- vLLM GitHub repository  
  - https://github.com/vllm-project/vllm  
- vLLM Omni (for multi modality and more advanced setups)  
  - https://github.com/vllm-project/vllm-omni  

Suggested practice:

- Install vLLM in a Python environment in the GPU VM
- Start the OpenAI compatible vLLM server with a 7B model
- Write a small Python client that:
  - Sends requests
  - Times latency
  - Logs tokens per second
- Run simple load tests with multiple concurrent requests

---

## 5. RAG and Tool Use (LangChain and Friends)

### 5.1 Goals

- Build small applications that turn a raw LLM into a useful system
- Implement a simple RAG pipeline that uses local files as knowledge
- Learn how to call tools from LLMs (for example shell, HTTP, or GPU telemetry)

### 5.2 LangChain and ecosystem

- LangChain official docs  
  - https://docs.langchain.com/  
- LangChain main site  
  - https://www.langchain.com/  
- Legacy reference docs  
  - https://langchain-doc.readthedocs.io/en/latest/index.html  

Suggested steps:

- Use LangChain with:
  - A local LLM endpoint (Ollama or vLLM OpenAI compatible)
  - A small vector store built from your own markdown notes or PDFs
- Build a basic RAG chain:
  - Ingest documents
  - Embed and store in a vector DB (can be in memory or file based)
  - Query with retrieval followed by generation

---

## 6. Agentic AI Systems

### 6.1 Concepts

- Agent vs plain chat model
- Tool use
- Reflection and self correction loops
- Planning multi step workflows
- Multi agent setups (planner, worker, critic)

### 6.2 Resources

**Agentic AI course by Andrew Ng**

- Course information  
  - https://learn.deeplearning.ai/courses/agentic-ai/information  

**Background and commentary**

- LinkedIn announcement about the Agentic AI course  
  - https://www.linkedin.com/posts/andrewyng_announcing-my-new-course-agentic-ai-building-activity-7381380126317404160-wW75  

Suggested practice:

- Implement a simple agent that:
  - Talks to a local LLM endpoint
  - Has at least two tools (for example file search, HTTP fetch, GPU stats script)
  - Uses a loop to decide which tool to call next until a task is completed

---

## 7. Production Mindset and Best Practices

### 7.1 Topics

- Observability:
  - Metrics, logs, traces
- Latency and throughput:
  - p50 and p95 latencies, tokens per second, requests per second
- Cost and resource efficiency:
  - How many tokens per second per GPU
  - Effect of batch size and quantization on utilization
- Reliability:
  - Timeouts, retries, backoff, circuit breakers
- Security:
  - Secrets management
  - Network exposure of admin endpoints
  - Data handling in logs

### 7.2 Resources

- OpenAI production best practices  
  - https://platform.openai.com/docs/guides/production-best-practices  
- Model optimization and prompting tips  
  - https://platform.openai.com/docs/guides/model-optimization  
- General article on best practices for using OpenAI models in production environments  
  - https://milvus.io/ai-quick-reference/what-are-the-best-practices-for-using-openai-models-in-production-environments  

---

## 8. Skill Checklist for Datacenter GPU and LLM Infra

### 8.1 Systems and Linux

- [ ] Comfortable with Linux shell and package management
- [ ] Can configure and debug services on a VM
- [ ] Understand basic networking, firewalls, and ports

### 8.2 GPU and drivers

- [ ] Set up GPU passthrough on Proxmox to a VM
- [ ] Install NVIDIA drivers and CUDA
- [ ] Run CUDA samples successfully
- [ ] Use `nvidia-smi` to monitor utilization and VRAM

### 8.3 LLM serving

- [ ] Run local models with Ollama in the VM
- [ ] Run vLLM OpenAI compatible server
- [ ] Write small clients that call the server and measure performance
- [ ] Perform basic load testing on a single GPU

### 8.4 Applications and agents

- [ ] Build a small RAG app over local documents
- [ ] Implement a simple agent that calls tools and runs multi step workflows
- [ ] Experiment with different models and record tradeoffs in VRAM, speed, and quality

---

## 9. Personal Experiments to Log

Ideas to log as separate markdown files or sections:

1. **Single GPU throughput experiment**

   - Model, quantization, context length, batch size
   - Throughput and latency
   - GPU utilization plots or numbers

2. **Agent that monitors GPU health**

   - Simple tool that reads GPU stats
   - Agent that summarizes and suggests actions in natural language

3. **Model tradeoff report for my environment**

   - Table with:
     - Model name
     - Parameters
     - Quantization
     - VRAM usage
     - Tokens per second
     - Subjective quality notes

Keeping these as structured notes will make this repo a solid portfolio artifact for datacenter GPU and LLM infra roles.
