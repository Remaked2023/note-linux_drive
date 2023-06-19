# 1、git 的基础

## 1. git init创建并初始化本地仓库

```git
git init 
```

        如果仓库已创建则会重新初始化，删除已提交到本地仓库的文件，但不会删除远程仓库连接等

## 2. 设置用户名和邮箱

```git
git config --global user.name "linhongyi"
git config --global user.email "linhongyi@agenewtech.com"
```

        设置上传远程仓库时的用户和邮箱，不影响使用，作为标识

## 3. 连接远程仓库，并与本地仓库连接

```git
git remote add origin git@...com        # http连接
```

        SSH秘钥连接不知道为什么成功不了，不写了

```git
git pull origin main                    # github默认分支是main 
```

```git
git fetch
git checkout main                        # git本地仓库默认分支是master
                                         # 切换成main和github仓库保持一致
```

## 4. 添加、提交、上传文件

```git
git add filename1 2 3 ...
git commit -m "..."                      # 快速批注并提交
git push origin main                     # 上传到origin仓库main分支
```

# 2、shell编程相关(考核2)

## 2.1 subcam_code_gen.sh

题干

```shell
2.1 shell编程实现，根据已有camera代码自动生成对应的前摄代码，即在芯片型号后增加字母s
(或者大写S)包括文件名和文件内容的修改。
已知：在如下三个文件夹下存放了相关camera代码，
"kernel-4.19/drivers/misc/mediatek/imgsensor/src/common/v1/" 
"vendor/mediatek/proprietary/custom/mt6739/hal/imgsensor/"
"vendor/mediatek/proprietary/custom/mt6739/hal/imgsensor_metadata/"
以"imx338"这颗IC为例，要求执行脚本后，生成对应imx338s的代码，
# ./subcam_code_gen.sh imx338 
自动生成如下文件，并修改了内容：
kernel-4.19/drivers/misc/mediatek/imgsensor/src/common/v1/ (其他目录类似)
|-- imx338s_mipi_raw 
|-- imx338s_eeprom.c ---注意每个文件的内容，假如原来是IMX338的代码应该变成IMX338S 
|-- imx338s_eeprom.h 
|-- imx338smipi_Sensor.c 
|-- imx338smipi_Sensor.h 
`-- Makefile
注意上面上说的三个目录下的代码不一样。
提示： 先拷贝已有文件夹、再重命名文件，最后修改文件内容(注意区分大小写)
```

shell脚本代码

```shell
#!/bin/bash

replaced=$1             # 要修改的文本

while [ -z "$replaced" ]            # 没有参数输入，退出程序
do
    read -p "修改目标为空，请输入要修改的文本:" replaced
done
echo "输入为：$replaced,1s后执行"
sleep 1s
echo "开始修改"

replacing=$replaced"s"        # 在要改的文本后加s

# 小写部分
for filePath in `find . -name "*$replaced*" | tac`        
do
    newPath=`echo $filePath | sed "s@\(.*\)$replaced@\1$replacing@g"`
    sudo mv "$filePath" "$newPath"
done

# 更改内容-小写部分
sed -i "s/$replaced/$replacing/g" `grep "$replaced" -rl .`

# 将变量更改为大写，再进行更改
replaced=`echo $replaced |tr a-z A-Z`
replacing=`echo $replacing | tr a-z A-Z`

for filePath in `find . -name "*$replaced*"`
do
        newPath=`echo $filePath | sed "s@\(.*\)$replaced@\1$replacing@g"`
           sudo mv "$filePath" "$newPath"
done

# 更改内容-大写部分
sed -i "s/$replaced/$replacing/g" `grep "$replaced" -rl .`

echo "修改完成"
```

### 2.1.1 思路

1. 在脚本运行时，如果**忘记带参数**，会导致脚本修改所有文件名，并将路径下所有目录和子目录下所有文件内容清空，**误操作成本过高**（~~再加上自己误操作了3次~~），所以加上了循环判断提醒重新输入，判断依据是替换文本`$replaced`长度是否为0，即[ -z $replaced ]

2. 思路上将整体分为两部分：大写匹配更改部分和小写匹配更改部分，然后在两个部分内分别进行文件名和文件内容的匹配更改，也就是总计4步的更替

3. `find $path -name "*$replaced*" | tac` 在路径下查找文件名包含"$replaced"的文件，然后对它们进行倒序排列，即从深层路径到浅层路径排列的方式

4. `echo $i | sed "s@\(.*\)$replaced@\1$replacing@g"` 对find找到的文件名进行替换输出，`sed "s@\(.*\)$replaced@\1$replacing@g"` s@...@...@...代表替换，@是分隔符，`\(.*\)` 和`\1` 匹配，意思相同，代表以'.'(一个任意字符)开始，忽略后续字符匹配，然后接上`$replaced` 和`$replacing` ，g代表全局替换。`echo $i` 即是最终输出也是输入sed的数据

5. grep -rl，-r表示对目录和子目录下所有文件进行匹配搜索，-l表示只输出文件名；
   
   ```shell
   sed -i "s/replaced/replacing/g" `grep "$replaced" -rl $path`
   ```

       将grep匹配得到的文件名集合作为sed指令的目标文件，进行批量文本替换

### 2.1.2 难点

1. 匹配更改本身需要花费更多时间，需要选中尽量少的目标范围，同时避免更改其他文本

2. 在更改文件名时需要先获取对应文件的路径，修改浅层文件夹名称导致深层文件路径更改 --用tac指令对处理匹配到的文件集合，从深层到浅层进行文件更名处理

3. 在修改时需要区分大小写，如imx338需要修改为imx338s，大写的IMX338需要修改为IMX338S --小写输入更改一次，大小写转化后再更改一次

4. sed、find、grep三者只有sed具有替换功能，但是sed没有足够丰富的查找和匹配能力，需要联合三者进行

5. 字符串修改需要符合linux命令行和shell编程要求

## 2.2 show_lcm_infor.sh

```shell
2.2 shell编程实现，查询工程里面某一个项目的LCM的配置情况，分别列出IC型号，模组厂商，
模组型号、玻璃厂商、分辨率等

这里以"M6101E"项目举例，已知在如下文件里面有如下配置信息：
kernel-4.19/arch/arm/configs/AGN_1250WQ_M6101E_E1_DS3216_Sgo_defconf
ig:CONFIG_CUSTOM_KERNEL_LCM=\
"ICNL9911C_HLT_MIPI4_HDP ILI9881H_HY_MIPI4_HDP ILI9882N_HLT_MIPI4_HDP"
    其中配置了三个LCM，注意双引号里面三个是用空格隔开的，当前的配置命名规则是：
    IC型号_模组厂商_传输通道_分辨率，请分别将其解析出来，显示在终端。

要求执行如下脚本：
# ./show_lcm_infor.sh M6101e
在终端输出显示如下信息：
LCM_NAME is: ICNL9911C_HLT_MIPI4_HDP 
IC is : ICNL9911C 
Module : HLT 
lane is: MIPI4 
resolution: HDP 

LCM_NAME is: ILI9881H_HY_MIPI4_HDP 
IC is : ILI9881H 
Module : HY 
lane is: MIPI4 
resolution: HDP 

LCM_NAME is: ILI9882N_HLT_MIPI4_HDP 
IC is : ILI9882N 
Module : HLT 
lane is: MIPI4 
resolution: HDP

提示：
已知在如下文件有配置内核版本是kernel-4.19
device/agenew/AGN_1250WQ_M6101E_E1_DS3216_Sgo/ProjectConfig.mk:
LINUX_KERNEL_VERSION = kernel-4.19

完成大致思路如下，可以自己根据需要拆分多个步骤：
第一步，先找到"M6101e"项目对应的"ProjectConfig.mk"文件，
解析其中"LINUX_KERNEL_VERSION"的配置信息(等号后面的值)，得到当前使用的内核版本
第二步，到当前内核版本目录下找到"M6101e"项目对应"defconfig"文件，
解析出其中CONFIG_CUSTOM_KERNEL_LCM的值，可以将其保存到临时文件里面或者变量里。
第三步，根据前面得到的值，进一步解析处理，得到LCM_NAME等信息，并显示打印出来
```

```shell
#!/bin/bash
device=$1       # 保存要查找配置的器件型号

while [ -z "$device" ]            # 检测是否带参，防止误操作
do
    read -p "warning: 无带参执行 请输入要查找的器件:" device
done

for devPath in `find . -name "*$device*" -type d`     # 获得对应的器件全称，并搜寻版本号和对应的配置信息
do
    deviceName=$(basename $devPath)    # 切割绝对路径获取器件名称
    configPath=`find $devPath -name "ProjectConfig.mk" -type f`    # 搜索Config文件
    if [ -z "$configPath" ]    # 找不到ProjectConfig.mk，执行下一个循环
    then
        continue
    fi
    #echo "$configPath"

    version=`grep "LINUX_KERNEL_VERSION" $configPath | cut -d ' ' -f3`    # 保存版本信息
    if [ -z "$version" ]
    then
        continue
    fi
    version=`find . -name "$version" -type d`        # 获取对应版本文件夹路径
    #echo $version

    for LCM_Config in `find "$version" -name "*$deviceName*defconfig" ! -name "*debug*" -type f`
    do
            LCM_NAME=`grep "CONFIG_CUSTOM_KERNEL_LCM" $LCM_Config | cut -d '"' -f2`    # 获取所有LCM_NAME
        for LCM in ${LCM_NAME}
        do
                echo "LCM_NAME is: $(echo $LCM)"        # LCM_NAME
            echo "IC is: $(echo $LCM | cut -d '_' -f1)"        # IC型号
            echo "Module : $(echo $LCM | cut -d '_' -f2)"        # Module模组厂商
            echo "lane is: $(echo $LCM | cut -d '_' -f3)"        # lane传输通道
            echo "resolution is: $(echo $LCM | cut -d '_' -f4)"    # resolution分辨率
        done
    done
done

if [ -z "$devPath" ]
then
    echo "没有找到对应器件，请检查输入后重试"
fi
```

### 2.2.1 思路

1. 目标是用输入参数模糊匹配目录下文件夹全称`devPath`（即`basename+$deviceName器件名`），然后进入对应目录寻找配置信息文件`ProjectConfig.mk`，检索匹配出版本信息变量`LINUX_KERNEL_VERSION`，然后再进入对应的版本目录文件，搜寻对应器件包含`LCM`名称的变量`CONFIG_CUSTOM_KERNEL_LCM`，最后分割输出`LCM_NAME`、`IC`、`Module`、`lane`、`resolution`

2. 同样是先做输入检查，确保`$replaced`长度不为0，循环判断，判空就提示要求重新输入

3. 在对应路径`devPath`模糊查找器件文件夹，循环列出能匹配上的文件夹名称，<u>因为模糊搜索可能使得有不止一个结果</u>，而后续<u>确定版本号、获取对应版本配置信息等操作都是建立在只找到单一目标上的</u>，所以需要将列出的所有目标一个一个进行处理输出，于是后续语句放在for循环中
   
   ```shell
   for devPath in `find . -name "*$device*" -type d`
   ```

4. 变量i此时获取的是绝对路径，利用linux系统自带的路径处理方法
   
   ```shell
   deviceName=$(basename $devPath)
   ```
   
   获取寻找到每个文件夹的命名，也就是模糊搜索匹配到的器件全称

5. 在目录中查找文件名为`"ProjectConfig.mk"`的普通文件，找到的绝对路径做值定义变量为`configPath`
   
   ```shell
   configPath=`find $devPath -name "ProjectConfig.mk" -type f`
   ```

6. 获取版本号匹配搜索关键变量名称获得整行，然后在此基础上以`"`作为分隔符分隔字符串取第二区域，存储在`version`变量，然后再通过`version`变量的值查找得到对应的版本文件路径，更新在`version`变量中

7. 在版本对应文件夹下用文件名称匹配的方式匹配同时包含`$deviceName`和`$deconfig`且不包含`$debug`字样的文件，并要求必须是普通文件的格式，以搜索结果为列表进行for循环

8. shell中，字符串str中用空格隔开各段信息则可表示数组，但是没法引用，`${str}`用此方法可列表循环

### 2.2.2 难点

1. 模糊匹配搜索得到的结果可能有多项，搜索时可能会导致变量结构变为数组，后续查找发生不匹配问题        --for循环内执行，将搜索结果分开处理

2. 搜索得到的结果是绝对路径，后续匹配需要更加精准的器件名确保搜索结果唯一，就需要对路径进行处理，得到文件名（器件名）    --`$(basename $devPath)` 路径处理

        Linux运行空间分为内核空间和用户空间。内核空间运行在不同级别，不能直接访问和共享数据。内核为应用层提供调用接口

        驱动程序代码源文件infrared_s3c2410.c 复制到/drivers/char目录

        在/drivers/char目录下修改Kconfig文件增加下列语句

```c
config INFRARED_REMOTE
    tristate "INFRARED DRiver for REMOTE"
    depends on ARCH_S3C64XX || ARCH_S3C2410
    default y
    help
```

在同目录下Makefile添加

```makefile
Obj-$(CONFIG_INFRARED_REMOTE)+=infrared_s3c2410.o
```

        在Linux内核源代码目录，执行make menuconfig 选择[device drivers]->[chaacter devices]，可在最后一行看到新增驱动

        模块运行在内核态，不能使用用户态C库函数中的printf函数，而要使用printk函数打印调试信息

Makefile文件:

```makefile
AR = ar
ARCH    = arm
CC = arm-none-linux-gnueabi-gcc
DEBFLAGS = -O2
obj-m    := smodule.o
KERNELDIR ?= /root/fgi/linux-4.5.2
PWD    := $(shell pwd)
modules:
    $(MAKE)-C $(KERNELDIR)M=$(PWD) LDDINC$(PWD)/../include modules
clean:
    rm -rf *.o *~ core .depend .* .cmd *.ko *.mod.c .tmp_versions
```

uname -r 获取内核版本号

MODULE_PARM(var, type, right) 用于先模块传递命令行参数。参数可以是整型、长整型、字符串等

printk的打印等级设置：

```c
int console_prink[4] = {
    CONSOLE_LOGLEVEL_DEFAULT,    /*  控制台日志级别 */
    MESSAGE_LOGLEVEL_DEFAULT,    /* 默认消息日志级别 */
    CONSOLE_LOGLEVEL_MIN,        /* 最小的控制台日志级别 */
    CONSOLE_LOGLEVEL_DEFAULT,    /* 默认控制台日志级别 */
};
#define console_loglevel (console_printk[0])
#define default_message_loglevel (console_printk[1])
#define minimum_console_loglevel (console_printk[2])
#define default_console_loglevel (console_printk[3])
```

Linux将存储器和外设分为3个基础大类：

- 字符设备：必须以串行顺序依次进行访问的设备

- 块设备：可以按任意顺序进行访问，以块为单位进行操作

- 网络设备：

Linux设备驱动的重难点：

- 编写Linux设备驱动弄要求工程师有非常好的硬件基础，懂得SRAM、Flash、SDRAM磁盘的读写方式，UART、I2C、USB等设备的接口以及轮询、中断、DMA的原理，PCI总线的工作方式以及CPU的内存管理单元MMU等

- 需要有C语言基础，灵活运用C语言的结构体、指针、函数指针及内存动态申请和释放等

- Linux内核基础，至少要明白驱动和内核接口，尤其是块设备、网络设备、Flash设备、串口设备等复杂设备，内核定义的驱动体系结构本身就非常复杂

- 多任务并发控制和同步基础，因为在驱动中会大量使用自旋锁、互斥、信号量、等待队列等并发与同步机制。

# 3、Linux编码风格

## 3.1 命名习惯

```c
#define PI 3.1415926

int min_value, max_value;    //单词间用下划线_连接，不使用驼峰

void send_data(void);        //函数也一样
```

## 3.2 代码缩进用"TAB"

## 3.3 代码括号"{"和"}"使用原则：

```c
struct var_data {        //1. 结构体、流程控制语句"{"不另起一行，符号前空格
    int len;
    char data[0];
};
if (a == b) {
    a = c;
    d = a;
}
for (i = 0;i < 10; i++) {
    a = c;
    d = a;
}

for (i = 0; i < 10; i++)    //2. 语句只有一行，不加"{"和"}"
    a = c;

if (x == y) {                //3. if和else混用，else语句不另起一行
    ...
} else if (x > y) {
    ...
} else {
    ...
}


int add(int a, int b)        //4. 对函数，"{"另起一行
{
    return a + b;
}
```

## 3.4 switch/case语句对齐

```c
switch (suffix) {
case 'G':
case 'g':
        mem <<= 30;
        break;
case 'M':
case 'm':
        mem <<= 20;
        break;
case 'K':
case 'k':
        mem <<= 10;
        /* fall through */
default:
        break;
}
```

## 3.5 空格分隔(看着办)

# 4、GNU C的编程特点

## 4.1 GNU C允许零长度数组

```c
struct var_data {
    int len;
    char data[0];    //此处定义0长度数组被允许
};
```

char data[0]仅仅意味着程序通过var_data结构体实例的data[index]成员可以访问len之后的第index个地址，**<u>它并没有为data[]数组分配内存，因此sizeof (struct var_data) = sizeof (int)</u>**

## 4.2 允许用变量定义数组

```c
int n = 4;
double x[n];
```

## 4.3 case可范围指定

    支持case x...y这样的语法，区间[x, y]中的数都满足case条件

```c
switch (ch) {
case '0'... '9': c -= '0';
    break;
case 'a'... 'f': c -= 'a' - 10;
    break;
case 'A'... 'F': c -= 'A' - 10;
    break;
}

//case '0'...'9'等价为标准C中case '0': case '1': case '2': ...... case '9': 
```

## 4.4 ()中的复合语句看做表达式，称为语句表达式

```c
#define min_t(type, x, y) \
({type __x = (x); type __y = (y); __x < __y ? __x : __y; })

int ia, ib, mini;
float fa, fb, minf;

mini = min_t(int, ia, ib);
minf = min_t(float, fa, fb);
```

        因为重新定义了__x和__y两个局部变量，所以用上述方式定义的宏不会有副作用，而在标准C中，对应的如下宏会产生副作用：

```c
#define min(x, y) ((x) < (y) ? (x) : (y))        
```

```c
    代码 min(++ia, ++ib)会展开为((++ia) < (++ib) ? (++ia) : (++ib))，
传入宏的"参数"增加两次
```

## 4.5 typeof关键字

    typeof(x)语句可以获得x的类型

```c
#define min_t(x, y) ({   \
const typeof(x) _x = (x); \
const typeof(y) _y = (y); \
(void) (&_x == &_y);      \
_x < _y ? _x : _y; })
//(void) (&_x == &_y)的作用是检查_x和_y的类型是否一致

int ia, ib, mini;
float fa, fb, minf;

mini = min_t(ia, ib);
minf = min_t(fa, fb);
```

## 4.6 可变参数宏

    标准C就支持可变参数函数，意味着函数的参数是不固定的，如: printf函数原型

```c
int printf( const char *format [,argument]...);
```

    GNU C中，宏也可以接受可变数目的参数

```c
#define pr_debug(fmt, arg...)    \
        printk(fmt, ##arg);
```

    这里arg表示其余的参数，可以有0个或多个，这些参数以及参数间的`,`构成arg的值，在宏扩展时替换arg，

```c
pr_debug("%s:%d", filename, line);
```

实际被替换为

```c
printk("%s:%d", filename, line);
```

```c
    "##"是为了处理arg不代表任何参数的情况，这时前面的逗号就多余了。
使用"##"后，GNU C预处理器会丢弃前面的逗号

pritnk("success!\n",)    避免像这种情况
```

## 4.7 标号元素 通过索引或结构体成员名指定初始化

    **标准C**要求<u>数组或结构体的初始化值</u>必须<u>以固定顺序出现</u>

    而**GNU C**中通过<u>指定索引或结构体成员名，允许初始化值以任意顺序出现</u>

```c
unsigned char data[MAX] = { [0 ... MAX-1] = 0};
//指定[0, MAX-1]范围的元素初始化为0

char data[MAX] = { [2] = 1 }
//指定下标为2的元素初始化为1

struct file_operations ext2_file_operations = {
    llseek: generic_file_llseek,
    read: generic_file_read,
    write: generic_file_write,
    ioctl: ext2_ioctl,
    mmap: generic_file_mmap,
    open: generic_file_open,
    release: ext2_release_file,
    fsync: ext2_sync_file,
};
//借助成员名初始化结构体(可乱序初始化 或 个别初始化)

struct file_operations ext2_file_operations = {
.llseek       = generic_file_llseek,
.read         = generic_file_read,
.write        = generic_file_write,
.aio_read     = generic_file_aio_read,
.aio_write    = generic_file_aio_write,
.ioct         = ext2_ioctl,
.mmap         = generic_file_mmap,
.open         = generic_file_open,
.release      = ext2_release_file,
.fsync        = ext2_sync_file,
.readv        = generic_file_readv,
.writev       = generic_file_writev,
.sendfile     = generic_file
};
//也支持标准C方式
```

## 4.8 当前函数名`__func__`

    GNU C中有两个标识符保存当前函数名字，__FUNCTION__保存函数在源码中的名字，__PRETTY_FUNCTION__保存带语言特色的名字。在C函数中，这两个名字是相同的

```c
/* __func__宏已被C99支持，不再使用__FUNCTION__ */
void example()
{
    printf("This is function:%s", __func__);
}
//__func__相当于"example"
```

## 4.9 特殊属性声明没讲明

    GNU C允许声明函数、变量、类型的特殊属性，在声明后添加`__attribute__((ATTRIBUTE))` 其中，ATTRIBUTE包括noreturn、format、section、aligned、packed等十多个属性，声明多个属性间用逗号隔开

    noreturn用于函数，表示该函数不返回。这会让编译器优化代码，并消除不必要的警告信息

```c
#define ATTRIB_NRET __attribute__((noreturn)) ....
asmlinage NORET_TYPE void do_exit(long error_code) ATTRIB_NORET;
```

    format用于函数，表示该函数使用printf、scanf或strfime风格的参数，指定format属性可以让编译器根据格式串检查参数类型

```c
asmlinkage int printk(const cha * fmt, ...) \
__attribute__((format (printf, 1, 2)));
```

    unused用于函数和变量，表示改函数或变量可能不会用到，避免编译器产生警告信息

    aligned属性用于变量、结构体或联合体，指定变量、结构体或联合体的对齐方式，以字节为单位

```c
struct example_struct {        //12
    char a;        //1
    int b;         //4
    long c;        //4
} __attribute__((aligned(4)));    //表示该结构体类型的变量以4字节对齐
```

    packed用于变量和类型，用于变量或结构体成员时表示使用最小可能的对齐，用于枚举、结构体或联合体类型时便是改类型使用最小内存

```c
struct example_struct {
    char a;            //1
    int b;            //4
    long c __attribute__((packed));        //4
};
//对齐的目的是更快访问对应内存，
```

## 4.10 内建函数

    不属于库函数的其他内建函数命名通常以`__builtin`开始，如：

- 内建函数`__builtin_return_address (LEVEL)`返回当前函数或其调用者的返回地址，参数LEVEL指定调用栈的级数。如0表示当前函数的返回地址，1表示当前函数的调用者的返回地址

- `__builtin_constant_p (EXP)` 用于判断一个值是否为编译时常数，如果参数EXP的值是常数，函数返回1，否则返回0

```c
//检测第一个参数是否为编译时常数以确定采用参数版本还是非参数版本
#define test_bit(nr, addr)    \
(__builtin_constant_p(nr) ?   \
constant_test_bit((nr, addr) : \
variable_test_bit((nr, addr)))
```

- `__builtin_expect (EXP, C)`用于为编译器提供分支预测信息，返回值是整数表达式EXP的值，C的值必须是编译时常数

```c
#define likely_notrace      __builtin_expect (!!(x), 1)
#define unlikely_notrace    __builtin_expect (!!(x), 0)
```

```c
    Linux内核编程中常用的likely()和unlikely()底层调用的likely_notrace()、
unlikely_trace()就是基于__builtin_expect (EXP, C)实现
    若代码中出现分支，则可能中断流水线。可通过likely()和unlikely()暗示分支
容易成立还是不容易成立
```

```c
if (likely(!IN_DEV_ROUTE_LOCALNET(in_dev)))
    if (ipv4_is_loopback(saddr))
        goto e_inval;
```

- 告诉gcc不使用GNU扩展语法编译，`-ansi-pedantic`，如在编译时使用，则以标准C语法进行编译，不嫩使用GNU扩展语法（如0长度数组）

## 4.11 do{ }while (0)语句

    do{ }while (0) 语句只执行一次，在Linux内核中常见，实际用法主要在宏定义中

```c
#define SAFE_FREE(p) do{ free(p); p = NULL;} while(0)
```

```c
if(NULL != p)
    SAFE_FREE(p);    //替换为 do{ free(p); p = NULL;} while(0)和;
else
    .../* do something */
```

    如果去掉do{}while(0)会被展开为

```c
if (NULL != p)
    free(p);    p = NULL;    // 此处不是单语句，导致报错
else
    ... /* do something */

1）因为if分支后有两个语句，导致else分支没有对应的if，编译失败。
2）假设没有else分支，则SAFE_FREE中的第二个语句无论if测试是否通过都会执行。
```

```c
    do{} while(0)作为宏定义函数的语句块集束，能方便C程序中每个语句后都加';'的习惯，
所以不用{}语句块的方式直接作为宏定义
```

## 4.12 goto语句

    Linux内核源代码中对goto一般只限于<u>错误处理</u>中，其结构如：

```c
if (register_a() != 0)
    goto err;
if (register_b() != 0)
    goto err1;
if (register_c() != 0)
    goto err2;
if (register_d() != 0)    //d寄存器申请失败，没有需要注销、释放的资源
    goto err3;
...

err3:                     //从最晚到最早申请资源的寄存器依次注销和释放资源
    unregister_c();
err2:
    unregister_b();
err1:
    unregister_a();
err:
    return ret;            //返回对应的ret信息结束进程
```

    这种 goto 用于错误处理的用法简单高效，只需保证在错误处理时注销、资源释放等，与正常的注册、资源申请顺序相反。

## 4.13 工具链

    Linaro工具链--ARM Linux开发常用

    典型ARM Linux工具链--arm-linux-gnueabihf-gcc、strip、gcc、objdump、ld、gprof、nm、readelf、addr2line等

# 5、Linux内核模块

## 5.1 Linux模块(Module)特点

- 模块本身不被编译入内核映像，从而控制了内核的大小

- 模块一旦被加载，它就和内核中的其他部分完全一样

## 5.2 最简单的内核模块代码

```c
/*
 * a simple kernel module: hello
 */

#include <linux/init.h>
#include <linux/module.h>

static int __init hello_init(void)    //加载函数
{
    printk(KERN_INFO "Hello World enter\n");
    return 0;
}
module_init(hello_init);    //内核模块加载

static void __exit hello_exit(void)    //卸载函数
{
    printf(KERN_INFO "Hello World exit\n");
}
moudule_exit(hello_exit);    //内核模块卸载

MODULE_AUTHOR("Barry Song <21cnbao@gmain.com");
MODULE_LICENSE("GPL v2");            //许可权限声明
MODULE_DESCRIPTION("A simple Hello World Module");
MODULE_ALIAS("a simplest module");
```

    编译产生.ko目标文件，通过`insmod ./hello.ko`加载，通过`rmmod hello`卸载，加载和卸载时分别输出"Hello World enter"和"Hello World exit"

## 5.3 内核模块相关的命令

```c
    1. 内核模块用于输出的函数是内核空间的printk()不是用户空间的printf()，printk()
可定义输出级别。
    2. lsmod 可以获得系统中已加载的所有模块以及模块间的依赖关系，lsmod命令实际是读取
并分析"/proc/modules"文件
    3. 已加载模块的信息也存于"/sys/module"，加载hello.ko后，内核将包含
"sys/module/hello"目录，该目录下有refcnt文件和一个sections目录，在
sys/module/hello目录下运行"tree -a"可获得目录树
    4. modprobe 命令能同时加载模块和模块依赖的其他模块，也可用于同时卸载这些。模块
之间的依赖关系存放在"/lib/modules/<kernel-version>/modules.dep"文件中，实际上
是在整体编译内核的时候由depmod工具生成
    5. 使用modinfo<模块名>命令可以获得模块信息，包括作者、说明、支持参数和vermagic
```

## 5.4 Linux内核模块程序结构

1） 模块加载函数

    通过`insmod`或`modprobe`命令加载内核模块时，加载函数会自动被内核执行，完成初始化工作

2） 模块卸载函数

3） 模块许可证声明

    许可证(`LICENSE`)声明描述内核模块的许可权限，如果不声明LICENSE，模块被加载时，将受到内核被污染（`Kernel Tainted`）的警告。

    Linux内核模块领域，可接受的LICENSE包括"`GPL`"、"`GPL v2`"、"`GPL and additional rights`"、"`Dual BSD/GPL`"、"`Dual MPL/GPL`"和"`Proprietary`"

    大多数情况下，内核模块应遵循GPL兼容许可权。最常见为

```c
MODULE_LICENSE("GPL v2");
```

4）模块参数（可选）

    加载时可传递的值，对应模块内部全局变量

5） 模块导出符号（可选）

    symbol，对应函数或变量，若导出，其他模块可使用

6） 模块作者等信息声明（可选）

### 5.4.1 模块加载函数

```c
/* Linux内核模块加载函数一般以__init标识声明 */
static int __init initialization_function(void)
{
    /* 初始化代码 */
}      
module_init(initialization_function);
/* 模块加载函数用module_init(函数名)的方式指定，初始化成功返回0，失败时返回错误编码，
 * 在Linux中，错误编码是接近0的负值，在<linux/errno.h>中定义，包含-ENODEV、-ENOMEM
 * 之类的符号值，可以通过perror等方法输出错误信息提示  
*/
```

```c
request_module(const char*fmt, ...);    //加载其他内核模块

/* 常用方法 */
request_module(module_name); 
```

- 在linux中，所有标识为__init的函数如果直接编译进入内核，成为内核镜像的一部分，在连接的时候都会放在.init.text区段中
  
  ```c
  #define __init  __attribute__((__section__ (".init.text"))
  ```

- __init函数在.initcall.init区段中也保存了一份函数指针，在初始化时内核通过指针调用这些__init函数，并在初始化完成后，释放init区段（包括.init.text、.initcall.init等）内存

- 数据可用__initdata形容，对于只是初始化阶段需要的数据，在初始化完成后也可以释放它们占用的内存
  
  ```c
  static int hello_data __initdata = 1;
  ```

### 5.4.2 模块卸载函数

```c
/* 卸载函数一般用__exit标识声明 */
static void __exit cleanup_function(void)    /* 不返回任何值*/
{
    /* 释放代码 */
}
module_exit(cleanup_function);
/* 用__exit告诉内核如果相关模块被编译进内核(built-in)，卸载函数直接忽略，不链进
 * 最后镜像。
 */
```

- 数据可用__exitdata来形容，只在退出阶段采用的数据

### 5.4.3 模块参数

```c
module_param(参数名, 参数类型, 参数读/写权限)
```

```c
static char *book_name = "dissecting Linux Device Driver";
module_param(book_name, charp, S_IRUGO);
static int book_num = 4000;
module_param(book_num, int S_IRUGO);
```

- 装载内核模块时，可向模块传递参数，`insmode(或modprobe) 模块名 参数名=参数值`，如果不传递，参数用模块内定的缺省值。如模块被内置，则无法insmod，可以用bootloader通过在bootargs你设置"模块名.参数值 = 值"的形式传参

- 参数类型: byte、short、ushort、int、uint、long、ulong、charp（字符指针）、bool或invbool(布尔的反)，编译时会比对定义类型

- 模块也可定义参数数组，`module_param_array(数组名, 数组类型, 数组长, 参数读/写权限)`

- 运行ismod和modprobe命令时，应使用逗号分隔输入的数组元素

- 参数读写权限不为0时，/sys/module/被加载模块名 目录下会出现parameters目录，其中包含一系列以参数名命名的文件节点，权限值与传入module_param()参数读写权限一致

```c
#include <linux/init.h>
#include <linux/module.h>

static char *book_name = "dissecting Linux Device Driver";
module_param(book_name, charp, S_IRUGO);    //绑定模块参数1

static int book_num = 4000;        
module_param(book_num, int, S_IRUGO);        //绑定模块参数2

static int __init book_init(void)    //加载函数
{
    printk(KERN_INFO "book name:%s\n", book_name);
    printk(KERN_INFO "book num:%d\n", book_num);
    return 0;
}
module_init(book_init);    //指定加载函数

static void __exit book_exit(void)    //卸载函数
{
    printk(KERN_INFO "book moudle exit\n");
}
moudule_exit(book_exit);    //指定卸载函数

MODULE_AUTHOR("Barry Song <baohua@kernel.org>");
MODULE_LICENSE("GPL v2");
MODULE_DECRIPTION("A simple Module for testing module params");
MODULE_VERSION("V1.0");
```

- 通过查看"/var/log/messages"日志文件可以看到内核输出
  
  ```shell
  tail -n 2 /var/log/messages
  ```

- insmod book.ko book_name='GoodBook' book_num=5000 加载book模块时带参

### 5.4.4 导出符号

```c
EXPORT_SYMBOL(符号名);
EXPROT_SYMBOL_GPL(符号名);    //只适用于包含GPL许可权的模块
```

```c
#include <linux/init.h>
#include <linux/module.h>

int add_integar(int a, int b)
{
    return a + b;
}
EXPORT_SYMBOL(add_integar);    //导出加运算函数符号

int sub_integar(int a, int b)
{
    return a - b;
}
EXPORT_SYMBOL(sub_integar);    //导出减运算函数符号

MODULE_LICENSE("GPL v2");
```

- 从"/proc/kallsyms"可以看到add_integar和sub_integar相关信息

### 5.4.5 模块声明和描述

1. MODULE_AUTHOR        模块作者

2. MODULE_DESCRIPTION 模块描述

3. MODULE_VERSION 模块版本

4. MODULE_DEVICE_TABLE 设备表

5. MODULE_ALIAS 模块别名
- 对于USB、PCI等设备驱动，通常会创建一个MODULE_DEVICE_TABLE，以表明该驱动模块所支持的设备
  
  ```c
  /* table of devices that work with this driver */
  static struct usb_device_id skel_table [] = {
  {    USB_DEVICE(USB_SKEL_VENDOR_ID,
             USB_SKEL_PRODUCT_ID)  },
      { }    /* terminating enttry*/
  };
  MODULE_DEVICE_TABLE (usb_device_id);
  ```

### 5.4.6 模块的编译

```makefile
KVERS = $(shell uname -r)
# KERNEL modules
obj-m += hello.o
#Specify flags for the module compilation.
#EXTRA_CFLAGS=-g -O0
build: kernel_modules
kernel_modules:
    make -C /lib/modules/$(KVERS)/build M=$(CURDIR) modules
clean:
    make -C /lib/modules/$(KVERS)/build M=$(CURDIR) clean
```

- 改Makefile文件应该与源代码helllo.c位于同一目录，开启其中的EXTRA_CFLAGS=-g -O0，可以得到包含调试信息的hello.ko模块。

- 如果一个模块包括多个.c文件(如file1.c、file2.c)
  
  ```makefile
  obj-m := modulename.o
  modulename-objs := file1.o file2.o
  ```

# 6、Linux文件操作

## 6.1 文件操作系统调用

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
```

### 6.1.1 创建文件

```c
int creat(const char *filename, mode_t mode);

/* mode指定文件的存取权限，同umask一起决定文件的最终权限(mode&umask) */
```

```c
int umask(int newmask);
/* 代表创建时需要去掉的一些存储区权限，可通过系统调用来改变*/
```

- mode可以是以下值的组合：
  
  - S_IRUSR 用户可以读
  
  - S_IWUSR 用户可以写
  
  - S_IXUSR 用户可以执行
  
  - S_IRWXU 用户可以读、写、执行
  
  - S_IRGRP 组可以读
  
  - S_IWGRP 组可以写
  
  - S_IXGRP 组可以执行
  
  - S_IRWXG 组可以读、写、执行
  
  - S_IROTH 其他人可以读
  
  - S_IWOTH 其他人可以写
  
  - S_IXOTH 其他人可以执行
  
  - S_IRWXO 其他人可以读
  
  - S_ISUID 设置用户的执行ID
  
  - S_ISGID 设置组的执行ID

### 6.1.2 打开文件

```c
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);
```

- open()函数有两个形式，flags可以是以下值的组合(| 或)
  
  - - O_RDONLY    只读打开
    
    - O_WRONLY 只写打开 三选一
    
    - O_RDWR 读写打开
  
  - O_APPEND 追加打开
  
  - O_CREAT 创建
  
  - O_EXEC 如果使用了O_CREATE并且文件已经存在，就会发生一个错误
  
  - O_NOBLOCK 以非阻塞方式打开
  
  - O_TRUNC 如文件已存在，清空文件内容

- 如果使用的是O_CREAT标志，则使用的是三参数，需要制定mode表示文件访问权限。

- Linux用5个数字来表示文件的各种权限：第一位设置用户id；二位设置组ID；第三位设置用户权限位；第四位组权限位；第五位其他人权限位

```c
open("test", O_CREAT, 10 705);
/* 等价 */
open("test", O_CREAT, S_IRWXU | S_IROTH | S_IXOTH | S_ISUID)
```

- 打开成功返回文件描述符(int类型)，失败时返回-1。对该文件的所有操作通过文件描述符进行操作实现

### 6.1.3 读写文件

```c
int read(int fd, const void *buf, size_t length);
/* 从文件描述符fd找到指定文件，读取length字节内容，写入到buf指向的缓冲区 */

int write(int fd, const void *buf, size_t length);
/* 从文件描述符fd找到指定文件，从buf指向的缓冲区读取length字节内容，写入到文件中 */
```

- 文件打开后，才能通过文件描述符fd对文件进行读写操作

- 读写操作前需要开辟对应的缓冲区，写操作需要先在缓冲区中填入要写入文件的内容，读操作会覆盖缓冲区的值

- 返回值为实际读取或写入的字节数

### 6.1.4 定位光标移动

```c
int lseek(int fd, offset_t offset, int whence);
```

- lseek()将文件读写指针相对whence移动offset个<u>字节</u>，操作成功时返回文件指针相对于文件头的位置

- whence可使用下述值：
  
  - SEEK_SET: 相对文件开头
  
  - SEEK_CUR: 相对文件读写指针的当前位置
  
  - SEEK_END: 相对文件末尾

- offset可为负值

```c
lseek(fd, -5, SEEK_CUR);    //将文件指针相对当前位置前移5个字节
```

```c
lseek(fd, 0, SEEK_END);    //返回值为文件内容长度
                           //(SEEK_END + offset - SEED_SET)
```

### 6.1.5 关闭文件

```c
int close(int fd);
```

Linux文件操作(用户空间):

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#define LENGTH 100
main()
{
    int fd, len;
    char str[LENGTH];

    fd = open("hello.txt", O_CREAT | O_RDWR,  S_IRUSR | S_IWUSR);
    /* 创建并打开文件hello.txt，权限为用户可读可写，读写方式打开 */
    if (fd) {    //如果文件打开成功，fd > 0
        write(fd, "Hello World", strlen("Hello World"));
        /* 写入字符串 */
        close(fd);
    }

    fd = open("hello.txt", O_RDWR);
    len = read(fd, str, LENGTH);
    /* 读取fd对应文件，用str数组接收，最长读取长度为100，实际读取长度记录在len */
    str[len] = '\0';    //不接收字符串结尾'\0'字符标识?
    printf("%s\n", str);
    close(fd);
}
```

## 6.2 C库 文件操作

        C库函数的文件操作独立于具体操作系统平台，能用，都能用

### 6.2.1 C库函数创建和打开文件

```c
FILE *fopen(const char *path, const char *mode);
```

- mode可用值
  
  - r、rb ：只读
  
  - w、wb： 只写，如果文件不存在则创建，否则文件被截断
  
  - a、ab: 追加打开，文件不存在则创建
  
  - r+、r+b、rb+: 读写打开
  
  - w+、w+b、wb+: 读写打开，不存在则创建，否则文件被截断
  
  - a+、a+b、ab+: 读和追加方式打开，不存在则创建

- b用于区分二进制文件和文本文件，在DOS、Windows系统中区分，但<u>在Linux中不区分二进制和文本文件</u>

### 6.2.2 C库函数读写文件

```c
int fgetc(FILE *stream);
int fputc(int c, FILE *stream);
char *fgets(char *s, int n, FILE *stream);
int fputs(const char *s, FILE *stream);
int fprintf(FILE *stream, const char *format, ...);
int fscanf(FILE *stream, const char *format, ...);
size_t fread(void *ptr, size_t size, size_t n, FILE *stream);
/* 从流中读取n个字段，每个字段为size字节，并将读取的字段
 * 放入ptr所指的字符数组中，返回实际读取的字段数
 * 读取字段数小于n时，可能是调用时出现错误也可能是读取到文件结尾，
 * 需要调用feof()和ferror()来判断
 */
size_t fwrite(const void *ptr, size_t size, size_t n, FILE * stream);
/* 从缓冲区ptr所指数组中把n个字段写入到流，返回实际写入字段数 */
```

```c
int fgetpos(FILE *stream, fpos_t *pos);
int fsetpos(FILE *stream, const fpos_t *pos); 
int fseek(FILE *stream, long offset, int whence);    //定位
```

### 6.2.3 C库关闭文件

```c
int fclose(FILE *stream);
```

利用C库函数完整打开、读、写文件(用户空间)：

```c
#include <stdio.h>
#define LENGTH 100
main()
{
    FILE *fd;
    char str[LENGTH];

    fd = fopen("hello.txt", "w+");    /* 创建并读写打开文件 */
    if (fd) {
        fputs("Hello World", fd);    /* 写入 */
        fclose(fd);
    }

    fd = open("hello.txt", "r");
    fgets(str, LENGTH, fd);    /* 读取文件内容，最长LENGTH，并保存到str */
    printf("%s\n", str);
    fclose(fd);
}
```

# 7、Linux文件系统

## 7.1 系统目录结构

- /bin
  
  - 包含基本命令，如ls、cp、mkdir等，这个目录中的文件都是可执行的

- /sbin
  
  - 系统命令，如modprobe、hwclock、ifconfig等，大多涉及系统管理的命令，这个目录中的文件都是可执行的

- /dev
  
  - 设备文件存储目录，应用程序通过对这些文件的读写和控制以访问实际设备

- /etc
  
  - 系统配置文件，一些服务器的配置文件也在这，如用户账号密码配置文件。busybox的启动脚本也在这个目录

- /lib
  
  - 系统库文件存放目录等

- /mnt
  
  - 一般用于存放挂载储存设备的挂载目录，比如含有cdrom等目录。可以参看/etc/fstab的定义。有时可让系统开机自动挂载文件系统，并把挂载点放在这个路径

- /opt
  
  - "可选"，有些软件会被安装在这

- /proc
  
  - 操作系统运行时，进程及内核信息(如CPU、硬盘分区、内存信息等)存放点。/proc目录为伪文件系统proc的挂载目录，proc并不是真正的文件系统，它存放于内存中

- /tmp
  
  - 存放用户运行程序时产生的临时文件

- /usr
  
  - 系统存放程序的目录，比如用户命令、用户库等

- /var
  
  - "变化"，目录内容经常变动，如/var/log用于存放系统日志

- /sys
  
  - Linux 2.6以后的内核所支持的sysfs文件系统被映射到此目录上。Linux设备驱动模型中的总线、驱动和设备都可以在sysfs文件系统中找到对应的节点。当内核检测到在系统中出现了新设备后，内核会在sysfs文件系统中为该新设备生成一项新的记录

## 7.2 Linux文件系统与设备驱动

- 字符设备的上层没有类似于磁盘的ext2等文件系统，它的file_oerations成员函数直接由设备驱动提供

- 块设备有两种访问方法，一种是不通过文件系统直接访问裸设备；另一种是通过文件系统来访问块设备

- ext2、fat、Btrfs等文件系统中会实现针对VFS（虚拟文件系统）的file_operations成员函数，设备驱动层将看不到file_operations的存在

- 设备驱动程序的设计中，一般而言，会更关心file和inode两个结构体

### 7.2.1 file结构体

```c
struct file {
    union {
    struct llist_node    fu_llist;
    struct rcu_head      fu_rcuhead;
    } f_u;
    struct path          f_path;
    #define f_dentry     f_path.dentry
    struct inode         *f_inode;        /* cached value */
    const struct file_operations *f_op;    /* 和文件关联的操作 */

    /*
     * Protects f_ep_links, f_flags.
     * Must not be taken from IRQ context.
     */
    spinlock_t        f_lock;
    atomic_long_t     f_count;
    unsigned int      f_flags;        /* 文件标志，
                                       * 如O_RDONLY、O_NONBLOCK、O_SYNC */
    fmode_t           f_mode;   /* 文件读/写模式，FMODE_READ和FMODE_WRITE */
    struct mutex      f_pos_lock;
    loff_t            f_pos;        /* 文件当前读写位置 */
    struct fown_struct f_owner;
    const struct cred  *f_cred;
    struct file_ra_statef_ra;

    u64                f_version;
    #ifdef    CONFIG_SECURITY
        void    *f_security;
    #endif
    /* needed for tty driver, and maybe others */
    void        *private_data;    /* 文件私有数据 */

    #ifdef    CONFIG_EPOLL
    /* Used by fs/eventpolll.c to link all the hooks to this file */
        struct list_head    f_ep_links;
        struct list_head    f_tfile_llink;
    #endif                /* #ifdef CONFIG_EPOLL */
    struct address_space    *f_mapping;
} __attribute__ ((aligned(4)));
  /* lest something weird decides that 2 is OK */
```

- 文件读/写模式f_mode、标志f_flags都是设备驱动关心的内容，而私有数据指针private_data在设备驱动中呗广泛运用，大多被指向设备驱动自定义以用于描述设备的结构体

```c
/* 判断设备文件打开方式是阻塞还是非阻塞 */

if (file->f_flags & O_NONBLOCK)    /* 非阻塞方式 */
    pr_debug("open: non-blocking\n");
else                               /* 阻塞 */
    pr_debug("open: blocking\n");
```

### 7.2.2 inode结构体

- VFS inode包含文件访问权限、属主、组、大小、生成时间、访问时间、最后修改时间等信息。inode是Linux管理文件系统的最基本单位，也是文件系统连接任何子目录、文件的桥梁。

```c
struct inode {
    ...
    umode_t i_mode;        /* inode的权限 */
    uid_t i_uid;           /* inode的拥有者id */
    gid_t i_gid;           /* inode的所属群组id */
    dev_t i_rdev;          /* 若是设备文件，此字段将记录设备的设备号 */
    loff_t i_size;         /* inode所代表的文件大小 */

    struct timespec i_atime;    /* inode最近一次存取时间 */
    struct timespec i_mtime;    /* inode最近一次修改时间 */
    struct timespec i_ctime;    /* inode的产生时间 */

    unsigned int        i_blkbits;
    blkcnt_t        i_blocks;    /* inode所使用的block数，一个block为512字节 */
    union {
        struct pipe_inode_info    *i_pipe;
        struct block_device       *i_bdev;
                /* 若是块设备，为其的对应的block_device结构体指针 */
        struct cdev               *i_cdev;
                /* 若是字符设备，为其对应的cdev结构体指针 */
    }
    ...
};
```

- i_rdev字段包含设备编号。Linux内核设备编号分为主设备编号和次设备编号，前者为dev_t的高12位，后者为dev_t的低20位。

```c
/* 从inode中获得主、次设备号 */
unsigned int iminor(struct inode *inode);    //次
unsigned int imajor(struct inode *inode);    //主
```

- /proc/devices文件可以查看系统中注册的设备，第一列为主设备号，第二列为设备名

- /dev目录可以查看系统中包含的设备文件，日期前的两列给出了对应设备的主设备号和次设备号

- 主设备号是与驱动对应的概念，<u>同一类设备</u>一般使用相同的主设备号（<u>同一驱动可支持多个同类设备</u>，因此用次设备号来描述使用该驱动的设备的序号，一般从0开始）

- 不同类的设备一般使用不同的主设备号，但也不排除在同一主设备号下包含有一定差异的设备

- 内核Documents目录下devices.txt文件描述了Linux设备号的分配情况，是有统一标准由其他组织维护的。

### 7.2.3 devfs

- devfs，设备文件系统，Linux 2.4内核引入，使得设备驱动程序能自主地管理自己的设备文件。

- devfs的优点：
  
  1）可以通过程序在设备初始化时在、dev目录下创建设备文件，卸载设备时删除

        2）设备驱动程序可以指定设备名、所有者和权限位，用户空间程序仍可以修改所有者和权限位

        3）不再需要为设备驱动程序分配主设备号以及处理次设备号，在程序中可以直接给register_chrdev()传递0主设备号以获得可用的主设备号，并在devfs_register()中指定次设备号

- <u>驱动程序</u>调用下面函数来进行<u>设备文件的创建和撤销</u>工作：

```c
/* 创建设备目录 */
devfs_handle_t devfs_mk_dir (devfs_handle_t dir, const char *name, 
                            void *info);
/* 创建设备文件 */
devfs_handle_t devfs_register (devfs_handle_t dir, const char *name, 
    unsigned int flags, unsigned int major, unsigned int minor, 
    umode_t mode, void *ops, void *info);

/* 撤销设备文件 */
void devfs_unregister(devfs_handle_t de);
```

- 在Linux 2.4的设备驱动编程中，分别在模块加载、卸载函数中创建和撤销设备文件是被普遍采用并值得大力推荐的好方法：

- ```c
  /* 在模块加载、卸载函数中创建和撤销设备文件 模板 */
  #include <linux/init.h>
  #include <linux/module.h>
  
  static devfs_handle_t devfs_handle;
  static int __init xxx_init(void)
  {
      int ret;          // 
      int i;            // 不知道干嘛的
      /* 在内核中注册设备 */
      ret = register_chrdev(XXX_MAJOR, DEVICE_NAME, &xxx_fops);
      /* 字符设备注册 */
      if (ret < 0) {
          printk(DEVICE_NAME " can't register major number\n");
          return ret;
      }
      /* 创建设备文件 */
      devfs_handle = devfs_register(NULL, DEVICE_NAME, DEVFS_FL_DEFAULT,
          XXX_MAJOR, 0, S_IFCHR | S_IRUSR |S_IWUSR, &xxx_fops, NULL);
      ...
      printk(DEVICE_NAME " initialized\n");
      return 0;
  }
  
  static void __exit xxx_exit(void)
  {
      devfs_unregister(devfs_handle);                /* 撤销设备文件 */
      unregister_chardev(XXX_MAJOR, DEVICE_NAME);    /* 注销设备 */
  }
  
  module_init(xxx_init);
  module_exit(xxx_exit);
  ```

### 7.2.4 udev 用户空间设备管理

- devfs被udev取代了，原因：

        1）devfs所做的工作被确信可以在用户态来完成；

        2）devfs有bug，且部分无法修复

        3）devfs停止维护

- Linux设计中强调的一个基本观点是机制和策略的分离。

- **Linux内核中，不应该实现策略**。机制相对固定，是做某样事情的固定步骤、方法；策略不固定，是每一个步骤所采用的不同方式。

- udev规则：
  
  1. udev完全在用户态工作，利用设备加入或移除时内核所发送的热插拔事件(Hotplug Event)来工作
  
  2. 热插拔时，设备的纤细信息会由内核通过netlink套接字发送出来，发出的事情叫uevent
  
  3. udev的设备命名策略、权限控制和事件处理都是在用户态下完成，利用从内核收到的信息来进行创建设备文件节点等工作
  
  4. 冷插拔设备在开机时就存在，udev无法独立识别冷插拔设备，需要往sysfs下的uevent节点写一个"add"，使内核重新发送netlink

```c
#include <linux/netlink.h>

static void die(char *s)
{
    write(2, s, strlen(s);
    exit(1);
}

int main(int argc, char *argv[])
{
    struct sockaddr_nl nls;
    struct pollfd pfd;
    char buf[512];
    // Open hotplug event netlink socket

    memset(&nls, 0, sizeof(struct sockaddr_nl));
    nls.nl_family = AF_NETLINK;
    nls.nl_pid = getpid();
    nls.nl_groups = -1;

    pfd.events = POLLIN;    // 设备插入
    pfd.fd = socket(PF_NETLINK, SOCK_DGRAM, NETLINK_KOBJECT_UEVENT);
    if (pfd.fd == -1)
        die("Not root\n");

    // Listen to netlink socket
    if (bind(pfd.fd, (void *)&nls, sizeof(struct sockaddr_nl)))
        die("Bind failed\n");
    while (-1 != poll(&pfd, 1, -1)) {
        int i, len = recv(pfd.fd, buf, sizeof(buf), MSG_DONTWAIT);
        if (len == -1) 
            die("recv\n");

        // Print the data to stdout.
        i = 0;
        while (i < len) {
            printf("%s\n", buf + i);
            i += strlen(buf + i) + 1;
        }
    }
    die("poll\n");

    // Dear gcc: shut up.
    return 0;
}
```

### 7.2.5 sysfs文件与Linux设备模型(跳过)

- sysfs把连接在系统上的设备和总线组织成为一个分级的文件，它们可以由用户空间存取，向用户空间导出内核数据结构以及它们的属性。

- 该文件系统是一个虚拟的文件系统，它可以产生一个包括所有系统硬件的层级视图，与提供进程和状态信息的proc文件系统十分类似。

- 遍历sysfs的shell脚本

- ```shell
  !/bin/bash
  
  # Populae block devices
  
  for i in /sys/*/dev /sys/block/*/*/dev
  do
      if [ -f $i ]
      then
          MAJOR=$(sed 's/:.*//' < $i)
          MINOR=$(sed 's/.*://' < $i)
          DEVNAME=$(echo $i | sed -e 's@/dev@@' -e 's@.*/@@')
          echo /dev/$DEVNAME b $MAJOR $MINOR
      fi
  done
  
  # Populate char devices
  
  for i in /sys/bus/*/devices/*/dev /sys/class/*/*/dev
  do
      if [ -f $i ]
      then
          MAJOR=$(sed 's/:.*//' < $i)
          MINOR=$(sed 's/.*://' < $i)
          DEVNAME=$(echo $i | sed -e 's@/dev@@' -e 's@.*/@@')
          echo /dev/$DEVNAME c $MAJOR $MINOR
          #mknod /dev $DEVNAME c $MAJOR $MINOR
          # 为整个系统的设备建立/dev/下面的节点
      fi
  done
  ```

### 7.2.6 udev组成和规则文件(跳过)

# 8、Linux字符设备驱动结构

## 8.1 cdev结构体

- cdev结构体 用于 描述字符设备

```c
struct cdev {
    struct kobject kobj; /* 内嵌的kobject对象 */
    struct module *owner; /* 所属模块 */
    struct file_operations *ops; /* 文件操作结构体 */
    struct list_head list;
    dev_t dev; /* 设备号，高12位为主设备号，低20位为次设备号 */
    unsigned int count;
};

MAJOR(dev_t dev)        // 获取主设备号的宏
MINOR(dev_t dev)        // 获取次设备号的宏
MKDEV(int major, int minor)    // 组合主次设备号获取dev_t的宏
```

- file_operations定义了字符设备驱动提供给虚拟文件系统的接口函数

```c
/* 用于初始化cdev成员，并建立cdev和file_operations之间的连接 */
void cdev_init(struct cdev *cdev, struct file_operation *fops) {
    memset(cdev, 0, siezof *cdev);
    INIT_LIST_HEAD(&cdev->list);
    kobject_init(&cdev->kobj, &ktype_cdev_default);
    cdev->ops = fops;    /* 将传入的文件操作结构体指针复制给cdev的ops*/
}
```

```c
/* 用于动态申请一个cdev内存 */
struct cdev *cdev_alloc(void)
{
    struct cdev *p = kzalloc(sizeof(struct cdev), GFP_KERNEL);
    if (p) {
        INIT_LIST_HEAD(&p->list);
        kobject_init(&p->kobj, &ktype_cdev_dynamic);
    }
    return p;
}
```

- cdev_add()和cdev_del()分别是向系统添加和删除一个cdev，完成字符设备的注册和注销。这两个函数的调用通常发生的字符设备驱动模块加载/卸载函数中

## 8.2 分配和释放设备号

- 需要先调用register_chrdev_region()或alloc_chrdev_region()向系统申请设备号，才能调用cdev_add()向系统注册字符设备

```c
/* 用于已知起始设备号的情况 */
int register_chrdev_region(dev_t from, unsigned count, 
                    const char *name);
/* 用于设备号未知，向系统动态申请未被占用的设备号的情况 */
int alloc_chrdev_region(dev_t *dev, unsigned baseminor, unsigned count,
                    const char *name);
```

- 在cdev_del()函数调用，从系统注销字符设备后，需要调用unregister_chrdev_region()以释放原先申请的设备号

```c
void unregister_chrdev_region(dev_t from, unsigned count);
```

## 8.3 file_operations结构体(文件操作集)

```c
struct file_operations {
    struct module *owner;
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *,
                 size_t, loff_t *);
    ssize_t (*aio_read) (struct kiocb *, const struct iovec *,
                 unsigned long, loff_t);
    ssize_t (*aio_write) (struct kiocb *, const struct iovec *,
                 unsigned long, loff_t);
    int (*iterate) (struct file *, struct dir_context *);
    unsigned int (*poll) (struct file *,
                 struct poll_table_struct *);
    long (*unlocked_ioctl) (struct file *,
                 unsigned int, unsigned long);
    long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
    int (*mmap) (struct file *, struct vm_area_struct *);
    int (*open) (struct inode *, struct file *);
    int (*flush) (struct file *, fl_owner_t id);
    int (*release) (struct inode *, struct file *);
    int (*fsync) (struct file *, loff_t, loff_t, int datasync);
    int (*aio_fsync) (struct kiocb *, int datasync);
    int (*fasync) (int, struct file *, int);
    int (*lock) (struct file *, int, struct file_lock *);
    ssize_t (*sendpage) (struct file *, struct page *,
                 int, size_t, loff_t *, int);
    unsigned long (*get_unmapped_area)(struct file *,
                 unsigned long, unsigned long,
                unsigned long, unsigned long);
    int (*check_flags)(int);
    int (*flock) (struct file *, int, struct file_lock *);
    ssize_t (*splice_write)(struct pipe_inode_info *, struct file *,
                loff_t *, size_t, unsigned int);
    ssize_t (*splice_read)(struct file *,
                loff_t *, struct pipe_inode_info *,
                size_t, unsigned int);
    int (*setlease)(struct file *, long, struct file_lock **);
    long (*fallocate)(struct file *file, int mode, loff_t offset,
                loff_t len);
    int (*show_fdinfo)(struct seq_file *m, struct file *f);
};
```

# 9、设备驱动基本要素和内核tips

kernel/arch/arm

ARM是统一编程，0-3G是用户空间，3-4G是内核空间，内核占1G中的一部分，预留一小部分虚拟地址空间给IO等

物理地址和虚拟地址

iommu，mmu

iommu是设备手册中写的物理地址，引脚地址

mmu是从物理地址映射过来的虚拟地址

ioremap() ----实现物理地址到虚拟地址的转换

系统调用

1. 库实现
   
       open对应系统调用sys_open
   
       printf()对应多个系统调用
   
       malloc和calloc    内核的sys_brk

2. syscall

3. int (?不知道是啥，内核层调用)

内核线程    （linux内核没有进程只有线程，因为内核只有一个）

![](C:/Users/Peak-Li/AppData/Roaming/marktext/images/2023-06-13-17-54-22-3S@EV`$S@EZMJS8U@PH2}50.png)

Linux设备在内核中是用设备号进行区分的，而决定这些设备号的正是设备驱动程序。另外，<u>在用户空间如何管理这些设备</u>，这也是与驱动程序息息相关的。一个完整的设备驱动必须具备以下基本要素:

1. **驱动的入口和出口**。驱动入口和出口部分的代码，并不与应用程序直接交互，仅仅只与内核模块管理子系统有交互。在加载内核的时候执行入口代码，卸载的时候执行出口代码。<u>这部分代码与内核版本关系较大，严重依赖于驱动子系统的架构和实现</u>。**(__init标识的入口函数和module_init(函数名)指定该函数)**

2. **操作设备的各种方法**。驱动程序实现了各种用于系统服务的各种方法，但是这些方法并不能主动执行，发挥相应的功能，只能被动的等待应用程序的系统调用，只有经过相应的系统调用，各方法才能发挥相应的功能，如应用程序执行read()系统调用，内核才能执行驱动 xx_read0)方法的代码。这部分代码主要与硬件和所需要实现的操作相关。**(以file_operations结构体类型定义的xx_fops，将.read / .write / .open / .release等成员初始化为对应的函数指针，然后在入口函数xx_init()中用cdev_init(struct cdev xx, &xx_fops)进行字符设备初始化，以及申请引脚资源)**

3. **提供设备管理方法支持**。包括**设备号的分配**（<u>alloc_chrdev_region(&amp;(struct cdev xx), 指定主设备号或0,申请次设备号数量, "标记名称"</u>)）和**设备的注册**（cdev_add加入内核）等。这部分代码与内核版本以及最终所采用的设备管理方法相关系，如采用udev，则驱动必须提供相应的支持代码。

# 10、根据设备树编写按键驱动模块(考核3)

## 10.1 题目

**根据dts设备树和原理图编写按键驱动程序，实现按键驱动功能。**

gpio_keys_agn {        

        **compatible** = "agn,gpio_key";        // 兼容性

        **key-gpios** = <&pio 2 GPIO_ACTIVE_LOW //引脚资源

            &pio 7 GPIO_ACTIVE_LOW>;

    };

![](file:///C:/Users/Peak-Li/AppData/Roaming/marktext/images/2023-06-15-11-58-08-image.png?msec=1687158543812)

## 10.2 gpio_keys_agn -v3 当前版本(未实际编译)

```c
#include <linux/init.h>        // __init __exit
#include <linux/kernel.h>    // printk
#include <linux/module.h>    // module_init module_exit
#include <linux/cdev.h>
#include <linux/fs.h>
#include <linux/uaccess.h>
#include <linux/device.h>
#include <linux/ioport.h>
#include <linux/io.h>
#include <linux/gpio.h>
#include <linux/miscdevice.h>   // 混杂设备
#include <linux/platform_device.h>  // 平台设备驱动模型
#include <linux/of.h>        // 设备树相关头文件
#include <linux/of_gpio.h>
#include <linux/of_address.h>
#include <linux/of_fdt.h>
#include <linux/of_platform.h>
#include <linux/string.h>
/* 少了对应芯片类型的头文件 */

#if 0             //设备树  
gpio_keys_agn {         
        compatible = "agn,gpio_key";        // 兼容性
        key-gpios = <&pio 2 GPIO_ACTIVE_LOW //引脚资源
            &pio 7 GPIO_ACTIVE_LOW>;
    };   
#endif

/* 定义按键值 */
#define KEY_VAL         0xF0              // 按键摁下
#define INV_KEY_VAL     0x00              // 按键松开

/* key设备结构体 */
struct key_dev {
    int key_num = 2;                   /* gpio引脚占用数 */
    int key_gpios[key_num];            /* key使用的GPIO编号，总计2个 */
    atomic_t key_vals[key_num];        /* 按键值，对应两个GPIO引脚 */
} keydev;


static int key_open (struct inode *inode, struct file *filp)
{
    printk("key_open\n");

    return 0;
}


/* 关闭函数没有资源释放 */
static int key_close (struct inode *inode, struct file *filp)
{
    printk("key_close\n");

    return 0;
}

/* 处理获取的引脚电平数据，处理后送数据到用户空间 */
/* file:打开文件路径
 * buf: 用户用于接收端口高低电平
 * len:接收数组长度(未使用)
 * off:偏移量(未使用)
 */
static ssize_t key_read (struct file *filp, char __user *buf,
                     size_t len, loff_t *off)
{
    /* 获取两引脚高低电平信息 */
    u8 key_val = 0;        // 最高0~7，设置8位
    int num = keydev->key_num;

    // 检测所有按键状态，
    for(int key_no = 0; key_no < num; key_no++) {
        if (gpio_get_value(keydev->key_gpios[key_no]) == 0) /* 按键按下 */
            atomic_set(&keydev->key_vals[key_no], KEY_VAL);
        else                                     /* 按键没有按下 */
            atomic_set(&keydev->key_vals[key_no], INV_KEY_VAL);
    }

    /* 保存按键值 */
    for(int key_no = num - 1; key_no >= 0; key_no--) {
        if (keydev->key_vals[key_no] == KEY_VAL) 
            key_val |= 0x01;        // 用1表示按键按下

        if (key_no != 0)
            key_val = key_val<<1;   // 从右到左依次表示key1~keyn按键状态
    }

    /* #include <linux/uaccess.h>
     * unsigned long copy_to_user(void __user *to,
     *    const void *from, unsigned long n);
     */
    copy_to_user(buf, &key_val, sizeof(key_val));  // 送出数据

    return num; // 返回读取有效数据位数
}

// 文件操作结构体，绑定open、close、read函数
static const struct file_operations key_fops = {
    .open = key_open,
    .release = key_close,
    .read = key_read,
    .module = THIS.MODULE,
};

// misc结构体，用于后续平台申请设备号
static struct miscdevice key_misc = {
    .minor = MISC_DYNAMIC_MINOR,        // 动态分配次设备号
    .name = "gpio-key",                 // 设置设备名
    .fops = &key_fops,                  // 绑定fops
};


// 平台入口函数
static int key_probe(struct platform_device *pdev)
{
    int rt;
    struct device_node *pdevnode = pdev->dev.of_node;

    // 注册混杂设备
    rt = misc_register(&key_misc);

    if(rt < 0){
        printk("misc_register fail\n");

        return rt;
    }

    // 从设备树中查找"key-gpios",得到对应gpio口、引脚号、使能，共两组
    for (int num = 0; num < keydev->key_num; num++) {
        rt = of_get_named_gpio(pdevnode, "key-gpios", \
                     keydev->key_gpios[num], num);

        if(rt < 0 ){
            printk(KERN_ERR "of_get_named_gpio key-gpios fail\n");

            goto err_of_get_named_gpio;
        }

        // 提示引脚信息，包括引脚地址和引脚有效信号
        printk(KERN_INFO "key%d %s\n", num, keydev->key_gpios[num]);

        // 释放gpio引脚
        gpio_free(keydev->key_gpios[num]);

        char key_name[6];
        sprintf(key_name, "key%d", num);

        // 重新申请gpio引脚资源
        rt = gpio_request(keydev_num[num], key_name);
        if (rt < 0){
            printk(KERN_ERR "gpio_request fail\n");

            goto err_gpio_request;
        }
    }

    return 0;

// 错误处理
err_gpio_request:
err_of_get_named_gpio:
    misc_deregister(&key_misc); // 注销设备

    return rt;
}

// 平台卸载函数
static int key_remove(struct platform_device *pdev)
{
    for (int num = 0; num < keydev->key_num; num++)
        gpio_free(keydev->key_gpios[num]);     

    misc_deregister(&key_misc); // 注销设备

    return 0;
}

// 定义设备匹配信息结构
static const struct of_device_id of_key_match[] = {
    {
        .compatible = "agn,gpio_key",
    },  //compatible 兼容属性名，需要与设备树节点属性一致
    {},
};

static struct platform_driver key_driver = {
    .probe = key_probe,                 // 平台初始化函数
    .remove = __devexit_p(key_remove),  // 平台驱动的卸载函数
    .driver = {
        .owner  = THIS_MODULE,
        .name   = "gpio-key",           // 命名平台驱动，不能为空，否则会在安装驱动时出错
        .of_match_table = of_key_match, // 设备树设备匹配
    },
};

// 模块入口函数
static int __init gpio_init(void)
{
    int rt = platform_driver_register(&key_driver); // 绑定平台驱动结构体

    if (rt <0){
        printk(KERN_INFO "platform_driver_register err\n");

        return rt;
    }

    printk(KERN_INFO "gpio_init\n");

    return rt;
}

// 模块出口函数
static void __exit gpio_exit(void)
{
    platform_driver_unregister(&key_driver);

    printk("gpio_exit\n");
}

module_init(gpio_init);

module_exit(gpio_exit);

MODULE_AUTHOR("Linhongyi");
MODULE_DESCRIPTION("gpio-key driver");
MODULE_LICENSE("GPL"); 
```

## 10.3 分析

- `compatible`兼容性属性可以用于定位设备树对应节点`gpio_keys_agn`中的信息，将`of_device_id`结构体中`.compatible`成员设置为`"agn,gpio_key"`与设备树兼容性属性保持一致，然后在`platform_driver`结构体中`.driver`成员的`.of_match_table`成员进行匹配绑定，再在模块入口函数中用`platform_driver_register(&key_driver)`进行结构体绑定，获取对应设备树设备和对应设备节点。

- 模块初始化调用`platform_driver_register(&key_driver)`后会进入`key_driver`结构体绑定的平台入口函数`key_probe()`，在其中需要完成注册设备和资源申请。

- 注册设备用到的是`miscdevice`结构体绑定，即混杂设备注册方式，绑定文件操作结构体变量key_fops地址，绑定对应`open()`、`read()`、`close()`函数在驱动中的对应函数入口，同时申请设备号

- `key-gpios`能对应引脚资源，即`pio口2号引脚`和`pio口7号引脚`，他们的使能信号都是低电平触发。在platform平台驱动模型入口函数中使用`of_get_named_gpio()`获取`"key-gpios"`标识，分别获取对应顺序编号0,1两个引脚资源地址，然后再进行资源申请操作。

- 在平台卸载函数中需要释放申请的引脚资源和对混杂设备进行注销，对应`gpio_free()`和`misc_deregister()`

- 在整体上，把按键事件变换为摁下和松开，对应为`KEY_VAL`(0xF0)和`INV_KEY_VAL`(0x00)的01事件，定义对应的宏用于读取判断；同时定义key设备结构体key_dev，保存gpio引脚占用数(即按键数)、存储对应gpio引脚的资源地址集，以及用于保存读取按键状态值的集合

- `key_read()`函数中设置一个u8变量`key_val`用于保存处理后的按键状态集，逻辑上检测所有按键状态，同时将`key_dev`结构体变量`keydev`的成员`key_vals`对应元素写`KEY_VAL`或`INVKEY_VAL`，再将按键状态以01形式按按键名从高到低顺序从右到左进移位进入`key_val`中，然后送出`key_val`到用户输入的`buf`中，并返回读取有效数据位数
  
  - 在用户存储数组`buf`中，应有如下数据位格式:
    
    ...keyn,keyn-1,...,key1 对应0为松开，1为摁下，最高8个按键状态
