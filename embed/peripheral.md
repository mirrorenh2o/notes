## 常识
一些外设、驱动、协议的小常识总结：
1. 设备方面：硬件设备——外设/硬件接口 ——驱动实现
- 外设/硬件接口：具有同一/相似功能的外设是多样化的(如摄像头等产品型号多样)，厂商需要根据自己的需求进行选择（如体积、功耗、价格、兼容性等）
- 驱动实现：如果厂商选择的是主流的外设，这个外设驱动内核已经支持的，厂商可以不用自己实现驱动，而是直接把内核已支持的设备加入到设备树中；如果选择的比较小众，就需要厂商自己实现驱动。

2. 系统层面：接口的类型——接口类型对应的不同通信协议——系统是否对应的协议软件编译进了系统软件栈
- 通信协议是跟具体的接口设备型号无关，只跟其类型有关。（如A设备使用的是网卡1，B设备使用的网卡2;但是只要是以太网网卡，一般都支持以太网网络通信协议）
- 实际系统是否支持某个功能/能力：还要看相关的软件是否被编译进了系统软件栈，就是系统中要有这个软件。

## 调研表

### **表一：RISC-V设备外设与驱动支持**
#### **字段说明**
| **字段名**         | **说明**                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Device Model**   | 芯片/开发板型号（如Allwinner D1、Kendryte K210）                       |
| **Vendor**        | 厂商名称（如Allwinner、Canaan）                                        |
| **Peripheral**    | 外设类型（如Ethernet、USB、UART）                                      |
| **HW Interface**  | 硬件接口标准（如SDIO、SPI、RMII）                                      |
| **Driver Name**   | 内核中的驱动名称（如`dwmac`、`sunxi-musb`）                            |
| **Mainline**      | 是否被Linux主线支持（✅/❌）                                            |
| **Notes**         | 特殊说明（如依赖补丁、需配置设备树）                                   |

#### **示例数据**
| Device Model       | Vendor     | Peripheral | HW Interface | Driver Name       | Mainline | Notes                          |
|--------------------|------------|------------|--------------|-------------------|----------|--------------------------------|
| Allwinner D1       | Allwinner  | Ethernet   | RMII         | sun8i-emac        | ✅        | 需设备树配置                  |
| Kendryte K210      | Canaan     | UART       | UART Lite    | dw_apb_uart       | ✅        | 默认启用                      |
| SiFive FU740       | SiFive     | PCIe       | PCIe Gen2    | fu740-pcie        | ✅        | 需PHY驱动                     |
| StarFive JH7110    | StarFive   | USB 3.0    | USB3.0       | jh7110-usb        | ✅        | 依赖PHY配置                   |

---

### **表二：外设类型、通信协议及协议软件**
#### **字段说明**
| **字段名**         | **说明**                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Peripheral Type**| 外设类型（如Ethernet、USB、UART）                                      |
| **Protocol**       | 支持的通信协议（如TCP/IP、MTP、YMODEM）                                |
| **Protocol Stack** | 实现协议的软件（如LwIP、Linux内核协议栈）                              |
| **Default in OS**  | 是否默认集成在主流OS中（Linux/RTOS）                                   |
| **Configuration**  | 启用协议所需配置（如内核编译选项、RTOS组件）                           |

#### **完整数据（主流外设与协议）**
| Peripheral Type | Protocol               | Protocol Stack             | Default in OS       | Configuration                          |
|-----------------|------------------------|----------------------------|---------------------|----------------------------------------|
| **Ethernet**    | TCP/IP, HTTP, MQTT     | Linux内核协议栈, LwIP      | Linux: ✅, RTOS: ❌  | Linux: `CONFIG_NET=y`, RTOS: 手动移植  |
| **Wi-Fi**       | 802.11, WPA2, TLS      | wpa_supplicant, LwIP       | Linux: ✅, RTOS: ❌  | 需Wi-Fi驱动 + 固件（如`brcmfmac`）     |
| **USB Device**  | MSC, MTP, RNDIS        | Linux Gadget API, TinyUSB  | Linux: ✅, RTOS: ❌  | 启用`CONFIG_USB_CONFIGFS`              |
| **UART**        | YMODEM, XMODEM, ASCII  | lrzsz, 自定义协议          | Linux: ✅, RTOS: ❌  | 需用户态工具（如`minicom`）            |
| **SPI/I2C**     | 自定义二进制协议       | 无标准协议栈               | ❌                  | 需厂商实现驱动 + 应用层协议             |
| **LoRa**        | LoRaWAN                | LoRaMAC-node, ChirpStack   | RTOS: ❌             | 需移植Semtech协议栈 + 射频驱动          |
| **Bluetooth**   | BLE, GATT              | BlueZ (Linux), NimBLE      | Linux: ✅, RTOS: ❌  | Linux: `CONFIG_BT=y`, RTOS: 集成SDK    |
| **SD Card**     | SD/MMC协议             | Linux MMC子系统            | Linux: ✅, RTOS: ❌  | 启用`CONFIG_MMC`                       |

---

### **关键结论**
1. **硬件与驱动的解耦**  
   - 同一外设类型（如Ethernet）的协议（TCP/IP）是通用的，但驱动需适配具体硬件（如`dwmac` vs `sun8i-emac`）。  
   - **厂商选择主流外设（如Realtek网卡）可减少驱动开发成本**。

2. **协议栈的通用性**  
   - 标准协议（如USB MSC、TCP/IP）的软件实现（如Linux内核、LwIP）是通用的，但需在系统中编译或移植。  
   - **Linux开箱即用，RTOS需手动集成**。

3. **你的表格设计改进建议**  
   - **表一**：增加`Vendor`字段区分厂商，补充`Mainline`状态。  
   - **表二**：明确协议栈与OS的默认支持关系，区分Linux/RTOS差异。

---

### **附：主流RISC-V外设驱动状态（补充）**
| Chip Model       | Ethernet Driver     | USB Driver        | Notes                     |
|------------------|---------------------|-------------------|---------------------------|
| Allwinner D1     | sun8i-emac          | sunxi-musb        | 主线支持                  |
| StarFive JH7110  | jh7110-dwmac        | jh7110-usb        | 需更新内核到6.6+          |
| Sophgo CV1800B   | dwmac-3.70a         | dwc3              | 依赖社区补丁              |

如果需要更详细的某类外设（如USB协议栈）实现分析，可以进一步展开。
