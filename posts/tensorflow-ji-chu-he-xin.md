---
title: 'Tensorflow2 核心基础'
date: 2021-02-09 11:28:07
tags: [Tensorflow]
published: true
hideInList: false
feature: 
isTop: false
---
[zhousanfu/Tensorflow_Demo](https://github.com/zhousanfu/Tensorflow_Demo)

一、安装

- [Mac M1芯片安装教程](https://towardsdatascience.com/tensorflow-2-4-on-apple-silicon-m1-installation-under-conda-environment-ba6de962b3b8)

```sql
# M1芯片配适版-Miniconda
wget -c "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh"
sh Miniforge3-MacOSX-arm64.sh

# Install specific pip version and some other base packages
pip install --force pip==20.2.4 wheel setuptools cached-property six
# Install all the packages provided by Apple but TensorFlow
pip install --upgrade --no-dependencies --force numpy-1.18.5-cp38-cp38-macosx_11_0_arm64.whl grpcio-1.33.2-cp38-cp38-macosx_11_0_arm64.whl h5py-2.10.0-cp38-cp38-macosx_11_0_arm64.whl tensorflow_addons-0.11.2+mlcompute-cp38-cp38-macosx_11_0_arm64.whl
# Install additional packages
pip install absl-py astunparse flatbuffers gast google_pasta keras_preprocessing opt_einsum protobuf tensorflow_estimator termcolor typing_extensions wrapt wheel tensorboard typeguard
# Install TensorFlow
pip install --upgrade --force --no-dependencies tensorflow_macos-0.1a1-cp38-cp38-macosx_11_0_arm64.whl
```