.. _user_configuration:

User Configuration
------------------------------------
Introduction
~~~~~~~~~~~~~~~~~~~~~~~~
The KM4 in |CHIP_NAME| device boots at 200MHz at the BootRom Stage, and switches to a higher frequency during the BootLoader Stage. There are some limits when changing SoC clock:


+-------+-------+-----------------+--------------+
| Clock | Cut   | Frequency       | Core voltage |
+=======+=======+=================+==============+
| PLL   |       | 300MHz ~ 600MHz | -            |
+-------+-------+-----------------+--------------+
| KM0   | A-Cut | ≤115MHz         | -            |
+-------+-------+-----------------+--------------+
| KM4   | A-Cut | ≤260MHz         | 0.9V         |
+-------+-------+-----------------+--------------+
| KM4   | A-Cut | ≤345MHz         | 1.0V         |
+-------+-------+-----------------+--------------+



.. note::
   The maximum operating speed of Flash with Wide Range VCC 1.65V~3.6V should use the speed limit of 1.65V~2.3V power supply.

.. only:: internal
    
    
    Flash使用1.8V供电时，只支持pad为strong driving的情况。
    


SoC Clock Switch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Flow
^^^^^^^^
1. (Optional) Find out the speed limit of PSRAM device embedded in |CHIP_NAME| if not sure.

   a. Print the value of ChipInfo_BDNum() function, which will get the chip info from OTP.

   b. Refer to PSRAM type in Chip_Info[] in \ ``{SDK}\component\soc\amebadplus\lib\ram_common\ameba_chipinfo_lib.c``\ .

For Now, This step can be skipped because wb955 is only used.

<<<<<<< HEAD
2. Check the value of Boot_SocClk_Info_Idx and SocClk_Info[] in \ ``{SDK}\component\soc\amebadplus\usrcfg\ameba_``\  \ ``bootcfg.c``\ .
=======
2. Check the value of Boot_SocClk_Info_Idx and SocClk_Info[] in \ ``{SDK}\component\soc\amebadplus\usrcfg\ameba_ bootcfg.c``\ .
>>>>>>> 0e8663f (update files)

.. image:: ../_static/user_config_rst/418472eafdc889c9bfa2a32acb00e811f3748ec1.png
   :width: 1046
   :align: center


3. Check the BOOT_ChipInfo_ClkInfoIdx() function in \ ``{SDK}\component\soc\amebadplus\bootloader\bootloader_km4.c``\ .

.. image:: ../_static/user_config_rst/5b5354622e7be590f629af9c1c610ea6f8fc4d5e.png
   :width: 869
   :align: center


No Limitation by PSRAM Divice , so BootLoader will set the SoC clock defined by SocClk_Info[Boot_SocClk_Info_Idx ].

+------------+-------------+---------------------------+---------------------------------------------------------------+-----------------+
| PSRAM type | PSRAM speed | SocClk_Info[x]            | Description                                                   | Clock Info      |
+============+=============+===========================+===============================================================+=================+
| No PSRAM   | -           | Boot_SocClk_Info_Idx = 0; | BootLoader will set the Soc clock according to SocClk_Info[0] | PLL: 520MHz     |
|            |             |                           |                                                               |                 |
|            |             |                           |                                                               | KM4: 260MHz     |
|            |             |                           |                                                               |                 |
|            |             |                           |                                                               | KM0: 86.6MHz    |
+------------+-------------+---------------------------+---------------------------------------------------------------+-----------------+
| WB955      | <=200M      | Boot_SocClk_Info_Idx = 1; | BootLoader will set the Soc clock according to SocClk_Info[1] | PLL: 330MHz     |
|            |             |                           |                                                               |                 |
|            |             |                           |                                                               | KM4: PLL/1      |
|            |             |                           |                                                               |                 |
|            |             |                           |                                                               | KM0: PLL/4      |
|            |             |                           |                                                               |                 |
|            |             |                           |                                                               | PSRASM: PLL/1/2 |
+------------+-------------+---------------------------+---------------------------------------------------------------+-----------------+

4. Refer to one of the following methods to change the SoC clock if needed.

<<<<<<< HEAD
   - Modify SocClk_Info[0] in\ `` {SDK}\component\soc\amebadplus\usrcfg\ameba_bootcfg.c``\ , refer to 1.2.2 step (2) for details.
=======
   - Modify SocClk_Info[0] in\ ``{SDK}\component\soc\amebadplus\usrcfg\ameba_bootcfg.c``\ , refer to 1.2.2 step (2) for details.
>>>>>>> 0e8663f (update files)

   - Modify Boot_SocClk_Info_Idx to [0, sizeof(SocClk_Info)), and then define your own clock info in SocClk_Info [Boot_SocClk_ Info_Idx].



.. note::
   Consider the limitations of the hardware and do not set the clock info illogically.


5. Rebuild the project and download the new image again.

Example
^^^^^^^^^^^^^^
1. Refer to 1.2.1 step (1) to find out the speed limit of PSRAM device if not sure (suppose the maximum speed is 200MHz).

2. Change KM4_CKD of SocClk_Info[0] to CLKDIV(3) if KM4 is wanted to run at 520MHz/3.

.. image:: ../_static/user_config_rst/e927169908a31ff5b16b5f6533e407a864aceabf.png
   :width: 1060
   :align: center
 

3. Rebuild the project and download the new image.

Now, the clock of KM4 is 173.3MHz, KM0 is 86.6MHz, PSRAM controller is 260MHz (twice the PSRAM), and core power is 0.9V. The clocks of left modules in |CHIP_NAME| will be set to a reasonable value by software automatically based on their maximum speeds.

Flash Clock Switch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Flash runs half as fast as the SPI Flash controller. By default, the speed of the SPI Flash controller is divided by the PLL, and the speed of the SPI Flash controller shall be less than SPIC_CLK_LIMIT (208MHz). If the Flash needs to run slower, change the value of Flash_Speed (SPIC0) or Data_Flash_Speed (SPIC1) in \ ``{SDK}\component\soc\amebadplus\usrcfg\ameba_flashcfg.c.``\ 

.. image:: ../_static/user_config_rst/3d0d60a1a7055593ecb53c72afea651c98c71040.png
   :width: 647
   :align: center


