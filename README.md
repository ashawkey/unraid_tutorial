# Unraid 搭建指南

> 整理时间2021年11月，仅供参考，请注意时效性。



### Abstract

本文描述如何心血来潮想在大学宿舍环境下搭建一台轻量NAS服务器，经历了平衡**价位、性能、功耗、噪音、空间**等反复折磨后总算勉勉强强差强人意运行起来的折腾历程。

考虑到群晖的硬件性价比感人、黑群晖对强迫症不友好、NAS机箱选择较少且价格普遍感人等因素，最终选择的搭配是普通ITX机箱+Unraid OS。目前流传的众多Unraid教程多多少少有些重要的部分已经失效，也不太适合校园网下的配置，故在基本稳定运行后总结了这篇指南，希望能够帮到有类似（奇怪）需求的人。



### Overview

* [硬件配置](./hardware.md)

  仅供参考，如果已经有目标了，请跳过。

* [Unraid配置](./unraid.md)

  基础的配置推荐官网文档以及各种视频教程。

* [2021年科学访问社区应用](./proxy.md)

  风紧墙高。

* [插件/Docker应用](./docker_app.md)

  一些推荐。

* [IPv6](./ipv6.md)

  适用于使用DHCPv6的校园网环境，没错是NAT66。

* [外部访问](./remote_access.md)

  暂时咕了。
  
  

### Acknowledgment

感谢所有曾经公开发布过Unraid教程的人们！

* https://post.smzdm.com/p/ad2dngvn/