# Installing CUDA
## step 1 Download
All archived version of CUDA is provided [here](https://developer.nvidia.com/cuda-toolkit-archive). Choose one, for example, [cuda 9.0](https://developer.nvidia.com/cuda-90-download-archive).
![]({{ site.url }}{{ site.baseurl }}/assets/images/20190117/cuda_ins.PNG)

Remember to choose the debian file. It's less prone to failure.

Type the following commands.
```bash
sudo dpkg -i cuda-repo-ubuntu1604-9-0-local_9.0.176-1_amd64.deb
sudo apt-key add /var/cuda-repo-<version>/7fa2af80.pub`
sudo apt-get update
sudo apt-get install cuda
```
You may find your installation at `/usr/local`, where `
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MDE5NTgyMywtMTUxNzc2ODAyLDIwMD
YyMTcwNzhdfQ==
-->