# Installing CUDA
## step 1 Download
All archived version of CUDA is provided [here](https://developer.nvidia.com/cuda-toolkit-archive). Choose one, for example, [cuda 9.0](https://developer.nvidia.com/cuda-90-download-archive).
![]({{ site.url }}{{ site.baseurl }}/assets/images/20190117/cuda_ins.PNG)

Remember to choose the debian file. It's less prone to failure.
## step 2 Install
`cd` your download path. It won't matter where you install the debian file, so no need to `mv` the file.

Type the following commands.
```bash
sudo dpkg -i cuda-repo-ubuntu1604-9-0-local_****.deb
sudo apt-key add /var/cuda-repo-<version>/7fa2af80.pub`
sudo apt-get update
sudo apt-get install cuda
```
**Note**

For the first line, just `tab` to complete your specific filename. `****` here means version number. In my case, it's `9.0.176-1_amd64`.

For the second line, you can find it after the execution of the first command line.

![]({{ site.url }}{{ site.baseurl }}/assets/images/20190117/cuda_key.PNG)

In my case, it's `/var/cuda-repo-9-0-local/7fa2af80.pub`.

You may find your installation at `/usr/local`, where `cuda` folder has a symbolic link to the nearby `cuda-9.0` folder.

# Installing CudNN
CudNN normally requires you to log in before downloading, but [this link](https://developer.nvidia.com/rdp/cudnn-archive) seems to save the time.

Choose one that matches your installed CUDA. For example, I installed cuda-9.0, so I install this version of cudnn.

![]({{ site.url }}{{ site.baseurl }}/assets/images/20190117/cudnn.PNG)

I would still recommend debian file as it's really convenient. Here is the [official installation guide](http://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html) for cudnn.

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MjMzNjM5ODAsLTQ5ODAxNzMwMywtMT
UxNzc2ODAyLDIwMDYyMTcwNzhdfQ==
-->