# Installing CUDA
## step 1 Download
All archived version of CUDA is provided [here](https://developer.nvidia.com/cuda-toolkit-archive). Choose one, for example, [cuda 9.0](https://developer.nvidia.com/cuda-90-download-archive).
![]({{ site.url }}{{ site.baseurl }}/assets/images/20190117/cuda_ins.PNG)

Remember to choose the debian file. It's less prone to failure.
## step 2 Install
`cd` your download path. It won't matter where you install the debian file, so no need to `mv` the file.

Type the following commands.
```bash
sudo dpkg -i cuda-repo-ubuntu1604-9-0-local_9.0.176-1_amd64.deb
sudo apt-key add /var/cuda-repo-<version>/7fa2af80.pub`
sudo apt-get update
sudo apt-get install cuda
```
You may find your installation at `/usr/local`, where `cuda` folder has a symbolic link to the nearby `cuda-9.0` folder.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwMzA2NTY0MzEsLTE1MTc3NjgwMiwyMD
A2MjE3MDc4XX0=
-->