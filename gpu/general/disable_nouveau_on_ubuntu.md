# nouveauドライバの無効化
- GPUドライバーをインストールする前に必要な作業
- OS環境：Ubuntu 22.04.4 LTS

      # vi /etc/modprobe.d/blacklist-nouveau.conf 
      blacklist nouveau
      options nouveau modeset=0
      
      # update-initramfs -u
      # reboot
