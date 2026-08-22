# 学习刷机
https://www.bilibili.com/video/BV1BY4y1H7Mc/?spm_id_from=333.337.search-card.all.click&vd_source=dc6ca277dcaf6155b5329334561a1f4a
## 安卓分区
```
|_Boot
|   |__Kernel 操作系统内核
|   |__Ramdisk 虚拟内存
|   (出问题:卡在启动时的第一屏)
|_System
|   |__操作系统
|   |__系统软件
|   (出问题:卡在启动时的第二屏)
|   (Vendor:手机厂商的固件 可能独立放 可能放在System内)
|_Data
|   (如题 数据)
|_Cache
|   (如题 缓存)
|_Recovery
   (不一定存在 可能是备用分区之类的 Recovery就被放在Boot里了)
   (简易Linux操作系统 类似PE)
```


## 手机端需要做的  
1. 启用开发者选项(点版本号多次)   
2. 开USB调试  
(在开发者页面)  
**ABD**:安卓调试桥(Android Debug Bridege) 命令行工具
3. OEM解锁  
(在开发者页面)  

## 电脑端需要做的
```  
Android SDK（ADB和Fastboot电脑端工具）：https://developer.android.google.cn/studio/releases/platform-tools?hl=zh-cn
```
装SDK 装好之后再有exe文件的文件夹里打开cmd

## BootLoader(线刷)
相当于电脑的BIOS 开启时最先启动 初始化硬件 引导操作系统内核(默认是上锁的)
根据型号 关机 之后 搜手机型号对应的开fastboot方式  
<small>例如HONOR是关机后长按音量-</small> 

(因为HORNOR 9X走不通 暂时放弃)