# NVIDIA QUADRO P6000のドライバーインストール
- サーバー： HPE ProLiant DL385p Gen8
- GPU: NVIDIA QUADRO P6000
- OS: AlmaLinux 9.4
- カーネル： 5.14.0-427.22.1.el9_4.x86_64

      # dnf update -y
      # reboot
      # uname -r
      5.14.0-427.22.1.el9_4.x86_64
      
      # lspci -v | grep -i nvidia
      44:00.0 VGA compatible controller: NVIDIA Corporation GP102GL [Quadro P6000] ...
      
      # dnf groupinstall -y "Development Tools"
      # dnf install -y acpid automake bzip2 elfutils-libelf-devel gcc gcc-c++ libglvnd-op
      engl libglvnd-glx libglvnd-devel make pciutils perl pkgconfig tar

      # vi /etc/modprobe.d/blacklist-nouveau.conf
      blacklist nouveau
      options nouveau modeset=0
      
      # dracut -f
      # reboot
      # lsmod | grep nouveau
      # pushd /etc/yum.repos.d/
      # curl -O https://developer.download.nvidia.com/compute/cuda/repos/rhel9/x86_64/cuda-rhel9.repo
      # popd
      # dnf search cuda
      # dnf install -y epel-release
      # dnf install -y dkms
      # dnf module install -y nvidia-driver:latest-dkms
      # dnf install -y cuda-12-5
      # reboot
      # lsmod | grep -i nvidia
      ...
      # nvidia-smi
      ...

[KOGA MASAZUMI](https://www.amazon.co.jp/stores/%E5%8F%A4%E8%B3%80%E6%94%BF%E7%B4%94/author/B0725M9C6T) ([@masazumi_koga](https://x.com/masazumi_koga))
