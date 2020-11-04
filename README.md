# micropython-lib


1.0 micropython的构建安装

​        本示例使用的上位机为 ubuntu 20 @ x86，MCU使用的是STM32F411EU，开发板使用的是WeAct，移植部分使用的是WeAct官方提供源代码，miropython框架采用原生框架。

1.1 准备工作

```bash
#install tool-chains on ubuntu 20.4，安装编译工具
sudo apt-get install gcc
sudo apt-get install gcc-arm-none-eabi
sudo apt install make
sudo apt-get install git
# pull micropython repository from github，从github下载micropython源代码
git clone https://github.com/micropython/micropython    
#也可以通过下一条命令clone进行加速
#git clone https://hub.fastgit.org/micropython/micropython
cd micropython
#设置micropython源代码的根目录
export MPY_HOME=$PWD
#通过submodule指令注册依赖的模块
git submodule update --init
#使用下一条命令，可以将子模块的仓库地址批量修改成fastgit进行加速
#find ${MPY_HOME} -name ".gitmodules"|xargs sed -i "s/https:\/\/github.com/http:\/\/192.168.100.22:3000/g"
```

1.2 预先编译交叉编译工具

```bash
cd ${MPY_HOME}/mpy-cross
make
```

1.3 选定开发板类型

```bash
cd ${MPY_HOME}/ports/stm32/boards
#单独拷贝一份模板出来，避免污染原有代码
cp -r STM32F4DISC MYBOARD
#最好使用开发板的官方移植代码，或者已有调整好的移植代码
#git clone https://github.com/mcauser/WEACT_F411CEU6.git
```

1.4 构建指定开发板的固件

```bash
cd ${MPY_HOME}/ports/stm32
make BOARD=MYBOARD
#使用github上的移植方案
#make BOARD=WEACT_F411CEU6
#构建的固件在${MPY_HOME}/ports/stm32/build-MYBOARD下
ls -l ${MPY_HOME}/ports/stm32/build-MYBOARD/firmware.dfu
ls -l ${MPY_HOME}/ports/stm32/build-MYBOARD/firmware.hex
```

2.0固件烧录

​      常见的烧录方式有openocd，dfu，st-linker，其中dfu方式需要先通过跳线，将开发板值为ISP状态，ISP是STM32芯片内置的一段处理逻辑，进入后等待固件下载，运行态时则从0x08000000地址开始执行程序。

2.1 安装烧录工具

```bash
sudo apt install openocd
sudo apt install dfu-util
```

2.2 执行烧录

```bash
cd ${MPY_HOME}/ports/stm32
#deploy with dfu-util
#man dfu-util 查看dfu-util工具的帮助文档
sudo dfu-util -e
sudo dfu-util -l
#根据 sudo dfu-util -l 查找结果，确定-d参数的值，如0483:df11
sudo dfu-util -a 0 -d 0483:df11 -D build-MYBOARD/firmware.dfu

#or deploy with st-linker util
#export STLINK_DEVICE="002:0035"  is not nessisury if st-flash can found st-linker device
make BOARD=MYBOARD deploy-stlink

#or deploy  with openocd
make BOARD=MYBOARD deploy-openocd
```

3.0 运行调试

​      烧录固件后接入USB TYPE-C重启，即可在上位机上看到两个设备，一个是U盘，一个是串口，通过U盘可以直接修改man.py文件，添加自己的逻辑，通过串口可以监控输出，也可以进入python命令行窗口进行交互式操作。

```bash
#在上位机安装串口通讯工具picocom
sudo apt-get install picocom
#单片机在运行态可以用picocom连接运行micropython命令，比如使用USB->TTL工具时
sudo picocom -b 9600 /dev/ttyUSB0
#WeAct的STM32F411CEU开发板的USB TYPE-C口在安装好固件以后可以在上位机虚拟机出串口，linux下一般为/dev/ttyACM0,windows下可能为COM3之类的
sudo picocom -b 9600 /dev/ttyACM0
#重新修改main.py后保存，在终端输入 ctl + d 组合键即可重新加载运行
```

4.0 注意事项

- STM32开发板在接入电源时注意不要正负极反接，否则可能导致开发板烧坏。
- 首次 烧入固件后开发板没有反应，需要重启后才能识别U盘和串口（串口一个从USB TYPE-C虚拟出来，一个从PA9/PA10引入）