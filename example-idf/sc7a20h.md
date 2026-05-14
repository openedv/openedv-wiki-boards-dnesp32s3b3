---
title: '三轴传感器实验'
sidebar_position: 12
---

# 三轴传感器实验

## 前言

本章，我们将介绍一款高性价比三轴角速度传感器。本章我们将使用ESP32S3 来驱动 SC7A20H，读取其原始数据，并结合 LCD 显示，教大家如何使用这款三轴传感器。

## SC7A20H简介

SC7A20HTR 是一款由士兰微电子推出的超低功耗、高精度三轴数字加速度计。它支持 ±2/±4/±8/±16g 多量程，提供 12-bit 分辨率和 1.56Hz 至 4.434kHz 的灵活输出速率。芯片工作电压宽（1.71V-3.6V），在低功耗模式下电流可低至 2µA，并具备掉电模式。它通过 I²C 或 SPI 接口通信，内置 32 级 FIFO、多种智能中断（如点击、自由落体、方向检测）以及自测试功能，采用紧凑的 2x2mm LGA-12 封装，非常适合空间和功耗敏感的便携式及物联网设备。

## 硬件设计

### 例程功能

在 LCD 显示屏上，我们能够看到 XYZ 的数据。当我们翻转开发板时，这些数据会根据开发板的翻转角度来计算出 pitch 俯仰角和 roll 翻滚角。

### 硬件资源

1. LED灯
2. 正点原子2.4寸LCD屏幕
3. SC7A20H

### 原理图

SC7A20H器件相关原理图，如下图所示：
<img src={require('./img/SC7A20H.png').default} alt="00" width="528" />

## 程序设计

### SC7A20H 函数解析

由于SC7A20H使用了IIC进行驱动，那么关于IIC的API函数的介绍作者在前边的IIC_EXIO实验章节中已经讲解过了，在此不再赘述。

### SC7A20H 驱动解析

在 IDF 版 13_sc7a20h 例程中，作者在 ```13_sc7a20h\components\BSP``` 路径下新增了一个 SC7A20H 文件夹，分别用于存放 sc7a20h.c、 sc7a20h.h sc7a20h.h 文件负责声明温度传感器相关的函数和变量，而 sc7a20h.c 文件则实现了温度传感器的驱动代码。下面，我们将详细解析这两个文件的实现内容。

#### sc7a20h.h文件

```
/* SC7A20HA相关宏定义 */

#define SC7A20H_ADDR            0x19                        /* SDO引脚悬空/高电平时的地址，接地时为0x18 */
#define SC7A20H_ID              0x11                        /* 设备ID */
#define ONE_G                   9.807f                      /* 加速度单位转换使用 */
#define M_PI                    (3.14159265358979323846f)   /* 陀螺仪单位转换使用 */
#define MAX_CALI_COUNT          100                         /* 采样次数 */

/* 寄存器地址定义 */
#define SC7A20H_REG_CTRL0           0x1F        /* 控制寄存器0 */
#define SC7A20H_REG_CTRL1           0x20        /* 控制寄存器1 */
#define SC7A20H_REG_CTRL2           0x21        /* 控制寄存器2 */
#define SC7A20H_REG_CTRL3           0x22        /* 控制寄存器3 */
#define SC7A20H_REG_CTRL4           0x23        /* 控制寄存器4 */
#define SC7A20H_REG_CTRL5           0x24        /* 控制寄存器5 */
#define SC7A20H_REG_CTRL6           0x25        /* 控制寄存器6 */
#define SC7A20H_DRDY_STATUS_REG     0x27        /* 状态寄存器 */
#define SC7A20H_REG_OUT_X_L         0x28        /* X轴低字节 */
#define SC7A20H_REG_OUT_X_H         0x29        /* X轴高字节 */
#define SC7A20H_REG_OUT_Y_L         0x2A        /* Y轴低字节 */
#define SC7A20H_REG_OUT_Y_H         0x2B        /* Y轴高字节 */
#define SC7A20H_REG_OUT_Z_L         0x2C        /* Z轴低字节 */
#define SC7A20H_REG_OUT_Z_H         0x2D        /* Z轴高字节 */
#define SC7A20H_REG_WHO_AM_I        0x0F        /* 设备ID寄存器 */

/* 控制寄存器1 (0x20) 位定义 */
#define SC7A20H_ODR_1_56HZ          0x10        /* 1.56Hz输出数据率 */
#define SC7A20H_ODR_12_5HZ          0x20        /* 12.5Hz输出数据率 */
#define SC7A20H_ODR_25HZ            0x30        /* 25Hz输出数据率 */
#define SC7A20H_ODR_50HZ            0x40        /* 50Hz输出数据率 */
#define SC7A20H_ODR_100HZ           0x50        /* 100Hz输出数据率 */
#define SC7A20H_ODR_200HZ           0x60        /* 200Hz输出数据率 */
#define SC7A20H_ODR_400HZ           0x70        /* 400Hz输出数据率 */
#define SC7A20H_ODR_1_48KHZ         0x80        /* 1.48kHz输出数据率 */
#define SC7A20H_ODR_2_66KHZ         0x90        /* 2.66kHz输出数据率 */
#define SC7A20H_ODR_4_434KHZ        0xA0        /* 4.434kHz输出数据率 */
#define SC7A20H_LPEN                0x08        /* 低功耗模式使能 */
#define SC7A20H_ZEN                 0x04        /* Z轴使能 */
#define SC7A20H_YEN                 0x02        /* Y轴使能 */
#define SC7A20H_XEN                 0x01        /* X轴使能 */
#define SC7A20H_ENABLE_ALL_AXES     (SC7A20H_XEN | SC7A20H_YEN | SC7A20H_ZEN) // 使能所有轴

/* 控制寄存器4配置 */
#define SC7A20H_SCALE_2G            0x00        /* ±2G量程 */
#define SC7A20H_SCALE_4G            0x10        /* ±4G量程 */
#define SC7A20H_SCALE_8G            0x20        /* ±8G量程 */
#define SC7A20H_SCALE_16G           0x30        /* ±16G量程 */
#define SC7A20H_BDU_ENABLE          0x88        /* 块数据更新使能 */

/* 中断映射 */
#define SC7A20H_MAP_INT1            0x01
#define SC7A20H_MAP_INT2            0x02

typedef struct {
    uint8_t data[2];
    float  acc_x;
    float  acc_y;
    float  acc_z;
    float  acc_g;
    float  pitch;                       /* 围绕X轴旋转,也叫做俯仰角 */
    float  roll;                        /* 围绕Z轴旋转,也叫翻滚角 */
} sc7a20h_rawdata_t;

/* 函数声明 */
float sc7a20h_get_temperature(void);                                /* 获取传感器温度 */
uint8_t get_euler_angles(float *pitch, float *roll, float *yaw);    /* 获取欧拉角数据 */
void sc7a20h_read_xyz(float *acc, float *gyro);                     /* 获取加速度计和陀螺仪的三轴数据 */
void sc7a20h_read_rawdata(sc7a20h_rawdata_t *rawdata);              /* 读取原始数据 */
esp_err_t sc7a20h_init(void);
```

#### sc7a20h.c文件

```
/* 全局变量缓存区 */
i2c_master_dev_handle_t sc7a20h_handle = NULL;
const char* sc7a20h_name = "sc7a20h"; 
#define M_G                         9.80665f
#define RAD_TO_DEG                  (180.0f / M_PI)                         /* 0.017453292519943295 */
#define SC7A20H_AUTO_INCREMENT      0x80

/**
 * @brief       读取sc7a20h寄存器的数据
 * @param       reg_addr       : 要读取的寄存器地址
 * @param       data           : 读取的数据
 * @param       len           : 数据大小
 * @retval      错误值        ：0成功，其他值：错误
 */
esp_err_t sc7a20h_register_read(const uint8_t reg, uint8_t *data, const size_t len)
{
    uint8_t reg_addr = reg;

    if (len > 1)
    {
        reg_addr |= SC7A20H_AUTO_INCREMENT;
    }

    return i2c_master_transmit_receive(sc7a20h_handle, &reg_addr, 1, data, len, -1);
}

/**
 * @brief       向sc7a20h寄存器写数据
 * @param       reg_addr       : 要写入的寄存器地址
 * @param       data           : 要写入的数据
 * @retval      错误值        ：0成功，其他值：错误
 */
static esp_err_t sc7a20h_register_write_byte(uint8_t reg, uint8_t data)
{
    esp_err_t ret;

    uint8_t *buf = malloc(2);
    if (buf == NULL)
    {
        ESP_LOGE(sc7a20h_name, "%s memory failed", __func__);
        return ESP_ERR_NO_MEM;
    }

    buf[0] = reg;
    buf[1] = data;

    ret = i2c_master_transmit(sc7a20h_handle, buf, 2, -1);

    free(buf);

    return ret;
}

/**
 * @brief 读取单轴12位加速度值（左对齐，带符号扩展）
 * @param lsb_addr: 低字节寄存器地址 (e.g., 0x28)
 * @param msb_addr: 高字节寄存器地址 (e.g., 0x29)
 * @retval int16_t: 原始12位有符号值（单位：LSB，±2048 对应 ±2G）
 */
static int16_t sc7a20h_read_axis_12bit(uint8_t lsb_addr, uint8_t msb_addr)
{
    uint8_t lsb, msb;
    esp_err_t ret;

    /* 先读低字节 */
    ret = sc7a20h_register_read(lsb_addr, &lsb, 1);
    if (ret != ESP_OK) 
    {
        return 0;
    }

    ret = sc7a20h_register_read(msb_addr, &msb, 1);

    if (ret != ESP_OK) return 0;

    uint16_t temp = ((uint16_t)msb << 8) | lsb;

    temp >>= 4;

    if (msb & 0x80) 
    {
        temp |= 0xF000;  /* 补高4位为1（16位补码） */
    } 
    else 
    {
        temp &= 0x0FFF;  /* 清高4位（确保非负） */
    }

    return (int16_t)temp;
}

uint8_t xyz_data[6] = {0};
short raw_data[3] = {0};
float accl_data[3];
float acc_normal;
float scale_factor = 0.001f;  /* 默认±2G量程，1mg/digit */

/**
 * @brief       读取三轴数据(原始数据、加速度、俯仰角和翻滚角)
 * @param       rawdata：sc7a20h数据结构体
 * @retval      无
 */
void sc7a20h_read_rawdata(sc7a20h_rawdata_t *rawdata)
{
    float sensor_acc_x;
    float sensor_acc_y;
    float sensor_acc_z;

    if (sc7a20h_register_read(SC7A20H_REG_OUT_X_L, xyz_data, 6) != ESP_OK)
    {
        return;
    }

    /* 组合高低字节，SC7A20H为12位数据，左对齐 */
    raw_data[0] = (int16_t)(((uint16_t)xyz_data[1] << 8) | xyz_data[0]) >> 4;
    raw_data[1] = (int16_t)(((uint16_t)xyz_data[3] << 8) | xyz_data[2]) >> 4;
    raw_data[2] = (int16_t)(((uint16_t)xyz_data[5] << 8) | xyz_data[4]) >> 4;

    sensor_acc_x = (float)raw_data[0] * M_G / 1024.0f;
    sensor_acc_y = (float)raw_data[1] * M_G / 1024.0f;
    sensor_acc_z = (float)raw_data[2] * M_G / 1024.0f;

    rawdata->acc_x = sensor_acc_y;
    rawdata->acc_y = -sensor_acc_x;
    rawdata->acc_z = -sensor_acc_z;

    rawdata->acc_g = sqrt(rawdata->acc_x*rawdata->acc_x + rawdata->acc_y * rawdata->acc_y + rawdata->acc_z*rawdata->acc_z);

    acc_normal = sqrtf(rawdata->acc_x * rawdata->acc_x + rawdata->acc_y * rawdata->acc_y + rawdata->acc_z * rawdata->acc_z);
    if (acc_normal == 0.0f)
    {
        rawdata->pitch = 0.0f;
        rawdata->roll = 0.0f;
        return;
    }

    accl_data[0] = rawdata->acc_x / acc_normal;
    accl_data[1] = rawdata->acc_y / acc_normal;
    accl_data[2] = rawdata->acc_z / acc_normal;

    rawdata->pitch = atan2f(rawdata->acc_y, rawdata->acc_z) * RAD_TO_DEG;
    rawdata->roll = atan2f(rawdata->acc_x, rawdata->acc_z) * RAD_TO_DEG;
}

/**
 * @brief       配置自由落体检测
 * @param       threshold：阈值 (mg)
 * @param       duration：持续时间 (ODR周期数)
 * @retval      无
 */
void sc7a20h_config_freefall(uint8_t threshold, uint8_t duration)
{
    /* 配置自由落体阈值 (THS = threshold / 7.81mg) */
    uint8_t ths_value = (uint8_t)(threshold / 7.81f);
    sc7a20h_register_write_byte(0x32, ths_value); /* AOI1_THS */

    /* 配置自由落体持续时间 */
    sc7a20h_register_write_byte(0x33, duration); /* AOI1_DURATION */

    /* 配置中断源为自由落体 */
    sc7a20h_register_write_byte(0x30, 0x90); /* AOI1_CFG (Z低和Y低检测) */
}

/**
 * @brief       初始化sc7a20h
 * @param       无
 * @retval      0, 成功;
                1, 失败;
*/
uint8_t sc7a20h_config(void)
{
    uint8_t id_data = 0;

    /* 读取设备ID */
    sc7a20h_register_read(SC7A20H_REG_WHO_AM_I, &id_data, 1);

    /* 检查设备ID */
    if (id_data != SC7A20H_ID) 
    {
        ESP_LOGE("sc7a20h", "Device ID mismatch: expected 0x%02X, got 0x%02X", SC7A20H_ID, id_data);
        return 1;
    }

    /* 配置控制寄存器1: 100Hz ODR, 使能三轴, 正常模式 */
    uint8_t ctrl_reg1_val = SC7A20H_ODR_100HZ | SC7A20H_ENABLE_ALL_AXES;
    sc7a20h_register_write_byte(SC7A20H_REG_CTRL1, ctrl_reg1_val);

    /* 配置控制寄存器4: ±2G量程, 块数据更新使能 */
    uint8_t ctrl_reg4_val = SC7A20H_SCALE_2G | SC7A20H_BDU_ENABLE;
    sc7a20h_register_write_byte(SC7A20H_REG_CTRL4, ctrl_reg4_val);

    /* 配置为正常模式 */
    sc7a20h_register_write_byte(0x2E, 0x00); /* FIFO_CTRL_REG (禁用FIFO) */
    sc7a20h_register_write_byte(0x24, 0x00); /* CTRL_REG5 (禁用高通滤波器) */

    /* 设置量程对应的缩放因子 */
    scale_factor = 0.001f; /* ±2G量程，1mg/digit */

    ESP_LOGI("sc7a20h", "SC7A20H initialized successfully!");
    return 0;
}

/**
 * @brief       sc7a20h初始化
 * @param       无
 * @retval      无
 */
esp_err_t sc7a20h_init(void)
{
    /* 未调用myiic_init初始化IIC */
    if (bus_handle == NULL)
    {
        ESP_ERROR_CHECK(myiic_init());
    }

    i2c_device_config_t sc7a20h_i2c_dev_conf = {
        .dev_addr_length = I2C_ADDR_BIT_LEN_7,  /* 从机地址长度 */
        .scl_speed_hz    = IIC_SPEED_CLK,       /* 传输速率 */
        .device_address  = SC7A20H_ADDR,        /* 从机7位的地址 */
    };
    /* I2C总线上添加sc7a20h设备 */
    ESP_ERROR_CHECK(i2c_master_bus_add_device(bus_handle, &sc7a20h_i2c_dev_conf, &sc7a20h_handle));

    while (sc7a20h_config())   /* 检测不到sc7a20h */
    {
        ESP_LOGE("sc7a20h", "sc7a20h init fail!!!");
        vTaskDelay(500);
    }

    return 0;
}
```

程序通过IIC总线初始化设备，验证芯片ID后配置100Hz输出速率和±2g量程。核心功能包括读取12位原始加速度数据（左对齐格式），将其转换为标准重力单位（m/s²），并计算俯仰角和翻滚角。代码还实现了自由落体检测配置，通过设置阈值和持续时间参数触发中断。数据处理采用符号扩展确保12位有符号数正确解析，同时支持自动递增寄存器地址以提高读取效率。

### CMakeLists.txt文件

打开本实验的BSP文件夹下的CMakeList.txt文件，其内容如下所示：

```
set(src_dirs
            MYIIC
            LCD
            MYSPI
            AW9523B
            SC7A20H)

set(include_dirs
            MYIIC
            LCD
            MYSPI
            AW9523B
            SC7A20H)

set(requires
            driver
            esp_lcd)

idf_component_register(SRC_DIRS ${src_dirs} INCLUDE_DIRS ${include_dirs} REQUIRES ${requires})

component_compile_options(-ffast-math -O3 -Wno-error=format=-Wno-format)
```

上述代码中的 SC7A20H 驱动需要由开发者自行添加，以确保 SC7A20H 驱动能够顺利集成到构建系统中。这一步骤是必不可少的，它确保了 SC7A20H 驱动的正确性和可用性，为后续的开发工作提供了坚实的基础。

### 实验应用代码

打开main.c文件，该文件定义了工程入口函数，名为main。该函数代码如下。

```
/**
 * @brief       显示原始数据
 * @param       x, y : 坐标
 * @param       title: 标题
 * @param       val  : 值
 * @retval      无
 */
void user_show_mag(uint16_t x, uint16_t y, char *title, float val)
{
    char buf[20];

    sprintf(buf,"%s%3.1f", title, val);                 /* 格式化输出 */
    lcd_fill(x + 30, y + 16, x + 160, y + 16, WHITE);   /* 清除上次数据(最多显示20个字符,20*8=160) */
    lcd_show_string(x, y, 160, 16, 16, buf, BLUE);      /* 显示字符串 */
}

/**
 * @brief       程序入口
 * @param       无
 * @retval      无
 */
void app_main(void)
{
    esp_err_t ret;
    uint8_t t = 0;
    sc7a20h_rawdata_t xyz_rawdata;
    
    ret = nvs_flash_init();             /* 初始化NVS */
    if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND)
    {
        ESP_ERROR_CHECK(nvs_flash_erase());
        ESP_ERROR_CHECK(nvs_flash_init());
    }

    my_spi_init();                      /* 初始化SPI */
    myiic_init();                       /* 初始化IIC */
    aw9523b_init();                     /* 初始化AW9523B */
    lcd_init();                         /* 初始化LCD */
    sc7a20h_init();                     /* 初始化SC7A20H */

    lcd_show_string(30, 50, 200, 16, 16, "ESP32-S3", RED);
    lcd_show_string(30, 70, 200, 16, 16, "SC7A20H TEST", RED);
    lcd_show_string(30, 90, 200, 16, 16, "ATOM@ALIENTEK", RED);
    
    lcd_show_string(30, 110, 200, 16, 16, " ACC_X :", RED);
    lcd_show_string(30, 130, 200, 16, 16, " ACC_Y :", RED);
    lcd_show_string(30, 150, 200, 16, 16, " ACC_Z :", RED);
    lcd_show_string(30, 170, 200, 16, 16, " Pitch :", RED);
    lcd_show_string(30, 190, 200, 16, 16, " Roll  :", RED);

    while (1)
    {
        vTaskDelay(pdMS_TO_TICKS(10));
        t++;

        if (t == 10)
        {   
            sc7a20h_read_rawdata(&xyz_rawdata);
            
            user_show_mag(30, 110, "ACC_X :", xyz_rawdata.acc_x);
            user_show_mag(30, 130, "ACC_Y :", xyz_rawdata.acc_y);
            user_show_mag(30, 150, "ACC_Z :", xyz_rawdata.acc_z);
            user_show_mag(30, 170, "Pitch :", xyz_rawdata.pitch);
            user_show_mag(30, 190, "Roll  :", xyz_rawdata.roll);
            
            t = 0;
            LEDR_TOGGLE();
        }
    }
}
```

从上述源码可知，我们首先初始化各个外设，如IIC、 SPI、 XL9555、 SC7A20H和LCD等驱动，然后调用 sc7a20h_read_rawdata ()函数测量数据，最终计算出pitch俯仰角和roll翻滚角数据，并在LCD上显示。LED灯每隔100毫秒状态翻转，实现闪烁效果。

## 下载验证

将程序下载到开发板后，实验内容如下图所示：

<img src={require('./img/XYZ.png').default} alt="00" width="380" />
