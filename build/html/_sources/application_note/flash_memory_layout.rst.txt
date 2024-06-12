.. _flash_layout:

Flash Layout
------------------------
This chapter introduces the default Flash layout of |CHIP_NAME| and how to modify the Flash layout if needed.

Introduction
~~~~~~~~~~~~~~~~~~~~~~~~
The default Flash layout used in SDK is illustrated in Figure 1\-1 and Table 1\-1. The layout takes the 8MB Flash as an example. The start address of boot manifest is fixed to 0x0800_0000, and other start addresses can be configured by users flexibly.



Figure 1\-1 Flash layout

Table 1\-1 Flash layout

.. only:: RTL8721D
    
    
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | Item                         | Physical address | Size (KB) | Description                                                                                           | Mandatory |
    +==============================+==================+===========+=======================================================================================================+===========+
    | KM4 Bootloader manifest      | 0x0800_0000      | 4         | KM4 bootloader manifest                                                                               | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | KM4 Bootloader               | 0x0800_1000      | 76        | KM4 bootloader (code/data), containing KM4 bootloader IMG, mapped to the virtual address 0x0F80_0000. | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | Key Certificate              | 0x0801_4000      | 4         | Public key hash information for other images.                                                         | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | IMG2 manifest                | 0x0801_5000      | 4         | KM0 & KM4 Application & KM4 IMG3 manifest                                                             | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | KM0 & KM4 Application        | 0x0801_6000      | 1912      | Combines KM0 image, KM4 image2 and KM4 image3                                                         | √         |
    |                              |                  |           |                                                                                                       |           |
    |                              |                  |           | - KM0_IMG: KM0 image (code/data), mapped to the virtual address 0x0C00_0000.                          |           |
    |                              |                  |           |                                                                                                       |           |
    |                              |                  |           | - KM4_IMG2: KM4 non\-secure image (code/data), mapped to the virtual address 0x0E00_0000.             |           |
    |                              |                  |           |                                                                                                       |           |
    |                              |                  |           | - KM4_IMG3: secure image (code/data)                                                                  |           |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | KM4 Bootloader manifest OTA2 | 0x0820_0000      | 4         | KM4 bootloader manifest                                                                               | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | KM4 Bootloader OTA2          | 0x0820_1000      | 76        | KM4 bootloader (code/data), containing KM4 bootloader IMG, mapped to the virtual address 0x0F80_0000. | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | Key Certificate OTA2         | 0x0821_4000      | 4         | Public key hash information for other images.                                                         | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | IMG2 manifest OTA2           | 0x0821_5000      | 4         | KM0 & KM4 Application & KM4 IMG3 manifest                                                             | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | KM0 & KM4 Application OTA2   | 0x0821_6000      | 1912      | Combines KM0 image, KM4 image2 and KM4 image3                                                         | √         |
    |                              |                  |           |                                                                                                       |           |
    |                              |                  |           | - KM0_IMG: KM0 image (code/data), mapped to the virtual address 0x0C00_0000.                          |           |
    |                              |                  |           |                                                                                                       |           |
    |                              |                  |           | - KM4_IMG2: KM4 non\-secure image (code/data), mapped to the virtual address 0x0E00_0000.             |           |
    |                              |                  |           |                                                                                                       |           |
    |                              |                  |           | - KM4_IMG3: secure image (code/data)                                                                  |           |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | FTL                          | 0x0870_0000      | 12        | For Bluetooth                                                                                         | ×         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | VFS                          | 0x0870_3000      | 128       | For file system                                                                                       | ×         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+



.. only:: RTL8711D
    
    
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | Item                         | Physical address | Size (KB) | Description                                                                                           | Mandatory |
    +==============================+==================+===========+=======================================================================================================+===========+
    | KM4 Bootloader manifest      | 0x0800_0000      | 4         | KM4 bootloader manifest                                                                               | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | KM4 Bootloader               | 0x0800_1000      | 76        | KM4 bootloader (code/data), containing KM4 bootloader IMG, mapped to the virtual address 0x0F80_0000. | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | Key Certificate              | 0x0801_4000      | 4         | Public key hash information for other images.                                                         | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | IMG2 manifest                | 0x0801_5000      | 4         | KM0 & KM4 Application manifest                                                                        | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | KM0 & KM4 Application        | 0x0801_6000      | 1912      | Combines KM0 image and KM4 image2                                                                     | √         |
    |                              |                  |           |                                                                                                       |           |
    |                              |                  |           | - KM0_IMG: KM0 image (code/data), mapped to the virtual address 0x0C00_0000.                          |           |
    |                              |                  |           |                                                                                                       |           |
    |                              |                  |           | - KM4_IMG2: KM4 image (code/data), mapped to the virtual address 0x0E00_0000.                         |           |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | KM4 Bootloader manifest OTA2 | 0x0820_0000      | 4         | KM4 bootloader manifest                                                                               | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | KM4 Bootloader OTA2          | 0x0820_1000      | 76        | KM4 bootloader (code/data), containing KM4 bootloader IMG, mapped to the virtual address 0x0F80_0000. | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | Key Certificate OTA2         | 0x0821_4000      | 4         | Public key hash information for other images.                                                         | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | IMG2 manifest OTA2           | 0x0821_5000      | 4         | KM0 & KM4 Application manifest                                                                        | √         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | KM0 & KM4 Application OTA2   | 0x0821_6000      | 1912      | Combines KM0 image and KM4 image2                                                                     | √         |
    |                              |                  |           |                                                                                                       |           |
    |                              |                  |           | - KM0_IMG: KM0 image (code/data), mapped to the virtual address 0x0C00_0000.                          |           |
    |                              |                  |           |                                                                                                       |           |
    |                              |                  |           | - KM4_IMG2: KM4 image (code/data), mapped to the virtual address 0x0E00_0000.                         |           |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | FTL                          | 0x0870_0000      | 12        | For Bluetooth                                                                                         | ×         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+
    | VFS                          | 0x0870_3000      | 128       | For file system                                                                                       | ×         |
    +------------------------------+------------------+-----------+-------------------------------------------------------------------------------------------------------+-----------+





.. note::
   The reserved space can be used by users.


Memory Management Unit (MMU)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
To achieve flexibility of image and for image encryption when RSIP is enabled (the fixed address is needed by IV when doing image encryption, refer to RSIP for more information), Flash MMU is applied by default. The default MMU layout used in SDK is illustrated in Figure 1\-2.



Figure 1\-2 Flash MMU layout

How to Modify Flash Layout
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The following locations in the Flash can be modified:

- Bootloader OTA2 location

- APP location

   - APP OTA1

   - APP OTA2

- FTL/VFS Location

Modifying Bootloader OTA2 Location
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The location of Bootloader OTA2 can be modified but requires 4KB alignment. The Bootloader OTA2 is disabled by default. If you want to enable Bootloader OTA2, shift the start address of Bootloader OTA2 right by 12 bits, then burn it to the OTP 0x36C~0x36D.


The method is to modify the address of \ ``IMG_BOOT_OTA2``\  in \ ``{SDK}\component\soc\amebadplus\usrcfg\ameba_flashcfg.c``\ . After burning the Bootloader OTA2 into Flash through OTA, Boot ROM will decide whether to use Bootloader OTA1 or Bootloader OTA2 according to the version number.

.. only:: internal
    
    
    BootLoader OTA2的位置可以修改，修改后的位置需要4KB对齐。默认状态下，BootLoader OTA2是不使能的，如果需要使能BootLoader OTA2，将BootLoader OTA2的起始地址右移12位之后，烧写进Efuse的0x36C~0x36D。
    
    修改component\rtl8720e\usrcfg\ameba_flashcfg.c中如下红色区域，通过OTA将BootLoader OTA2烧写进Flash后，BootRom根据版本号来决定使用BootLoader OTA1还是BootLoader OTA2。
    


.. image:: ../_static/flash_memory_layout_rst/a8921a4f08d70fca0ef51dd3cf89af03c828f1e5.png
   :width: 1091
   :align: center
 

If the anti\-rollback function is enabled to ensure that the version of the Bootloader can only be incrementing but cannot be rolled back, the version of the Bootloader needs to be changed before compiling. That is, IMG_VER in \ ``{SDK}\amebadplus_gcc_project\manifest.json``\  needs to be modified.

.. image:: ../_static/flash_memory_layout_rst/bf0452ef5426e69a5bed57e287657aacdd7fb09c.png
   :width: 365
   :align: center




.. note::
   The location of Bootloader OTA1 is fixed to 0x0800_0000, and cannot be modified.


Modifying APP Location
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
APP OTA1
****************
Follow the steps to modify the location of APP OTA1:

1. Modify the address of IMG_APP_OTA1 in \ ``{SDK}\component\soc\amebadplus\usrcfg\ameba_flashcfg.c``\ .

.. image:: ../_static/flash_memory_layout_rst/1b23f396dfc1e8f895577c1286fce88d55447c93.png
   :width: 1163
   :align: center


2. Re\-build the project to generate the Bootloader and APP OTA1.

3. Modify the address of \ ``km0_km4_app.bin``\  if you update the location of APP OTA1 through ImageTool, and download the new Bootloader and APP OTA1.

.. image:: ../_static/flash_memory_layout_rst/d0357ca252c907fe084a97606147dd5ec1c0a404.png
   :width: 1414
   :align: center


After that, Bootloader will load the image from the new location of APP OTA1 if the version of APP OTA1 is bigger.

APP OTA2
****************
1. Modify the address of \ ``IMG_APP_OTA2``\  in \ ``{SDK}\component\soc\amebadplus\usrcfg\ameba_flashcfg.c``\ .

2. Re\-build and download the new Bootloader and APP OTA2 as described in section 1.3.2.1 step (2)~(3).

.. image:: ../_static/flash_memory_layout_rst/85e2ba3eda65b3e93392cc252d88ef84e537b095.png
   :width: 1092
   :align: center


After burning the APP OTA2 into Flash through OTA, Bootloader will load the image from the new location of APP OTA2 if the version of APP OTA2 is bigger.

Modifying FTL/VFS Location
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
1. Modify the addresses of FTL and VFS1 in \ ``{SDK}\component\soc\amebadplus\usrcfg\ameba_flashcfg.c``\ .

2. Update the application image.

.. image:: ../_static/flash_memory_layout_rst/c69810c4d2f35aa46f86d2b86c75751116554a8d.png
   :width: 1093
   :align: center


Flash Protect Enable
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Before loading APP IMG, the Bootloader will read the Status Register from Flash. If only Quad Enable (QE) Bit is set in the output of bitwise AND between Status Register of Flash and status_mask in Flash_AVL ({SDK}\component\soc\amebadplus\usrcfg\ameba_flashcfg.c), do nothing, or the output of bitwise AND will be written to the Flash Status Register.



.. note::
   By default, setting the QE bit will unlock all the Block Protect Bits. To avoid this operation, set Block Protect bits corresponding to Status_mask in Flash_AVL to 0. For example, change the Status_mask of Winbond in the Flash_AVL to 0x000043C0.

.. only:: internal
    
    
    BootLoader加载IMG2之前，BootLoader会读取Flash的Status Register，并与component\soc\amebadplus\usrcfg\ameba_flashcfg.c中Flash_AVL的Status_mask值进行按位与操作，如果按位与的结果只有Quad Enable(QE) Bit置位，那么不进行任何操作，否则将按位与的结果写入Flash的Status Register。
    
    
    
    .. note::
       默认状态下，设置QE Bit时会解锁所有的Block Protect Bit，如需避免该操作，将Flash_AVL中Status_mask对应的Block Protect Bits设置0，例如将Flash_AVL中winbond的Status_mask修改为0x000043C0。
    



.. image:: ../_static/flash_memory_layout_rst/06ad55f13b1f04b9cba13d7d3f568a946e88b336.png
   :width: 1128
   :align: center


In order to avoid the image being damaged due to improper operation when using LittleFS to write user data, it is recommended to modify the location of FTL/LittleFS to the last 64KB area of Flash, and set the Block Protect Bit in the Status Register of Flash at the same time.



.. note::
      - Only the last 64KB area of Flash can be modified, and the other areas are protected. Remember to unlock the Flash during OTA upgrade, and keep it locked when OTA is completed.

      - For some Flashes, you cannot set the Flash to allow only the last block to be modified through Block Protect Bit. In this case, it is recommended to enable the Flash block protection of the first half part.


.. _memory_layout:

Memory Layout
--------------------------
This chapter introduces the default memory layout of |CHIP_NAME| and how to modify the memory layout if needed.

RAM Layout
~~~~~~~~~~~~~~~~~~~~
In total, there are 512KB SRAM on chip, and the size of PSRAM can be 0MB/4MB/8MB/16MB…, which is decided by users. The RAM layout is illustrated in Figure 2\-1.



Figure 2\-1 RAM layout

SRAM0 (First 40KB) Layout
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The first 40KB SRAM0 layout is illustrated in Table 2\-1Figure 2\-2 and Table 2\-1. It is the same for all situations.



Figure 2\-2 SRAM0 (first 40KB) layout

Table 2\-1 SRAM0 (first 40KB) layout

+----------------------+---------------+--------+-------------------------------------------+-----------+
| Items                | Start address | Size   | Description                               | Mandatory |
+======================+===============+========+===========================================+===========+
| KM0_ROM_BSS_RAM      | 0x2000_0000   | 4KB    | KM0 ROM BSS                               | √         |
+----------------------+---------------+--------+-------------------------------------------+-----------+
| KM0_MSP_RAM          | 0x2000_1000   | 4KB    | KM0 Main Stack Pointer                    | √         |
+----------------------+---------------+--------+-------------------------------------------+-----------+
| KM0_STDLIB_HEAP_NS   | 0x2000_2000   | 4KB    | KM0 ROM STDLIB heap                       | √         |
+----------------------+---------------+--------+-------------------------------------------+-----------+
| KM4_MSP_NS           | 0x2000_3000   | 4KB    | KM4 non\-secure Main Stack Pointer        | √         |
+----------------------+---------------+--------+-------------------------------------------+-----------+
| KM4_ROM_BSS_COMMON   | 0x2000_4000   | 3.25KB | KM4 ROM secure and non\-secure common BSS | √         |
+----------------------+---------------+--------+-------------------------------------------+-----------+
| KM0_BOOT_RAM         | 0x2000_4D00   | 64B    | KM0 IMG2 entry                            | √         |
+----------------------+---------------+--------+-------------------------------------------+-----------+
| KM0_IPC_RAM          | 0x2000_4E00   | 512B   | Exchange messages between cores           | √         |
+----------------------+---------------+--------+-------------------------------------------+-----------+
| KM4_ROM_BSS_NS       | 0x2000_5000   | 4KB    | KM4 ROM non\-secure common BSS            | √         |
+----------------------+---------------+--------+-------------------------------------------+-----------+
| KM4_STDLIB_HEAP_NS   | 0x2000_6000   | 4KB    | KM4 ROM non\-secure STDLIB heap           | √         |
+----------------------+---------------+--------+-------------------------------------------+-----------+
| KM4_ROM_BSS_S        | 0x3000_7000   | 4KB    | KM4 ROM secure\-only BSS                  | √         |
+----------------------+---------------+--------+-------------------------------------------+-----------+
| KM0_RTOS_STATIC_0_NS | 0x2000_8000   | 4KB    | KM0 RTOS static pool position             | √         |
+----------------------+---------------+--------+-------------------------------------------+-----------+
| KM4_MSP_S            | 0x3000_9000   | 4KB    | KM4 secure Main Stack Pointer             | √         |
+----------------------+---------------+--------+-------------------------------------------+-----------+

.. only:: RTL8721D
    
    RAM & PSRAM Layout
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    There are 288KB SRAM for KM4 and 96KB SRAM for KM0, which can be used for Power Management Controller (PMC) code and performance\-cared text and data. Figure 2\-4 and Table 2\-3 illustrate the RAM layout with PSRAM.
    
    
    
    Figure 2\-3 RAM layout (with PSRAM)
    
    Table 2\-2 RAM layout (with PSRAM)
    
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | Item           | Start address | Size (KB) | Description                                                       | Mandatory |
    +================+===============+===========+===================================================================+===========+
    | SRAM0          | 0x2000_0000   | 40        | For ROM BSS, MSP, …                                               | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM4 Bootloader | 0x3000_A000   | 24        | KM4 secure bootloader, including code and data                    | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM4 IMG3       | 0x2007_0000   | 64        | KM4 IMG3, can be recycled if IMG3 is not needed                   | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM4 BDRAM      | 0x2001_0000   | 288       | KM4 BDRAM data, BSS and heap                                      | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM0 BDRAM      | 0x2006_8000   | 96        | KM0 BDRAM data, BSS and heap                                      | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM4 PSRAM      | 0x6000_0000   | 3220      | KM4 PSRAM code, can be empty                                      | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM0 PSRAM      | 0x6032_5000   | 876       | KM0 PSRAM code, can be empty                                      | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM4 HEAP EXT   | 0x6FFF_FFFF   | 0         | If KM4 heap is not enough, it can be used to extend the heap size | x         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM0 HEAP EXT   | 0x6FFF_FFFF   | 0         | If KM0 heap is not enough, it can be used to extend the heap size | x         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+

.. only:: RTL8711D
    
    RAM & PSRAM Layout
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    There are 352KB SRAM for KM4 and 96KB SRAM for KM0, which can be used for Power Management Controller (PMC) code and performance\-cared text and data. Figure 2\-5 and Table 2\-4 illustrate the RAM layout with PSRAM.
    
    
    
    Figure 2\-4 RAM layout (with PSRAM)
    
    Table 2\-3 RAM layout (with PSRAM)
    
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | Item           | Start address | Size (KB) | Description                                                       | Mandatory |
    +================+===============+===========+===================================================================+===========+
    | SRAM0          | 0x2000_0000   | 40        | For ROM BSS, MSP, …                                               | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM4 Bootloader | 0x3000_A000   | 24        | KM4 secure bootloader, including code and data                    | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM4 BD RAM     | 0x2001_0000   | 352       | KM4 BDRAM data, BSS and heap                                      | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM0 BD RAM     | 0x2004_0000   | 96        | KM0 BDRAM data, BSS and heap                                      | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM4 PSRAM      | 0x6000_0000   | 3220      | KM4 PSRAM code, can be empty                                      | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM0 PSRAM      | 0x6032_5000   | 876       | KM0 PSRAM code, can be empty                                      | √         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM4 HEAP EXT   | 0x6FFF_FFFF   | 0         | If KM4 heap is not enough, it can be used to extend the heap size | x         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+
    | KM0 HEAP EXT   | 0x6FFF_FFFF   | 0         | If KM0 heap is not enough, it can be used to extend the heap size | x         |
    +----------------+---------------+-----------+-------------------------------------------------------------------+-----------+

.. only:: RTL8721D or nda
    
    TrustZone Layout
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    There are only eight SAU entries to set memory to non\-secure or non\-secure callable attribution. To maintain the flexibility of the system, we resolve the problem of insufficient SAU entries through the method described below:
    
    - IDAU partitions the address space of adjacent 512MB into two parts, secure and non\-secure, according to the address bit[28].
    
    - The two secure/non\-secure address spaces are mapped to the same physical memory, which is address aliasing. Secure address space provides the window of secure access to this physical memory, and non\-secure address space provides the window of non\-secure access to it.
    
    
    
    Figure 2\-5 Result of IDAU conjunction with SAU
    
    Figure 2\-7 illustrates the TrustZone layout. Refer to User Manual for more information about TrustZone.
    
    
    
    Figure 2\-6 TrustZone layout
    
    
    
    .. note::
       Each NSC entry in NSC area consists of two instructions sg and b.w together. The encoding of b.w instruction restricts the destination branch address, which cannot be too far away, such as 0x2XXX_XXXX and 0x3XXX_XXXXs. Therefore, the |CHIP_NAME| will allocate a secure area (KM4_BD_RAM_ENTRY) near to NSC area (0x2XXX_XXXX) to place the NSC function. To see more details, refer to the TrustZone specification.
    

How to Modify Memory Layout
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The following memory size can be modified.

- Bootloader

- BD_RAM

- BD_PSRAM

- Heap

Modifying Bootloader Size
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
If you need to enlarge the size of KM4_BOOT_RAM_S, the modified KM4_BOOT_RAM_S size should be 4KB aligned because the MPC protection is protected in unit 4KB.


Follow the steps to modify the size of Bootloader:

1. Modify the CONFIG Link Option in menuconfig to choose whether to place the Bootloader (IMG1) on FLASH or SRAM. When SRAM is selected, the size of KM4_BOOTLOADER_RAM_S is 24K; when FLASH is selected, the size of KM4_BOOTLOADER_RAM_S is 4K.

.. image:: ../_static/flash_memory_layout_rst/317a7635d2df4eaa61d78e8bd0fb8e748bbaa79a.png
   :width: 455
   :align: center


2. By modifying KM4_IMG1_SIZE in \ ``{SDK}\amebadplus_gcc_project\amebaDplus_layout.ld``\ , users can change the size of the KM4 BOOTLOADER RAM S.

.. image:: ../_static/flash_memory_layout_rst/a27a4eb43a21e2e823270f1862da74effe17d3e5.png
   :width: 408
   :align: center


3. Re\-build the project to generate the Bootloader.

4. Modify the end address of km4_boot_all.bin if Bootloader is too large, and download the new Bootloader.

.. image:: ../_static/flash_memory_layout_rst/4ed6c24886cfdf50ebec90c4fe8a899a6883167d.png
   :width: 1821
   :align: center


After that, Boot ROM will load the new Bootloader if the version of new Bootloader is bigger.

Modifying BD_RAM Size
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Follow the steps to modify the size of KM4 BD RAM:

1. Users can change the KM4 BD RAM size by modifying RAM_KM0_IMG2_SIZE in \ ``{SDK}\amebadplus_gcc_project\amebaDplus_``\  \ ``layout.ld``\  to change the end address of KM4_BD_RAM.

.. image:: ../_static/flash_memory_layout_rst/c014baacb7498076fda32fee1380dce15b8f3d39.png
   :width: 399
   :align: center


2. Re\-build and download the new Bootloader and IMG2 OTA2 as described in Section 1.3.2.1 step (2)~(3).

Modifying BD_PSRAM Size
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
If user wants to modify the KM4 BD PSRAM size, please modify the running position of IMG2 in menuconfig first. Any option with PSRAM is acceptable. Then modify PSRAM_KM4_IMG2_SIZE in \ ``{SDK}\amebadplus_gcc_project\amebaDplus_layout.ld``\  to change the end address of KM4_BD_PSRAM.

.. image:: ../_static/flash_memory_layout_rst/68e1672a9be0e0bdb3ec171e30264a4bddbe51ae.png
   :width: 985
   :align: center


Extending Heap Size
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The heap size consists of multi\-blocks and is passed to the operating system by the os_heap_init() function in component\os\freertos\ freertos_heap5_config.c. By default, PSRAM_HEAP1_START is invalid address and PSRAM_HEAP1_SIZE is 0.

.. image:: ../_static/flash_memory_layout_rst/a92d39c85a6c25045fb56f51b199e3ca3bc25a9f.png
   :width: 1020
   :align: center
 

If the heap of KM4 is not enough, define Heap Start and Heap Size for some unused areas in amebaDplus_layout.ld, and then use the os_heap_add function to add the area to the heap array. The address shall be a valid value in PSRAM, then re\-build and download the new image to let KM4 use the extended heap.

.. code::

   bool os_heap_add(u8 *start_addr, size_t heap_size);


.. note::
   The symbols defined in linker script (amebaDplus_layout.ld) need to be declared in ameba_boot.h before they can be used.



If KM0 heap is not enough, modify the KM0_PSRAM_HEAP_EXT accordingly.

