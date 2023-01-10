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

### 3、##

​		1、**##**本质是一个“胶水运算”（连接字符串的作用）用于把参数宏中的“形参”与其它没有天然分割的内容粘连在一起，例如：

```c
#源代码
#define def_u32_array(__name, __size)     uint32_t array_##__name[__size];`
#实际使用
def_u32_array(sample_buffer, 64)
#函数展开
uint32_t array_sample_buffer[64];
```

​		2、特殊用法

```c

/* 1、这里定义了一个宏"safe_atom_code()"，在括号内，无论你填写任何内容，都会被无条件的放置到“__VA_ARGS__”所在的位置，你可以认为括号里的“...”实际上就是对应"__VA_ARGS__"。 */
#define safe_atom_code(...)                          \
        {                                            \
            uint32_t int_flag = __disable_irq();     \
            __VA_ARGS__                              \
            __set_PRIMASK(int_flag);                 \
        }
/* 代码确保在向寄存器GCLD_PORT->DAT写入数据时不会被其它中断打断。*/
static __inline void wr_dat (uint_fast16_t dat) 
{
    safe_atom_code(
        LCD_CS(0);
        GLCD_PORT->DAT = (dat >>   8);   /* Write D8..D15 */
        GLCD_PORT->DAT = (dat & 0xFF);   /* Write D0..D7 */
        LCD_CS(1);
    )
}


/* 2、当我们使用参数宏的时候在括号里不填写任何内容，最终会展开为仅有默认值的情况： */
#define EXAMPLE(...)     ( 默认值 ,##__VA_ARGS__)
/* 使用时 */
EXAMPLE();
/* 展开时 */
( 默认值 )
/* 当我们提供任意有效值时，则会被展开为逗号表达式 */
EXAMPLE(我们提供的值);
/* 展开时 */
( 默认值, 我们提供的值 )/*  */
/*********************************************/
#define XXXX_INIT(...)    xxxx_init((NULL,##__VA_ARGS__))
typedef struct xxxx_cfg_t {
    ...
} xxxx_cfg_t;
int xxxx_init(xxxx_cfg_t *cfg_ptr);

typedef struct msg_t msg_t;
struct {
    uint16_t msg;
    uint16_t mask;
    int (*handler)(msg_t *msg_ptr);
} msg_t;

#define def_msg_map(__name, ...)                            \
    const msg_t __name[] = {__VA_ARGS__};
    
#define add_msg(__msg, __handler, ...)                      \
    {                                                       \
        .msg = (__msg),                                     \
        .handler = &(__handler),                            \
        .msk = (0xFFFF, ##__VA_ARGS__),                     \
    }

/*! \note 高字节表示操作的类别:
          比如0x00表示控制类，0x01表示WRITE，0x02表示READ
 */
enum {
    SIGN_UP      = 0x0001,
    WRITE_MEM    = 0x0100,
    WRITE_SRAM   = 0x0101,
    WRITE_FLASH  = 0x0102,
    WRITE_EEPROM = 0x0103,
    
    READ_MEM     = 0x0200,
    READ_SRAM    = 0x0201,
    READ_FLASH   = 0x0202,
    READ_EEPROM  = 0x0203,
};
extern int iap_sign_up_handler(msg_t *msg_ptr);
extern int iap_write_mem(msg_t *msg_ptr);
extern int iap_read_mem(msg_t *msg_ptr);
def_msg_map( iap_message_map
    /* 严格的将 SIGN_UP 映射到 对应的处理函数中 */
    add_msg( SIGN_UP,   iap_sign_up_handler ),
    /* 批量处理所有的WRITE操作，使用掩码进行过滤*/
    add_msg( WRITE_MEM, iap_write_mem,       0xFF00 ), 
    /* 批量处理所有的READ操作，使用掩码进行过滤 */
    add_msg( READ_MEM,  iap_read_mem,        0xFF00 ),
)
```







### 4、逗号表达式

