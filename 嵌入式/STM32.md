# 引脚基础  知识
## 引脚编号
找芯片上面圆圈 最近的 逆时针转 1
## 特殊引脚  
**VDD** 供电 接3.3V  
**VSS** 接地0V  
**NRST** 复位(接复位引脚)  
**VBAT** 备用电源(纽扣电池)  
**BOOT0** 启用模式选择  
## 普通引脚
分组:GPIOA GPIOB GPIOC GPIOD(STM32是大脑 GPIO是神经)  
数字编号:1-16  
不一定连续
# GPIO工作模式
共八种 每一个引脚可以选择其中之一  
## 输出  
### 类型
**通用输出推挽**  
**通用输出开漏**  
**复用输出推挽**  
**复用输出开漏**  

#### 推挽/开漏
**推挽**  
MOS管交替导通输入  
输入1:P-MOS导通 输出高电压  
输入0:M-MOS导通 输出高电压  

**开漏**  
P-MOS始终断开  
输入0:输出低电压 M-MOS联通  
输入1:不输出电压 高阻抗(断路)  

#### 通用/复用:  
通用:CPU直接控制0/1  
复用:

### IO最大输出速度
低速:2MHZ  
中速:10MHZ  
高速:50MHZ  
选取满足要求的最小值(速度越高 耗电增加 EMI问题) 

### 亮灯实验
```C
#include "stm32f10x.h"
#include "delay.h"
//这个程序用于做在引脚PC13的板LED灯亮灯测试
int main(void)
{
	//1. 启用GPIOC用时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC,ENABLE);
	//2. 初始化IO引脚:
	GPIO_InitTypeDef obj = { 0 };
	
	obj.GPIO_Pin = GPIO_Pin_13;//13号引脚
	obj.GPIO_Mode = GPIO_Mode_Out_PP;//通用输出推挽
	obj.GPIO_Speed = GPIO_Speed_2MHz;//IO最大输出速度 低速:2MHZ 
	
	
	GPIO_Init(GPIOC,&obj);
	//3. 向输出数据寄存器写一bit
	while(1)
	{
		GPIO_WriteBit(GPIOC,GPIO_Pin_13,Bit_RESET);//输出低电平(0),LED点亮
		Delay(500);
		GPIO_WriteBit(GPIOC,GPIO_Pin_13,Bit_SET);//输出高电平(1),LED灭
		Delay(500);
	}
}
```

## 输入   
**输入上拉**  
**输入下拉**  
**输入浮空**  
**模拟模式**  
