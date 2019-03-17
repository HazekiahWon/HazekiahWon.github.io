---
title: "Dealing with conflicting pytorch and tensorflow in conda"
categories:
  - debug

tags:
  - conda
  - pytorch
  - tensorflow
  - cudnn
author_profile: true
toc_sticky: true
---

<a name="3738bd69"></a>
# Problem Description
With Pytorch 1.0.1 installed in conda, I want to further install tensorflow-gpu by `conda install tensorflow-gpu`. After the installation, pytorch when calling cudnn utilities will throw exception:  <br />`cuDNN version incompatability`.<br />![cudnn mismatch.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552800606453-2d0d3ed2-3876-4e16-918a-b4b740170f2c.png#align=left&display=inline&height=350&name=cudnn%20mismatch.PNG&originHeight=595&originWidth=1269&size=53032&status=done&width=746)

<a name="Traceback"></a>
# Traceback
According to [this link]([https://github.com/pytorch/pytorch/issues/2286](https://github.com/pytorch/pytorch/issues/2286)), pytorch has maintain a cudnn version in its library. However, another version of cudnn is installed with tensorflow library, resolved by conda. This version is 7301 while the self-maintained pytorch version is 7401.

![condatf.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552801279152-3628eceb-ab9f-409f-a22f-95a602ecfbf0.png#align=left&display=inline&height=331&name=condatf.PNG&originHeight=460&originWidth=1036&size=39339&status=done&width=746)<br />

![normtch.PNG](https://cdn.nlark.com/yuque/0/2019/png/278890/1552801332835-326b0e37-0c9e-4b41-a52c-f8d3fdd6f322.png#align=left&display=inline&height=110&name=normtch.PNG&originHeight=110&originWidth=491&size=5420&status=done&width=491)

<a name="Solution"></a>
# Solution
`pip install tensorflow-gpu` instead of `conda install tensorflow-gpu`, because pip will not by itself install a version of cudnn.

