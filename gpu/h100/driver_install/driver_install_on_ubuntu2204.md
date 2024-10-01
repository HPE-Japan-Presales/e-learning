# H100 x2 (NVLINK構成) でのdmesg実行時のエラーへの対処

- 参考情報：　https://forums.developer.nvidia.com/t/nvrm-this-pci-i-o-region-assigned-to-your-nvidia-device-is-invalid/229899
- "pci=realloc=off"は、PCIブリッジリソースの再割り当てを無効にする設定。BIOSによって割り当てられたリソースをそのまま使用し、カーネルが再割り当てを試みない
- 現時点で、H100以外のGPUでは、 "pci=realloc=off"が不要だが、場合によっては、このパラメータが必要なこともある可能性あり
 
# dmesgのエラーメッセージ
    # dmesg |grep -i nvidia
    ...
    NVRM: This PCI I/O region assigned to your NVIDIA device is invalid
    ...

# 対処
    # vi /etc/default/grub
    ...
    GRUB_CMDLINE_LINUX_DEFAULT="pci=realloc=off"
    ...

    # update-grub
    # reboot

[KOGA MASAZUMI](https://www.amazon.co.jp/stores/%E5%8F%A4%E8%B3%80%E6%94%BF%E7%B4%94/author/B0725M9C6T) ([@masazumi_koga](https://x.com/masazumi_koga))
