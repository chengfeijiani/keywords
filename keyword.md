# 关键字

### 1、__attribute__ 

​		1、__attribute__( at(绝对地址) )的作用分两个，一个是绝对定位到Flash，另个一是绝对定位到RAM。

​		2、C语言中的关键字__attribute__ ，直接用的是绝对定位

```c
#ifdef ARMC_AC6
uint16_t testValue __attribute__((section(".ARM.__at_0x64000000")));
#else
/*绝对定位方式访问SRAM,这种方式必须定义成全局变量*/
uint8_t testValue __attribute__((at(ADDR_SRAM_EXT)));
#endif
```

​		3、**定位到flash中**，一般用于固化的信息，如出厂设置的参数，上位机配置的参数，ID卡的ID号，flash标记等等

```c
const u16 gFlashDefValue[512] __attribute__((at(0x0800F000))) = {0x1111,0x1111,0x1111,0x0111,0x0111,0x0111};//定位在flash中,其他flash补充为00
const u16 gflashdata__attribute__((at(0x0800F000))) = 0xFFFF;
```

​		4、**定位到RAM中**，一般用于数据量比较大的缓存，如串口的接收缓存，再就是某个位置的特定变量

```c
u8 USART2_RX_BUF[USART2_REC_LEN] __attribute__ ((at(0X20001000)));//接收缓冲,最大USART_REC_LEN个字节,起始地址为0X20001000.
```

**注意：**

​		**1、绝对定位不能在函数中定义，局部变量是定义在栈区的，栈区由MDK自动分配、释放，不能定义为绝对地址，只能放在函数外定义。**

​		**2、定义的长度不能超过栈或Flash的大小，否则，造成栈、Flash溢出**

### 2、inline

​		1、在程序中，对于短小而频繁调用的函数，可以申明成**inline**函数，编译器会会自动将该函数展开；复杂的函数时不会被展开的。

![img](https://img-blog.csdnimg.cn/img_convert/a9b4717c3f6f6c136dabfa5c686044ba.png)

