[toc]
# 树莓派环境搭建
烧录工具烧录并根据以下链接参考，配网编译emll库。

交叉编译工具链配置https://blog.csdn.net/gogo0707/article/details/124367587
工具链下载https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads#panel2a
树莓派配网 https://zhuanlan.zhihu.com/p/435714438
指令查询网站 https://developer.arm.com/search#f[navigationhierarchiestopics]=Intrinsics
sudo apt-get install gcc-arm-linux-gnueabihf
cmake .. -DCMAKE_INSTALL_PREFIX=../install -DCMAKE_C_COMPILER=/usr/bin/arm-linux-gnueabihf-gcc -DEML_ARMV7A=ON
make