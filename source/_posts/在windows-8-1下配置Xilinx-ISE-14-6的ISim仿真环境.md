title: 在windows 8.1下配置Xilinx ISE 14.6的ISim仿真环境
date: 2014-05-07 22:25:11
tags: windows
---
windows 8.1下的Xilinx ISE 14.6无法成功地调用自带的ISim模拟器，报错结果如下：
``` bash
$ Time Resolution for simulation is 1ps.
$ Waiting for 3 sub-compilation(s) to finish…

$ ERROR:Simulator:861 – Failed to link the design
$ Process “Simulate Behavioral Model” failed
```

经过近一天的摸索，发现原因是ISim调用的是安装Xilinx ISE 14.6自带的MinGW来link的，而Windows 8.1不支持低版本的MinGW。为了能成功调用ISim，需要更新MinGW，默认的MinGW的安装路径是…\Xilinx\14.4\ISE_DS\ISE\gnu\MinGW\5.0.0\nt。新版本的MinGW的下载地址为：http://sourceforge.net/projects/mingw/files/OldFiles/MinGW%205.1.4/MinGW-5.1.4.exe/download?use_mirror=iweb. 下载成功后安装并且将安装内容替换Xilinx ISE 14.6安装后自带的MingGW即可。