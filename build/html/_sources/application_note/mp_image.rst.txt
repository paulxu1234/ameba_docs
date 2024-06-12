.. _mp_image:

MP Image
----------------
The MP image is used for Wi\-Fi & BT performance verification and Wi\-Fi & BT parameters calibration in massive production. This chapter mainly illustrates how to build and use the MP image.

.. only:: internal
    
    
    Refer to \ *WS_RTL8721Dx_MP_FLOW.pdf*\  for more details about MP flow.
    



Refer to the MP document for more details about MP flow.

How to Build MP Image
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The steps of building MP image are depcited below:

1. Switch to work directory \ ``{SDK}\amebadplus_gcc_project``\ .

2. Use command \ ``$ make menuconfig``\  to modify the configurations.

   a. Enable MP



   b. Enable Wi\-Fi, and configure KM4 as AP core and KM0 as NP core.



   c. Enable BT.



   d. Save and exit the menuconfig.

3. Use command \ ``$ make all``\  to rebuild the projects of KM0 and KM4.


The MP image (km0_km4_app_mp.bin) will be generated in \ ``{SDK}\amebadplus_gcc_project``\  and all the images releated to the MP image can be found in the paths below:

- {SDK}\amebadplus_gcc_project\project_km4\asdk\image_mp

- {SDK}\amebadplus_gcc_project\project_km0\asdk\image_mp

How to Combine Images
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Before downloading images into the chip, the MP image needs to be combined with normal image. In massive production, the MP image is used to calibrate the parameters first. After calibration, you can use the command illustrated in Section 1.4 to switch to the normal image and boot from it. Since the chip boots from OTA1 by default when both OTA1 and OTA2 images are valid and with the same version number, the MP image should be located in the OTA1 field.


The steps of combining images are depcited below:

1. Check if both the normal image and MP image are valid.

2. Use the Image Tool to combine all the images, including km4_boot_all.bin, km0_km4_app_mp.bin and km0_km4_app.bin, which are located in \ ``{SDK}\amebadplus_gcc_project``\ .

   a. Set the image offsets in Image Tool according to the Flash layout, which can be found in \ ``{SDK}\component\soc\``\  \ ``amebadplus\usrcfg\ameba_flashcfg.c``\ .

      - Set km0_km4_app_mp.bin in \ ``IMG_APP_OTA1``\  section

      - Set km0_km4_app.bin in \ ``IMG_APP_OTA2``\  section

   b. Click \ ``Generate``\  button in Image Tool, and save the combined image named Image_All.bin.

.. image:: ../_static/mp_image_rst/e2ffa39d854d8b2ac51d8eaa0ae4e198c1ce9f17.png
   :width: 961
   :align: center


 

3. Download the Image_All.bin into the device after the combination is finished.



.. note::
   The normal image (km0_km4_app.bin) is built with MP disabled. Check the normal image in \ ``{SDK}\amebadplus_gcc_project``\ . All the images related to normal image can be found in paths below:

      - {SDK}\amebadplus_gcc_project\project_km4\asdk\image

      - {SDK}\amebadplus_gcc_project\project_km0\asdk\image


Boot Flow
~~~~~~~~~~~~~~~~~~
1. Reset the device after downloading image is finished.

2. Check if the device boots from MP image successfully.

.. image:: ../_static/mp_image_rst/a3bdfe153f651887b4a5b63b4174805c2aa718d6.png
   :width: 1025
   :align: center


3. Start the massive production flow if the device boots from MP image successfully.

How to Switch Image
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
When MP is finished, you need to switch the image from MP image to the normal image to verify the application.

(1)    Use command “ATSC” from serial terminal to clear the signature of MP image in order to assign the MP image invalid.

.. image:: ../_static/mp_image_rst/89cf3beccbe736165c7fc5dbb761feeb08b4c48b.png
   :width: 830
   :align: center


(2)    Reset the device, then the device will boot from the normal image located in OTA2 field.

.. image:: ../_static/mp_image_rst/0fe015c7a88fe15784611d5966b9e639d7cef61d.png
   :width: 807
   :align: center


.. only:: nda
    
    Encryption
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    .. only:: RTL8721D
        
        
        There are some encryption method supported for the device, including Secure Boot, RSIP and RDP.
        
    
    
    .. only:: RTL8711D
        
        
        There are some encryption method supported for the device, including Secure Boot and RSIP.
        
    
    

    The MP image shouldn't be encrypted, and would be programmed into Flash in plaintext. For security reason, when you need to enable any of encryptions for normal image, it should be processed in the MP application code by calling OTP programming APIs.
    
    1. Program encryption enable bit into OTP.
    
    2. Program keys into OTP.
    

    .. only:: RTL8721D
        
        
        Once encryption is enabled and you need to switch back from normal image to MP image, consider three cases described in Section 1.5.1, 1.5.2 and 1.5.3.
        
    
    
    .. only:: RTL8711D
        
        
        Once encryption is enabled and you need to switch back from normal image to MP image, consider two cases described in Section 1.5.1 and 1.5.2.
        
    
    
    Secure Boot Enable
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    When secure boot is enabled and you need to switch from normal image to MP image, MP image must be re\-built with secure boot enabled. The steps are shown below:
    
    1. Modidy manifest.json located in \ ``{SDK}\amebadplus_gcc_project``\ .
    
       - Set SECURE_BOOT_EN to 1.
    
    .. image:: ../_static/mp_image_rst/eb1f1b2275b3cfc100d6eb27ff296555f662505e.png
       :width: 645
       :align: center
    
    
    2. Re\-build MP image as described in section 1.1.
    
    3. Combine encrypted MP image with encrypted normal image as described in Section 1.2.
    
    4. Download image into Flash and reset the board, the device will boot from MP image.
    
    RSIP Enable
    ^^^^^^^^^^^^^^^^^^^^^^
    When RSIP is enabled and you need to switch from normal image to MP image, MP image must be re\-built with RSIP enabled. The steps are shown below:
    
    1. Modify manifest.json located in \ ``{SDK}\amebadplus_gcc_project.``\ 
    
       - Set RSIP_EN\ `` ``\ to 1.
    
    .. image:: ../_static/mp_image_rst/425140675ffb016f7b60619a37d2a9f731633ce0.png
       :width: 634
       :align: center
    
    
    2. Re\-build MP image as described in section 1.1.
    
    3. Combine encrypted MP image with encrypted normal image as described in section 1.2.
    
    4. Download image into flash and reset the board, the device will boot from MP image.
    
    .. only:: RTL8721D
        
        RDP Enable
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        When RDP is enabled and you need to switch from normal image to MP image, MP image must be re\-built with both RDP and TrustZone enabled. The steps are shown below:
        
        1. Modidy manifest.json located in \ ``{SDK}\amebadplus_gcc_project``\ .
        
           - Set RDP_EN to 1.
        
        .. image:: ../_static/mp_image_rst/8f651c9bdec6fcde70cb4a665171ab17fb030b44.png
           :width: 620
           :align: center
        
        
        2. Enable TrustZone by command \ ``$make menuconfig``\ .
        
        
        
        3. Re\-build MP image as described in section 1.1.
        
        4. Combine encrypted MP image with encrypted normal image as described in section 1.2.
        
        5. Download image into flash and reset the board, the device will boot from MP image.
        
