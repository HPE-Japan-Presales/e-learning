# nouveauドライバの無効化(Ubuntu 22.04.4 LTS)
- GPUドライバーをインストールする前に必要な作業
- OS環境：Ubuntu 22.04.4 LTS

      # vi /etc/modprobe.d/blacklist-nouveau.conf 
      blacklist nouveau
      options nouveau modeset=0
      
      # update-initramfs -u
      # reboot

[KOGA MASAZUMI](https://www.amazon.co.jp/stores/%E5%8F%A4%E8%B3%80%E6%94%BF%E7%B4%94/author/B0725M9C6T) ([@masazumi_koga](https://x.com/masazumi_koga))
