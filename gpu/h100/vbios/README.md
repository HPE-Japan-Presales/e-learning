# GPUのVBIOS/CECの更新方法

H100のVBIOS/CEC情報：https://support.hpe.com/connect/s/softwaredetails?language=en_US&collectionId=MTX-163a30752dd343d2

- HPE提供のH100のVBIOS/CECの更新は、RHEL上で実施
- RPMパッケージをインストール後、スクリプトの実行
  
# GPU VBIOSのインストール

        # rpm –ivh NVIDIA_H100_PCIe_GPU-96.00.74.00.1C-1-0.x86_64.rpm
        # cd /usr/local/bin/NVIDIA_H100_PCIe_GPU-96.00.74.00.1C-1/  
        # chmod +x NVIDIA_H100_PCIe_GPU-96.00.74.00.1C.scexe
        # init 3
        # rmmod nvidia
        # ./NVIDIA_H100_PCIe_GPU-96.00.74.00.1C.scexe

# CECファームウェアのインストール

        # rpm –ivh NVIDIA_H100_PCIe_CEC-00.02.0134.0000-1-0.x86_64.rpm
        # cd /usr/local/bin/NVIDIA_H100_PCIe_CEC-00.02.0134.0000-1/
        # chmod +x NVIDIA_H100_PCIe_CEC-00.02.0134.0000.scexe
        # init 3
        # rmmod nvidia

# スキャンの実施

        # bash ./NVIDIA_H100_PCIe_CEC-00.02.0134.0000.scexe -s
        HPE Firmware- CEC BIOS ( NVIDIA) flashing Utility Version 1.0 - output files saved in *.log
        Gathering System information(Check system.log).......................Complete!
                NVIDIA Firmware Information for CEC Device Devices:
                Graphics Device      (10DE,2331,10DE,1626) S:00,B:C2,D:00,F:00
                CEC Firmware Version : 00.02.0025.0000_n00
                    - Active  Version : 00.02.0025.0000_n00
                    - Pending Version : N/A
                    - Active  Version : 03000001
                    - Pending Version : N/A
                Graphics Device      (10DE,2331,10DE,1626) S:00,B:D5,D:00,F:00
                CEC Firmware Version : 00.02.0114.0000_n00
                    - Active  Version : 00.02.0114.0000_n00
                    - Pending Version : N/A
                    - Active  Version : 05e00001
                    - Pending Version : N/A

        Gathering Nvidia Device information(Check gpu.log).......................Complete!
        Gathering Nvidia Device information for CEC Device Firmware Match. Please wait this may take few minutes....
         
         <0> CEC Device              
             |__ Graphics Device      (10DE,2331,10DE,1626) 00:C2:00.0
         <1> CEC Device              
             |__ Graphics Device      (10DE,2331,10DE,1626) 00:D5:00.0
         
        Complete!

# CECの更新
        # ./NVIDIA_H100_PCIe_CEC-00.02.0134.0000.scexe
 

[KOGA MASAZUMI](https://www.amazon.co.jp/stores/%E5%8F%A4%E8%B3%80%E6%94%BF%E7%B4%94/author/B0725M9C6T) ([@masazumi_koga](https://x.com/masazumi_koga))

