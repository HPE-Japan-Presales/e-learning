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

# NVLINKされたGPUの動作確認サンプルプログラムの実行
- NVLINKされたGPUのP2P通信の確認

      # su - testuser01
      $ whoami
      testuser01
      testuser01@dl380ag11:~$ git clone https://github.com/NVIDIA/cuda-samples
      $ cd /home/testuser01/cuda-samples/Samples/0_Introduction/simpleP2P/
      $ make
      testuser01@dl380ag11:~/cuda-samples/Samples/0_Introduction/simpleP2P$ ./simpleP2P 
      [./simpleP2P] - Starting...
      Checking for multiple GPUs...
      CUDA-capable device count: 2
      
      Checking GPU(s) for support of peer to peer memory access...
      > Peer access from NVIDIA H100 PCIe (GPU0) -> NVIDIA H100 PCIe (GPU1) : Yes
      > Peer access from NVIDIA H100 PCIe (GPU1) -> NVIDIA H100 PCIe (GPU0) : Yes
      Enabling peer access between GPU0 and GPU1...
      Allocating buffers (64MB on GPU0, GPU1 and CPU Host)...
      Creating event handles...
      cudaMemcpyPeer / cudaMemcpy between GPU0 and GPU1: 236.01GB/s
      Preparing host buffer and memcpy to GPU0...
      Run kernel on GPU1, taking source data from GPU0 and writing to GPU1...
      Run kernel on GPU0, taking source data from GPU1 and writing to GPU0...
      Copy data back to host from GPU0 and verify results...
      Disabling peer access...
      Shutting down...
      Test passed

- P2P通信帯域計測

      $ cd /home/testuser01/cuda-samples/Samples/5_Domain_Specific/p2pBandwidthLatencyTest
      $ make
      $ ./p2pBandwidthLatencyTest 
      [P2P (Peer-to-Peer) GPU Bandwidth Latency Test]
      Device: 0, NVIDIA H100 PCIe, pciBusID: c2, pciDeviceID: 0, pciDomainID:0
      Device: 1, NVIDIA H100 PCIe, pciBusID: d5, pciDeviceID: 0, pciDomainID:0
      Device=0 CAN Access Peer Device=1
      Device=1 CAN Access Peer Device=0
      
      ***NOTE: In case a device doesn't have P2P access to other one, it falls back to normal memcopy procedure.
      So you can see lesser Bandwidth (GB/s) and unstable Latency (us) in those cases.
      
      P2P Connectivity Matrix
           D\D     0     1
           0       1     1
           1       1     1
      Unidirectional P2P=Disabled Bandwidth Matrix (GB/s)
         D\D     0      1 
           0 1622.17  26.10 
           1  25.00 1634.15 
      Unidirectional P2P=Enabled Bandwidth (P2P Writes) Matrix (GB/s)
         D\D     0      1 
           0 1634.15 256.62 
           1 256.99 1660.74 
      Bidirectional P2P=Disabled Bandwidth Matrix (GB/s)
         D\D     0      1 
           0 1669.42  35.17 
           1  35.23 1671.85 
      Bidirectional P2P=Enabled Bandwidth Matrix (GB/s)
         D\D     0      1 
           0 1667.81 509.35 
           1 509.15 1669.81 
      P2P=Disabled Latency Matrix (us)
         GPU     0      1 
           0   2.34  16.59 
           1  16.48   2.31 
      
         CPU     0      1 
           0   2.21   6.29 
           1   6.20   2.14 
      P2P=Enabled Latency (P2P Writes) Matrix (us)
         GPU     0      1 
           0   2.35   2.56 
           1   2.52   2.31 
      
         CPU     0      1 
           0   2.24   1.72 
           1   1.75   2.18 


[KOGA MASAZUMI](https://www.amazon.co.jp/stores/%E5%8F%A4%E8%B3%80%E6%94%BF%E7%B4%94/author/B0725M9C6T) ([@masazumi_koga](https://x.com/masazumi_koga))
