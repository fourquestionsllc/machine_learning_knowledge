Here’s a summary of some key Amazon Web Services (AWS) EC2 GPU instance families, showing which GPUs they use, how many GPUs per instance type, and the GPU memory size **per GPU** (or total, when stated). This should help you pick the right size for machine-learning training.

---

| Instance Family                     | GPU Model                       | # GPUs (per instance)                                                    | GPU Memory (per GPU)                                                                                                                                         |
| ----------------------------------- | ------------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **G6 / G6e**                        | NVIDIA L4 Tensor Core           | Up to 8 GPUs (in multi-GPU variants) ([Amazon Web Services, Inc.][1])    | 24 GiB per GPU in single-GPU versions. ([Amazon Web Services, Inc.][2]) Fractional GPU versions go down to 3 GiB (1/8 GPU). ([Amazon Web Services, Inc.][2]) |
| **G5**                              | NVIDIA A10G Tensor Core         | Up to 8 GPUs ([Amazon Web Services, Inc.][1])                            | 24 GiB per GPU ([Amazon Web Services, Inc.][3])                                                                                                              |
| **G4dn**                            | NVIDIA T4 Tensor Core           | Up to 8 GPUs ([Amazon Web Services, Inc.][1])                            | 16 GiB per GPU (for typical G4dn.xlarge etc) ([Amazon Web Services, Inc.][1])                                                                                |
| **P4 / P4d**                        | NVIDIA A100 Tensor Core         | 8 GPUs in e.g. p4d.24xlarge ([Amazon Web Services, Inc.][1])             | 40 GiB (or up to 320 GiB total across 8 GPUs) ([MareArts][4])                                                                                                |
| **P5 / P5e / P5en**                 | NVIDIA H100 / H200 Tensor Core  | 8 GPUs (in the 48xlarge/large variants) ([Amazon Web Services, Inc.][1]) | Up to 640-1128 GiB total GPU memory across 8 GPUs (so ~80-141 GiB per GPU) ([Amazon Web Services, Inc.][1])                                                  |
| **P6 / P6e (future / ultra large)** | NVIDIA Blackwell (B200 / GB200) | 8 GPUs in P6-B200 etc. ([Amazon Web Services, Inc.][1])                  | Up to ~1,432 GiB total GPU memory across 8 GPUs (~179 GiB per GPU) ([Amazon Web Services, Inc.][1])                                                          |

---

### 📝 Notes & Considerations

* “GPU memory per GPU” is important for training large models (batch size + model size must fit).
* “Number of GPUs per instance” matters for parallelism and throughput (if you need multi-GPU within one node).
* Be aware of **network bandwidth** (EFA/NCCL support) when doing multi-node/multi-GPU training — the high-end instances (P4, P5, P6) give very high interconnect bandwidth. ([Amazon Web Services, Inc.][1])
* Availability of instance types may vary by AWS region and you may need to request quota increases for high-end GPU instances.
* GPU memory is distinct from system RAM. Even if GPU memory is large, you also need sufficient system memory, fast storage, and good data pipeline.

---

[1]: https://aws.amazon.com/ec2/instance-types/accelerated-computing/?utm_source=chatgpt.com "Accelerated computing"
[2]: https://aws.amazon.com/ec2/instance-types/g6/?utm_source=chatgpt.com "Amazon EC2 G6 Instances | Amazon Web Services"
[3]: https://aws.amazon.com/ec2/instance-types/g5/?utm_source=chatgpt.com "Amazon EC2 G5 Instances | Amazon Web Services"
[4]: https://study.marearts.com/2023/02/aws-ec2-gpu-instance-comparison.html?utm_source=chatgpt.com "MareArts Computer Vision Study.: AWS ec2 gpu instance comparison"
