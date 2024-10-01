# サーバーから2枚のH100 GPUがNVLINK構成でどのように見えるのか

- NVLINKは、複数のGPUを１つに見せるわけではない
- NVLINKにより、高速なGPU間データ転送を実現
- サーバー機種： HPE ProLiant DL380a Gen11
- GPU： NVIDIA H100 x2枚（NVLINKで2枚を接続）
- OS： Ubuntu Server 22.04.4 LTS
- GPUドライバー：555.42.02
- CUDA: 12.5

# 単純にnvidia-smiを実行した結果

- PCIeに装着されたH100が2枚認識されている
- GPUメモリ使用量も別々に同時に確認可能

      root@dl380ag11:~# nvidia-smi
      Mon Jun 10 14:09:28 2024       
      +-----------------------------------------------------------------------------------------+
      | NVIDIA-SMI 555.42.02              Driver Version: 555.42.02      CUDA Version: 12.5     |
      |-----------------------------------------+------------------------+----------------------+
      | GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
      | Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
      |                                         |                        |               MIG M. |
      |=========================================+========================+======================|
      |   0  NVIDIA H100 PCIe               Off |   00000000:C2:00.0 Off |                    0 |
      | N/A   28C    P0             44W /  350W |       1MiB /  81559MiB |      0%      Default |
      |                                         |                        |             Disabled |
      +-----------------------------------------+------------------------+----------------------+
      |   1  NVIDIA H100 PCIe               Off |   00000000:D5:00.0 Off |                    0 |
      | N/A   29C    P0             49W /  350W |       1MiB /  81559MiB |      0%      Default |
      |                                         |                        |             Disabled |
      +-----------------------------------------+------------------------+----------------------+
                                                                                               
      +-----------------------------------------------------------------------------------------+
      | Processes:                                                                              |
      |  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
      |        ID   ID                                                               Usage      |
      |=========================================================================================|
      |  No running processes found                                                             |
      +-----------------------------------------------------------------------------------------+

# トポロジ表示
- NVLINKの場合は、NV12と表示される

      root@dl380ag11:~# nvidia-smi topo -m
              GPU0    GPU1    CPU Affinity    NUMA Affinity   GPU NUMA ID
      GPU0     X      NV12    32-63,96-127    1               N/A
      GPU1    NV12     X      32-63,96-127    1               N/A
      
      Legend:
      
        X    = Self
        SYS  = Connection traversing PCIe as well as the SMP interconnect between NUMA nodes (e.g., QPI/UPI)
        NODE = Connection traversing PCIe as well as the interconnect between PCIe Host Bridges within a NUMA node
        PHB  = Connection traversing PCIe as well as a PCIe Host Bridge (typically the CPU)
        PXB  = Connection traversing multiple PCIe bridges (without traversing the PCIe Host Bridge)
        PIX  = Connection traversing at most a single PCIe bridge
        NV#  = Connection traversing a bonded set of # NVLinks

# 対向GPU確認
- お互いに相手のGPUが見えるかを確認

      root@dl380ag11:~# nvidia-smi nvlink -R
      GPU 0: NVIDIA H100 PCIe (UUID: GPU-6968dfc2-d444-d92c-c36b-6abe53445cb9)
               Link 0: Remote Device 00000000:D5:00.0: Link 0
               Link 1: Remote Device 00000000:D5:00.0: Link 1
               Link 2: Remote Device 00000000:D5:00.0: Link 2
               Link 3: Remote Device 00000000:D5:00.0: Link 3
               Link 4: Remote Device 00000000:D5:00.0: Link 6
               Link 5: Remote Device 00000000:D5:00.0: Link 7
               Link 6: Remote Device 00000000:D5:00.0: Link 8
               Link 7: Remote Device 00000000:D5:00.0: Link 9
               Link 8: Remote Device 00000000:D5:00.0: Link 12
               Link 9: Remote Device 00000000:D5:00.0: Link 13
               Link 10: Remote Device 00000000:D5:00.0: Link 14
               Link 11: Remote Device 00000000:D5:00.0: Link 15
      GPU 1: NVIDIA H100 PCIe (UUID: GPU-9b1446e2-8a89-4cf6-b68c-a98d99e958d3)
               Link 0: Remote Device 00000000:C2:00.0: Link 0
               Link 1: Remote Device 00000000:C2:00.0: Link 1
               Link 2: Remote Device 00000000:C2:00.0: Link 2
               Link 3: Remote Device 00000000:C2:00.0: Link 3
               Link 4: Remote Device 00000000:C2:00.0: Link 6
               Link 5: Remote Device 00000000:C2:00.0: Link 7
               Link 6: Remote Device 00000000:C2:00.0: Link 8
               Link 7: Remote Device 00000000:C2:00.0: Link 9
               Link 8: Remote Device 00000000:C2:00.0: Link 12
               Link 9: Remote Device 00000000:C2:00.0: Link 13
               Link 10: Remote Device 00000000:C2:00.0: Link 14
               Link 11: Remote Device 00000000:C2:00.0: Link 15

# NVLINK帯域確認
- NVLINKのリンク速度確認

      root@dl380ag11:~# nvidia-smi nvlink -s
      GPU 0: NVIDIA H100 PCIe (UUID: GPU-6968dfc2-d444-d92c-c36b-6abe53445cb9)
               Link 0: 26.562 GB/s
               Link 1: 26.562 GB/s
               Link 2: 26.562 GB/s
               Link 3: 26.562 GB/s
               Link 4: <inactive>
               Link 5: <inactive>
               Link 6: 26.562 GB/s
               Link 7: 26.562 GB/s
               Link 8: 26.562 GB/s
               Link 9: 26.562 GB/s
               Link 10: <inactive>
               Link 11: <inactive>
               Link 12: 26.562 GB/s
               Link 13: 26.562 GB/s
               Link 14: 26.562 GB/s
               Link 15: 26.562 GB/s
               Link 16: <inactive>
               Link 17: <inactive>
      GPU 1: NVIDIA H100 PCIe (UUID: GPU-9b1446e2-8a89-4cf6-b68c-a98d99e958d3)
               Link 0: 26.562 GB/s
               Link 1: 26.562 GB/s
               Link 2: 26.562 GB/s
               Link 3: 26.562 GB/s
               Link 4: <inactive>
               Link 5: <inactive>
               Link 6: 26.562 GB/s
               Link 7: 26.562 GB/s
               Link 8: 26.562 GB/s
               Link 9: 26.562 GB/s
               Link 10: <inactive>
               Link 11: <inactive>
               Link 12: 26.562 GB/s
               Link 13: 26.562 GB/s
               Link 14: 26.562 GB/s
               Link 15: 26.562 GB/s
               Link 16: <inactive>
               Link 17: <inactive>

