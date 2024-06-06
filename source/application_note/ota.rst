.. _ota_firmware_update:

OTA Firmware Update
--------------------------------------
Introduction
~~~~~~~~~~~~~~~~~~~~~~~~
Over\-the\-air (OTA) programming provides a methodology of updating device firmware remotely via TCP/IP network. For OTA via TCP/IP network, the |CHIP_NAME| provides solutions to implement OTA firmware upgrade from local server or cloud.

Image Slot
^^^^^^^^^^^^^^^^^^^^
There are two slots for all the images in the Flash layout as shown in Figure 1\-1, which named OTA1 and OTA2 respectively. Each image can be chosen to boot from OTA1 or OTA2.



Figure \- OTA1 and OTA2 position

Version Number
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The device boot from OTA1 or OTA2 mainly depends on the version number in certificate and manifest. As shown in Figure 1\-2, there is a 2\-byte major version and 2\-byte minor version in manifest and certificate.


The combination of major version and minor version is the 4\-byte version number. OTA select flow checks the whole version number.

.. code::

   Version number = (Major version << 16) | Minor version


Figure \- Major and minor version



.. note::
      - For bootloader, version number can be 0 to 32767; for application, version number can be 0 to 65535.

.. only:: RTL8721D
    
    
    .. note::
          - For detailed layouts of manifest and certificate, refer to \ ``错误!未找到引用源。``\ .
    




As described in 1.1.1, there are two slots (OTA1 and OTA2) for all the images in the Flash layout. When reboot after OTA upgrade finished, the device would check the image to determine to boot from OTA1 or OTA2.


The general principle of the OTA scheme is checking the image pattern first and then comparing the version number of OTA1 and OTA2.


The following items must be checked for each image:

1. Check the image pattern for both OTA1 and OTA2.

   - If only one image pattern is valid, boot from the valid image.

   - If both the image pattern are invalid, boot fail.

2. Compare the version number of OTA1 and OTA2.

   - If both OTA1 and OTA2 are valid and with different version numbers, the device will boot from the image with a bigger version.

   - If both OTA1 and OTA2 are valid but with the same version number, the device will boot from OTA1.



Figure \- OTA select diagram

Anti\-rollback
^^^^^^^^^^^^^^^^^^^^^^^^^^
Anti\-rollback is the function to prevent version rollback attack. When the anti\-rollback is enabled, the version number in certificate or manifest must not be smaller than the anti\-rollback version stored in OTP. Otherwise, this image will be regarded as invalid and the chip will not boot from invalid image. Normally, if OTA update is security\-related, user can program a bigger anti\-rollback version number in OTP and update image with a bigger major version at the same time to prevent rollback attack.


The anti\-rollback flow is shown in Figure 1\-4. Once the anti\-rollback is enabled, the device will compare the major version numbers got from OTA1 and OTA2 images respectively with the anti\-rollback version number in OTP. If the major version number in the image is smaller than the anti\-rollback version number, this image will be regarded as invalid.



Figure \- Anti\-rollback flow

Bootloader
~~~~~~~~~~~~~~~~~~~~
OTA Image
^^^^^^^^^^^^^^^^^^
The KM4 bootloader image named km4_boot_all.bin can be updated through OTA, which can be chosen to boot from OTA1 or OTA2. The layout of KM4 bootloader image is illustrated in Figure 1\-5.



Figure \- Layout of KM4 bootloader image

OTA Select Flow
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The KM4 ROM will select OTA image according to the image version number in bootloader manifest.



Figure \- KM4 bootloader OTA select flow

Application
~~~~~~~~~~~~~~~~~~~~~~
OTA Image
^^^^^^^^^^^^^^^^^^
.. only:: RTL8721D
    
    
    The application image named km0_km4_app.bin, including KM0, KM4 non\-secure application image and KM4 secure image, can be updated through OTA, which can be chosen to boot from OTA1 or OTA2. The layout of the whole application image is illustrated in Figure 1\-7.
    


.. only:: RTL8711D
    
    
    The application image named km0_km4_app.bin can be updated through OTA, which can be chosen to boot from OTA1 or OTA2. The layout of the whole application image is illustrated in Figure 1\-7.
    




Figure \- Layout of application image

OTA Select Flow
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The application image OTA select flow is illustrated in Figure 1\-8.



Figure \- Application image OTA select flow

Building OTA Image
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Modifying Configurations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
3. Modify the version number in configuration file: \ ``manifest.json``\ .

+---------------+------+-------------------------------------------------------------------+------------------------------+
| File          | Tag  | Description                                                       | Path                         |
+===============+======+===================================================================+==============================+
| manifest.json | boot | Configure major and minor version for KM4 bootloader              | {SDK}\amebadplus_gcc_project |
+---------------+------+-------------------------------------------------------------------+------------------------------+
| manifest.json | app  | Configure major and minor version for certificate and application | {SDK}\amebadplus_gcc_project |
+---------------+------+-------------------------------------------------------------------+------------------------------+

   a. Modify the version number for bootloader.

.. image:: ../static/ota_rst/76c80f63927f9569d753a29cc021bd6a8a9e01f1.png
   :width: 384
   :align: center


   b. Modify the version number for certificate and application.

.. image:: ../static/ota_rst/11cb3d15604e4979f14b1cc38b0c6259772a0798.png
   :width: 372
   :align: center


4. Change the bootloader version of anti\-rollback and enable anti\-rollback if necessary.

   a. Change the bootloader version of anti\-rollback

By default, all images use the same anti\-rollback version in OTP as threshold to prevent anti\-rollback attack.

+--------------------+----------------------+---------+------------------------------------------+
| Name               | OTP address          | Length  | Description                              |
+====================+======================+=========+==========================================+
| BOOTLOADER_VERSION | Physical 0x36E~0x36F | 16 bits | The bootloader version of anti\-rollback |
+--------------------+----------------------+---------+------------------------------------------+

The bootloader version of anti\-rollback is 0 by default. Users can change the number of '0' bit to enlarge the bootloader version. For example, users can program the bootloader version of anti\-rollback to 1 by the following command:

.. code::

   EFUSE wraw 36E 2 FFFE
   b. Enable anti\-rollback

Users can program OTP by the following command to enable anti\-rollback.

.. code::

   EFUSE wraw 368 1 BF


.. note::
         - Once anti\-rollback is enabled, it cannot be disabled.

         - If bootloader and application do not use the same anti\-rollback version, modify BOOT_OTA_GetCertRollbackVer() in \ ``{SDK}\component\soc\amebadplus\bootloader\boot_ota_km4.c``\  and define another anti\-rollback version in OTP for the application.


5. Write the bootloader OTA2 address into OTP if users need to upgrade the bootloader, which sets the bootloader OTA2 address according to Flash_Layout in \ ``{SDK}\component\soc\amebadplus\usrcfg\ameba_flashcfg.c``\ , refer to 1.8.

.. code::

   EFUSE wraw 36C 2 6082


.. note::
         - The address of bootloader OTA2 is the value of OTP 0x36C with 12\-bit left shifted, or is the value of OTP 0x36C \* 4K.

         - If the address of bootloader OTA2 is 0xFFFFFFFF by default, the bootloader won't be upgraded when in OTA upgrade and the device always boots from bootloader OTA1.

         - The above commands are used in the serial terminal tool.


6. Rebuild the project using "make all" command to generate the signed images.

7. Download the images into Flash, and reset the board.

.. image:: ../static/ota_rst/873ba40b44e77b94f2642cd3e9f5f7ea1cab1652.png
   :width: 641
   :align: center


Generating OTA Image Automatically
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The OTA image will be generated automatically when building the project.

1. km0_km4_app.bin is included in OTA_All.bin by default.

2. If the bootloader is needed to be upgraded, 

   a. Type command "make menuconfig" under \ ``{SDK}\amebadplus_gcc_project``\  and choose:

CONFIG OTA OPTION \-> Upgrade Bootloader, save and exit.

   b. Modify the bootloader related configurations as described in 1.4.1.

3. Rebuild the project by command "make all" under \ ``{SDK}\amebadplus_gcc_project``\ . The OTA image file called OTA_All.bin will be generated in \ ``{SDK}\amebadplus_gcc_project``\ .

Updating from Local Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This section introduces the design principles and usage of OTA from local server. It has well\-transportability to porting to OTA applications from cloud.


The OTA from local server shows how the device updates the image from a local download server. The local download server sends the image to the device based on the network socket, as Figure 1\-9 shows.


Make sure both the device and the PC are connecting to the same local network.



Figure \- OTA update diagram via network

Firmware Format
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The firmware format is illustrated in Figure 1\-10.



Figure \- Firmware format

Table \- Firmware header

+---------------+----------------+---------+-------------------------------------------------------------------+
| Items         | Address offset | Size    | Description                                                       |
+===============+================+=========+===================================================================+
| Version       | 0x00           | 4 bytes | The version of OTA Header, default 0xFFFFFFFF                     |
+---------------+----------------+---------+-------------------------------------------------------------------+
| Header Number | 0x04           | 4 bytes | The number of OTA Header. For |CHIP_NAME|, this value can be 1, 2 |
+---------------+----------------+---------+-------------------------------------------------------------------+
| Signature     | 0x08           | 4 bytes | OTA Signature is string. For |CHIP_NAME|, this value is “OTA”     |
+---------------+----------------+---------+-------------------------------------------------------------------+
| Header Length | 0x0C           | 4 bytes | The length of OTA header. For |CHIP_NAME|, this value is 0x18     |
+---------------+----------------+---------+-------------------------------------------------------------------+
| Checksum      | 0x10           | 4 bytes | The checksum of OTA image                                         |
+---------------+----------------+---------+-------------------------------------------------------------------+
| Image Length  | 0x14           | 4 bytes | The size of OTA image                                             |
+---------------+----------------+---------+-------------------------------------------------------------------+
| Offset        | 0x18           | 4 bytes | The start position of OTA image in current image                  |
+---------------+----------------+---------+-------------------------------------------------------------------+
| Image ID      | 0x1C           | 4 bytes | The image ID of current image                                     |
|               |                |         |                                                                   |
|               |                |         | - OTA_IMGID_BOOT: 0x0                                             |
|               |                |         |                                                                   |
|               |                |         | - OTA_IMGID_APP: 0x1                                              |
+---------------+----------------+---------+-------------------------------------------------------------------+

OTA Flow
^^^^^^^^^^^^^^^^
The OTA demo locates in \ ``{SDK}\component\soc\amebadplus\misc\ameba_ota.c``\ . The image upgrade is implemented in the following steps:

3. Connect to the server. The IP address, port and OTA type are needed.

4. Acquire the older firmware address to be upgraded according to the MMU setting. If the address is re\-mapping to OTA1 space by MMU, the OTA2 address would be selected to upgrade. Otherwise, the OTA1 address would be selected.

5. Receive the firmware file header to get the target OTA image information, such as image number, image length and image ID.

6. Download the new firmware from server.

7. Erase the Flash space for new firmware and write it into Flash except Manifest structure.

8. Verify the checksum. If the checksum is error, OTA fails.

9. If the checksum is ok, write Manifest structure to the upgraded firmware region to indicate boot from a new firmware next time.

10. OTA is finished and reset the device. Then it would boot from the new firmware.



Figure \- OTA operation flow

OTA Demo
~~~~~~~~~~~~~~~~
Follow these steps to run the OTA demo to update from local server:

1. Edit \ ``{SDK}\component\example\ota\example_ota.c``\ .

   a. Edit the host according to the server IP address.

.. code::

   #define PORT   8082
   static const char *host = "192.168.31.193";   //"m-apps.oss-cn-shenzhen.aliyuncs.com"
   static const char *resource = "OTA_All.bin"; //"051103061600.bin"
   b. Edit the OTA type to OTA_LOCAL.

.. code::

   ret = ota_update_init(ctx, (char *)host, PORT, (char *)resource, OTA_LOCAL);
2. Rebuild the project with the command "make all EXAMPLE\=ota" and download the images to the device.

3. Modify the major and minor version number in Manifest to a bigger version as described in Section 1.1.2.



.. note::
   The bootloader will select OTA image with a bigger version number by default. If users don't want to modify the version number, modify OTA_CLEAR_PATTERN to 1 defined in ameba_ota.h before step (2). It should only be used in the development stage.


4. Rebuild the project and copy \ ``OTA_All.bin``\  into the folder \ ``{SDK}\tools\DownloadServer``\ .

5. Edit \ ``{SDK}\tools\DownloadServer\start.bat``\ .

   c. port \= 8082

   d. file name \= OTA_All.bin

.. code::

   @echo off
   DownloadServer 8082 OTA_All.bin
   set /p DUMMY=Press Enter to Continue ...
6. Click the \ ``start.bat``\ , and start the download server program.

7. Reboot the DUT and connect the device to the AP which the OTA Server in.

8. Reboot DUT to execute the new firmware after finishing image download.

OTA Firmware Swap
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The Figure 1\-12 shows the firmware swap procedure after OTA upgrade.



Figure \- OTA firmware swap procedure

User Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Modify the memory layout in \ ``{SDK}\component\soc\amebadplus``\ \\ ``usrcfg\ameba_flashcfg.c``\  if needed.

.. image:: ../static/ota_rst/b122cd1ded7d01a4f992c800b441ba8c373cc04e.png
   :width: 1614
   :align: center


Figure \- Flash layout

