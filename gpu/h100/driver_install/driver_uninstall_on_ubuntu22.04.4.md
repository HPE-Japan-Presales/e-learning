# GPUドライバをアンインストールする手順

- ハードウェア： HPE ProLiant DL380a Gen11
- GPU： NVIDIA H100
- OS: Ubuntu Server 22.04.4 LTS

      # apt-get --purge remove nvidia-*
      # apt-get --purge remove cuda-*
      # apt-get autoremove
      # apt-get autoclean

[KOGA MASAZUMI](https://www.amazon.co.jp/stores/%E5%8F%A4%E8%B3%80%E6%94%BF%E7%B4%94/author/B0725M9C6T) ([@masazumi_koga](https://x.com/masazumi_koga))
