# 多点视频汇集直播

本项目计划旨在解决常见“一对多直播”的逆问题：“多对一”直播。

## 目标

1. 鲁棒性：单个上行视频流的故障不能导致总体输出故障。
1. 兼容性：上行终端配置简单，甚至不需预装软件（基于HTML5）。
1. 合理的延迟：限于上行软件的简陋，无法追求超低延迟，但延迟应控制在分钟级以下（十秒级别）
1. 开箱即用：服务器配置简单。

## 架构设想

以下是尚未实现（但已进行可行性调研）的项目结构。

1. 核心服务器 Core
接收多路视频数据，进行处理合并，生成合成后的数据流。

初定数据输入输出皆使用rtmp，使用nginx处理（只负责中转流量，不进行计算）。使用多个ffmpeg进程实例生成不同的合并后画面；修改画面配置后重启进程。

nginx提供两类rtmp接口，一类用于接收并转发上行原始视频流，一类用于作为本地视频管道；通过简单的ffmpeg命令将上行流指向对应的管道，再从对应的管道获取画面生成下行流，可以避免在计算中打断上下行流量。

提供网页控制台更改ffmpeg配置，以更改输出流画面；初定使用node.js程序生成命令行参数、调用，并提供网页界面。

2. 上行服务器 Edge
接收客户端的上行数据，提供基本的代理功能（进行必要的压缩）。同时，用 Websocket 接收 HTML5 客户端的上行数据（进行必要的转码）。在核心服务器上可同时运行一份上行服务器的实例，用于接受直接连接的客户端。

由于涉及 HTML5 采集视频，需使用 SSL ，考虑使用nginx进行傻瓜化SSL代理。

3. 下行隧道服务器 Proxy
无甚实际价值，当下行客户端到核心服务器网络不通畅时使用。

## 服务器需求

边缘服务器需要进行对等的流量转发和转码工作，带宽需求不定，有轻度编码运算。使用有富余带宽的小型VPS即可。

核心服务器涉及接收流并合并转码，带宽需求大于两倍最终画面码率，有中等强度的编码运算——有 O(N) 的解码和 O(1) 的编码。鉴于此，保证核心服务器有下行客户端两到三倍的带宽和两到三倍的CPU算力即可，有大带宽的中高端VPS即可胜任。

核心服务器可以运行在本地，有足够好的互联网下行环境即可。

## 依赖和版权

本项目依赖许多开源项目（的代码或二进制程序），故采用GPLv3协议发布所有源码。各开源项目的版权声明详见各项目主页。
1. ffmpeg w/ plugins
1. node.js w/ socket.io
1. nginx w/ rtmp

## 时间表

此人希望能在10个月内完成所有编码（在2016年10月前），作为业余项目，时间应该是充足的。欢迎fork并将已有的子项目改作他用。

项目被拆分成了几个子项目，难度依次递增

1. 下行服务器（配合客户端实现自适应码率下行视频流，例如配置HLS）
1. 上行服务器（websocket+MediaStreamRecorder.js）
1. 核心服务器（接收rtmp视频流并动态分配，）
