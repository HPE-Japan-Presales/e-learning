# NVIDIA A100 x2（非NVLINK構成）がUbuntuからどのように見えるのか
- サーバー：HPE ProLiant DL380 Gen11
- GPU；NVIDIA A100 x 2枚
- OS：Ubuntu Server 22.04.4 LTS 
- カーネル： 6.5.0-35
- GPUドライバー：555.42.02
- CUDA：12.5

# nvidia-smiの実行例
    # nvidia-smi
    Thu Jun 13 17:53:27 2024       
    +-----------------------------------------------------------------------------------------+
    | NVIDIA-SMI 555.42.02              Driver Version: 555.42.02      CUDA Version: 12.5     |
    |-----------------------------------------+------------------------+----------------------+
    | GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
    |                                         |                        |               MIG M. |
    |=========================================+========================+======================|
    |   0  NVIDIA A100-PCIE-40GB          Off |   00000000:12:00.0 Off |                    0 |
    | N/A   38C    P0             32W /  250W |       1MiB /  40960MiB |      0%      Default |
    |                                         |                        |             Disabled |
    +-----------------------------------------+------------------------+----------------------+
    |   1  NVIDIA A100-PCIE-40GB          Off |   00000000:8A:00.0 Off |                    0 |
    | N/A   37C    P0             37W /  250W |       1MiB /  40960MiB |      0%      Default |
    |                                         |                        |             Disabled |
    +-----------------------------------------+------------------------+----------------------+
                                                                                             
    +-----------------------------------------------------------------------------------------+
    | Processes:                                                                              |
    |  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
    |        ID   ID                                                               Usage      |
    |=========================================================================================|
    |  No running processes found                                                             |
    +-----------------------------------------------------------------------------------------+

# GPUリストの表示

    # nvidia-smi --list-gpus 
    GPU 0: NVIDIA A100-PCIE-40GB (UUID: GPU-3fb142e7-7102-c7ac-bd5a-fc829e11ed84)
    GPU 1: NVIDIA A100-PCIE-40GB (UUID: GPU-14ac2834-0d81-0508-69eb-e25264c08d21)

# 非NVLINK構成の確認
    # nvidia-smi nvlink -s
    GPU 0: NVIDIA A100-PCIE-40GB (UUID: GPU-3fb142e7-7102-c7ac-bd5a-fc829e11ed84)
    NVML: Unable to retrieve NVLink information as all links are inActive
    GPU 1: NVIDIA A100-PCIE-40GB (UUID: GPU-14ac2834-0d81-0508-69eb-e25264c08d21)
    NVML: Unable to retrieve NVLink information as all links are inActive

# ECCが有効になっているか、ECCでエラーがないかの確認
    # nvidia-smi -q -d ECC
        
    ==============NVSMI LOG==============
    
    Timestamp                                 : Thu Jun 13 17:51:27 2024
    Driver Version                            : 555.42.02
    CUDA Version                              : 12.5
    
    Attached GPUs                             : 2
    GPU 00000000:12:00.0
        ECC Mode
            Current                           : Enabled
            Pending                           : Enabled
        ECC Errors
            Volatile
                SRAM Correctable              : 0
                SRAM Uncorrectable Parity     : 0
                SRAM Uncorrectable SEC-DED    : 0
                DRAM Correctable              : 0
                DRAM Uncorrectable            : 0
            Aggregate
                SRAM Correctable              : 0
                SRAM Uncorrectable Parity     : 0
                SRAM Uncorrectable SEC-DED    : 0
                DRAM Correctable              : 0
                DRAM Uncorrectable            : 0
                SRAM Threshold Exceeded       : No
            Aggregate Uncorrectable SRAM Sources
                SRAM L2                       : 0
                SRAM SM                       : 0
                SRAM Microcontroller          : 0
                SRAM PCIE                     : 0
                SRAM Other                    : 0
    
    GPU 00000000:8A:00.0
        ECC Mode
            Current                           : Enabled
            Pending                           : Enabled
        ECC Errors
            Volatile
                SRAM Correctable              : 0
                SRAM Uncorrectable Parity     : 0
                SRAM Uncorrectable SEC-DED    : 0
                DRAM Correctable              : 0
                DRAM Uncorrectable            : 0
            Aggregate
                SRAM Correctable              : 0
                SRAM Uncorrectable Parity     : 0
                SRAM Uncorrectable SEC-DED    : 0
                DRAM Correctable              : 0
                DRAM Uncorrectable            : 0
                SRAM Threshold Exceeded       : No
            Aggregate Uncorrectable SRAM Sources
                SRAM L2                       : 0
                SRAM SM                       : 0
                SRAM Microcontroller          : 0
                SRAM PCIE                     : 0
                SRAM Other                    : 0






    
