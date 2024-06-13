.. only:: nda
    
    .. _secure_boot:
    
    Secure Boot
    ----------------------------------
    Introduction
    ~~~~~~~~~~~~~~~~~~~~~~~~
    Secure boot aims at firmware protection, which prevents attackers from modifying or replacing firmware maliciously. When the chip is power on, the secure boot ROM executes to check the validation of the image signature. If the signature is valid, authentication will be successful, which means that the firmware is safe and the subsequent operations can be continued. Otherwise, the SoC clears the stack and goes into an endless loop.
    

    Users do not need to implement the secure boot code themselves, which is already contained in the SDK.
    

    To generate the Public Key and signature of bootloader, we provide an image operation tool named "elf2bin". With the tool, users can generate and append signature-related information to images. Refer to the following sections for operation details.
    

    The principle of secure boot is illustrated in Figure 1-1. With elf2bin tool, the public key, signature, image hash and other fields are generated in the manifest.
    
    
    
    Figure - Secure boot diagram
    
    The manifest (4KB) is located at the beginning of the image. Each of the following images has a manifest:
    
    - KM4 bootloader
    
    - Key certificate
    
    - KM0 & KM4 application image
    

    The images validation process is as follows:
    
    1. KM4 Boot ROM validates KM4 bootloader.
    
    2. KM4 bootloader validates the key certificate.
    
    .. only:: RTL8721D
        
        
        3. KM4 bootloader validates the KM0 & KM4 application image, which includes NSPE image (KM0 & KM4 non-secure image) and SPE image (KM4 secure image) separately.
        
    
    
    .. only:: RTL8711D
        
        
        4. KM4 bootloader validates the KM0 & KM4 application image (NSPE image).
        
    
    

    The KM4 bootloader and key certificate are signed against RoT (Root of Trust) key. The RoTPK hash is stored in OTP.
    

    The key pair of signing KM0 & KM4 application image should be different from the RoT key pair. Users can choose one key or different keys to sign the image.
    

    The key certificate contains the public key hash of other images. So KM4 bootloader verifies the certificate firstly to get the public key hash of the images of the following stage. Then it verifies the following stage images against the public key hash from the key certificate.
    

    Refer to Chapter Security of User Manual for more signing and validation details.
    
    Secure Boot OTP
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    Public Key Hash & Secure Boot Enable
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    +--------+-----------------------+----------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Mode   | Name                  | OTP address          | Length   | Description                                                                                                                                                 |
    +========+=======================+======================+==========+=============================================================================================================================================================+
    | Normal | PK1                   | Physical 0x320~0x33F | 32 bytes | To be compatible with different algorithms and curves, the public key hash (SHA256) is stored in OTP (32 bytes).                                            |
    +--------+-----------------------+----------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Normal | PK2                   | Physical 0x340~0x35F | 32 bytes | To be compatible with different algorithms and curves, the public key hash (SHA256) is stored in OTP (32 bytes).                                            |
    +--------+-----------------------+----------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Normal | PK1_W_Forbidden_EN    | Physical 0x365[1]    | 1 bit    | When programmed, PK1 cannot be modified anymore.                                                                                                            |
    +--------+-----------------------+----------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Normal | PK2_W_Forbidden_EN    | Physical 0x365[2]    | 1 bit    | When programmed, PK2 cannot be modified anymore.                                                                                                            |
    +--------+-----------------------+----------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Normal | SECURE_BOOT_EN_PHY    | Physical 0x368[3]    | 1 bit    | When either of the two bits is programmed, secure boot is enabled.                                                                                          |
    |        |                       |                      |          |                                                                                                                                                             |
    |        |                       |                      |          | - When the device is in the development or debugging stage, it is recommended to use SECURE_BOOT_EN_LOG, because it can be restored to disable secure boot. |
    |        |                       |                      |          |                                                                                                                                                             |
    |        |                       |                      |          | - When the device is in the MP stage, it is recommended to program SECURE_BOOT_EN_PHY to enable secure boot permanently.                                    |
    +--------+-----------------------+----------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Normal | SECURE_BOOT_EN_LOG    | Logical 0x3[2]       | 1 bit    | When either of the two bits is programmed, secure boot is enabled.                                                                                          |
    |        |                       |                      |          |                                                                                                                                                             |
    |        |                       |                      |          | - When the device is in the development or debugging stage, it is recommended to use SECURE_BOOT_EN_LOG, because it can be restored to disable secure boot. |
    |        |                       |                      |          |                                                                                                                                                             |
    |        |                       |                      |          | - When the device is in the MP stage, it is recommended to program SECURE_BOOT_EN_PHY to enable secure boot permanently.                                    |
    +--------+-----------------------+----------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | RMA    | RMA_PK                | Physical 0x720~0x73F | 32 bytes | The hash of secure boot public key in RMA mode.                                                                                                             |
    |        |                       |                      |          |                                                                                                                                                             |
    |        |                       |                      |          | Users can maintain a RMA key that is different from PK1/PK2 for security reasons.                                                                           |
    +--------+-----------------------+----------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | RMA    | RMA_PK_W_Forbidden_EN | Physical 0x702[2]    | 1 bit    | When programmed, RMA_PK cannot be modified anymore.                                                                                                         |
    +--------+-----------------------+----------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------+


    The hash of bootloader public key should be stored in PK1 (physical 0x320~0x33F). The key certificate can use the same public key as the bootloader or use a different public key. It can be configured in \ ``{SDK}\component\soc\amebadplus\usrcfg\ameba_bootcfg.c``\ .
    
    .. code::
    
       u32 Cert_PKHash_OTP_ADDR = SEC_PKKEY_PK1_0;  //or SEC_PKKEY_PK2_0
    By default, SEC_PKKEY_PK1_0 is selected and the key certificate uses the same public key as bootloader.
    

    For some module manufacturers, the bootloader and application are developed by different manufacturers. To prevent the public key of bootloader from leaking, users can select SEC_PKKEY_PK2_0 to use PK2 to validate the certificate.
    

    Refer to the Security chapter of User Manual for the usage of RMA_PK.
    
    Authentication and Hash Algorithms Selection
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    +----------------------+---------------------+--------+-----------------------------------------------+
    | Name                 | OTP address         | Length | Description                                   |
    +======================+=====================+========+===============================================+
    | SECURE_BOOT_AUTH_ALG | Physical 0x36B[3:0] | 4 bits | Determines the authentication algorithm used. |
    |                      |                     |        |                                               |
    |                      |                     |        | - 0x0: ED25519                                |
    +----------------------+---------------------+--------+-----------------------------------------------+
    | SECURE_BOOT_HASH_ALG | Physical 0x36B[7:4] | 4 bits | Determines the hash algorithm used.           |
    |                      |                     |        |                                               |
    |                      |                     |        | - 0x0: SHA 256                                |
    |                      |                     |        |                                               |
    |                      |                     |        | - 0x1: SHA 384                                |
    |                      |                     |        |                                               |
    |                      |                     |        | - 0x2: SHA 512                                |
    |                      |                     |        |                                               |
    |                      |                     |        | - 0x3: HMAC-SHA256                            |
    |                      |                     |        |                                               |
    |                      |                     |        | - 0x4: HMAC-SHA384                            |
    |                      |                     |        |                                               |
    |                      |                     |        | - 0x5: HMAC-SHA512                            |
    +----------------------+---------------------+--------+-----------------------------------------------+

    
    
    .. note::
          - If these two fields are not programmed, it will refer to the auth_alg and hash_alg settings in the manifest.
    
          - If these two fields are programmed, the values should equal the auth_alg and hash_alg settings in the manifest. If not, secure boot will fail.
    
    
    HMAC Key
    ^^^^^^^^^^^^^^^^
    +------------------------------+----------------------+----------+----------------------------------------------------------------------------------+
    | Name                         | OTP address          | Length   | Description                                                                      |
    +==============================+======================+==========+==================================================================================+
    | S_IPSEC_Key2                 | Physical 0x220~0x23F | 32 bytes | When HMAC is adopted instead of SHA, the HMAC key should be programmed here.     |
    +------------------------------+----------------------+----------+----------------------------------------------------------------------------------+
    | S_IPSEC_Key2_R_Protection_EN | Physical 0x365[5]    | 1 bit    | When this bit is programmed, S_IPSEC_Key2 cannot be read out by the CPU anymore. |
    +------------------------------+----------------------+----------+----------------------------------------------------------------------------------+
    | S_IPSEC_Key2_W_Protection_EN | Physical 0x365[6]    | 1 bit    | When this bit is programmed, S_IPSEC_Key2 cannot be modified anymore.            |
    +------------------------------+----------------------+----------+----------------------------------------------------------------------------------+

    Image Signing
    ~~~~~~~~~~~~~~~~~~~~~~~~~~
    The signing steps for the image are as follows. For the key certificate, it does not have image payload, so go to step (3) directly.
    
    1. Calculate the hash of the image.
    
    For EdDSA, the hash algorithm is determined by the manifest, which can be SHA256/384/512 or HMAC-SHA256/384/512.
    
    2. Fill the hash value into the manifest.
    
    3. Sign the manifest to get the signature.
    
    For EdDSA, the hash algorithm used when signing is fixed to SHA512.
    
    
    
    Figure - Image signing flow
    
    How to Use Secure Boot
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    The following steps illustrate how to use secure boot.
    
    1. Generate key pairs using the following tool and command:
    
    .. code::
    
       $ cd {SDK}/amebadplus_gcc_project/project_km4/asdk/gnu_utility/image_tool
       $ ./elf2bin keypair <auth_alg> <output_filename>
    The \ ``auth_alg``\  can be ED25519.
    
    For example, you can type the following command to generate ED25519 key-pair file, named \ ``keypair.json``\ :
    
    .. code::
    
       $ ./elf2bin keypair ed25519 keypair.json
    The default key pairs used by default SDK locate in \ ``{SDK}\amebadplus_gcc_project\project_km4\asdk\gnu_utility\image_tool``\ .
    
    2. Copy the generated key pairs into boot, cert or app field in \ ``{SDK}\amebadplus_gcc_project\manifest.json``\ .
    
    .. only:: RTL8721D
        
        
        +---------------+-------+-------------------------------------------------------------------------------------------+
        | File          | Field | Description                                                                               |
        +===============+=======+===========================================================================================+
        | manifest.json | boot  | Used to sign the bootloader                                                               |
        +---------------+-------+-------------------------------------------------------------------------------------------+
        | manifest.json | cert  | Used to sign the Key Certificate                                                          |
        +---------------+-------+-------------------------------------------------------------------------------------------+
        | manifest.json | app   | Used to sign KM0 & KM4 application image, including NSPE image and SPE image (if enabled) |
        +---------------+-------+-------------------------------------------------------------------------------------------+

    
    
    .. only:: RTL8711D
        
        
        +---------------+-------+-------------------------------------------------------+
        | File          | Field | Description                                           |
        +===============+=======+=======================================================+
        | manifest.json | boot  | Used to sign the bootloader                           |
        +---------------+-------+-------------------------------------------------------+
        | manifest.json | cert  | Used to sign the Key Certificate                      |
        +---------------+-------+-------------------------------------------------------+
        | manifest.json | app   | Used to sign KM0 & KM4 application image (NSPE image) |
        +---------------+-------+-------------------------------------------------------+

    
    
    By default, both the bootloader and key certificate are signed with RoT key, so the contents of boot and cert\ `` ``\ fields in manifest\ ``.json ``\ are the same. Please generate your own key pairs and overwrite them.
    
    3. Program secure boot related OTP bits after the system boots up.
    
       a. ROTPK hash:
    
    The hash value is from "public key hash" of boot field in\ `` ``\ manifest\ ``.json``\  file, as follows:
    
    .. image:: ../_static/secure_boot_rst/45a9595e0ef78386e6e7c34f560cc28437463cb4.png
       :width: 659
       :align: center
    
    
    Using the following command to program OTP PK1:
    
    .. code::
    
       efuse wraw 0x320 20 72B2E1CB0E8F715262AF38DFA0E522C95660D0EBFD920F4B1A229845E599C697
    If the key pair of signing Key Certificate is different from that of signing bootloader, you should also program the hash value of cert field in manifest\ ``.json ``\ into OTP PK2.
    
    .. code::
    
       efuse wraw 0x340 20 XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
       b. HMAC key
    
    If using HMAC algorithm instead of SHA to generate the digest of image data, you should program the HMAC key into OTP.
    
    The HMAC key is from \ ``manifest.json ``\ file, as follows:
    
    .. image:: ../_static/secure_boot_rst/03721978a8cb98a901b79a84e22dbe5cdab3ead5.png
       :width: 599
       :align: center
    
    
    Using the following command to program HMAC key:
    
    .. code::
    
       efuse wraw 0x220 20 9874918301909234686574856692873911223344556677889900aabbccddeeff
    Please generate your own HMAC key for MP devices.
    
    
    
    .. note::
       For a device, there is only one HMAC key, which means all boot stages should use the same HMAC key.
    
    
       c. Secure Boot Enable bit
    
          - When in device-development stage, it is recommended to program SECURE_BOOT_EN_LOG bit, which can be disabled afterward. Use efuse rmap first to check value in 0x3, then enable SECURE_BOOT_EN_LOG bit (0x3 bit2).
    
    .. code::
    
       efuse rmap
    .. image:: ../_static/secure_boot_rst/23ba2d1698addd19270e7f03d496f22a9120b470.png
       :width: 603
       :align: center
    
    
    .. code::
    
       efuse wmap 0x3 1 e4
          - When in device-MP stage, you should program the SECURE_BOOT_EN_PHY bit to enable secure boot permanently.
    
    .. code::
    
       efuse rraw
    .. image:: ../_static/secure_boot_rst/394c9a596e20ef4e8d9d6b9cf37f5145dd55b12d.png
       :width: 493
       :align: center
    
    
    .. code::
    
       efuse wraw 0x368 1 F7
       d. Other security-related bits
    
    For MP devices, you should also program some other security-related bits for security reasons, such as key read/write protection bits, SECURE_BOOT_AUTH/HASH_ALG bits, etc.
    
    4. Modify the manifest configuration file.
    
    The manifest configuration file locates in \ ``{SDK}\amebadplus_gcc_project``\ .
    
    .. only:: RTL8721D
        
        
        +---------------+-------+-------------------------------------------------------------------------+
        | File          | Field | Description                                                             |
        +===============+=======+=========================================================================+
        | manifest.json | boot  | Used to generate manifest for bootloader                                |
        +---------------+-------+-------------------------------------------------------------------------+
        | manifest.json | app   | Used to generate manifest for Key Certificate, NSPE image and SPE image |
        +---------------+-------+-------------------------------------------------------------------------+

    
    
    .. only:: RTL8711D
        
        
        +---------------+-------+--------------------------------------------------------------+
        | File          | Field | Description                                                  |
        +===============+=======+==============================================================+
        | manifest.json | boot  | Used to generate manifest for bootloader                     |
        +---------------+-------+--------------------------------------------------------------+
        | manifest.json | app   | Used to generate manifest for Key Certificate and NSPE image |
        +---------------+-------+--------------------------------------------------------------+

    
    
    For each field of configuration file:
    
       e. Set SECURE_BOOT_EN to 1.
    
       f. Fill in HASH_ALG, which can be sha256/sha384/sha512/hmac256/hamc384/hmac512.
    
       g. Fill in HMAC_KEY if step b) selects hmac256, or hmac384, or hmac512.
    
    .. image:: ../_static/secure_boot_rst/4055c60d28b492d1581d07fc8e43b12243bf791c.png
       :width: 583
       :align: center
    
    
    5. Generate signed image-tool flashloader binary by using "\ ``make gen_imgtool_floader RTLNAME=xxx``\ " command under \ ``{SDK}\amebadplus_gcc_project ``\ folder if you are going to download images with ImageTool. Then copy the following binary to the ImageTool folder to overwrite the original one: \ ``{SDK}\amebadplus_gcc_project\floader_amebadplus.bin``\ .
    
    
    
    .. note::
       In order to prevent attackers from injecting malicious code, the flashloader binary should also be signed when secure boot is enabled. Only signed flashloader is allowed to execute.
    
    
    6. Rebuild the project by "make" command to generate the following signed images automatically, then download them into Flash.
    
    +-----------------------+------------------+------------------+
    | Project               | Encrypted image  | Download address |
    +=======================+==================+==================+
    | km4_bootloader        | km4_boot_all.bin | 0x0800_0000      |
    +-----------------------+------------------+------------------+
    | cert.bin              | Km0_km4_app.bin  | 0x0801_4000      |
    +-----------------------+------------------+------------------+
    | km0_application       | Km0_km4_app.bin  | 0x0801_4000      |
    +-----------------------+------------------+------------------+
    | km4_application       | Km0_km4_app.bin  | 0x0801_4000      |
    +-----------------------+------------------+------------------+
    | Km4_img3 (if enabled) | Km0_km4_app.bin  | 0x0801_4000      |
    +-----------------------+------------------+------------------+

    7. Reset the device. When secure boot is successful, you can see the following log:
    
       - "IMG1 SBOOT EN": secure boot is enabled
    
       - "IMG1(OTA1) VALID, ret: 0": bootloader authentication pass
    
       - "IMG2 VERIFY PASS": application image authentication pass
    
    .. code::
    .. only:: internal
        
        How to Use ELF2BIN Manually
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        cd {SDK}/amebadplus_gcc_project/project_km4/asdk/gnu_utility/image_tool
        
        1. 生成manifest.bin
        
        .. code::
        
           $ ./elf2bin manifest <manifest.json> < manifest.json> <img_file> <manifest.bin> [app|boot]
        其中：
        
        - manifest.json提供version/imgid等讯息，如果enable secure boot，则会使用manifest.json中的key info (algorithm, private key, public key, public key hash)
        
        - key info in manifest.json提供用于生成manifest.bin中signature字段所需的key相关信息
        
        - <img_file>是image源文件
        
        - <manifest.bin>存放生成的manifest.bin
        
        - [app|boot]，根据该信息生成app image的manifest还是bootloader的manifest
        

        For example, you can type in the following command to generate app image的manifest.bin:
        
        .. code::
        
           $ ./elf2bin.exe manifest $MANIFEST_JSON $MANIFEST_JSON $KM4_IMG_DIR/km0_km4_app.bin $KM4_IMG_DIR/manifest.bin app
        2. RSIP加密
        
        .. code::
        
           $ ./elf2bin rsip <src.bin> <dst.bin> <virtual_addr> <manifest.json> [app|boot]
        其中：
        
        - <src.bin>是需要加密的原始bin文件
        
        - <dst.bin>是加密后的bin文件
        
        - <virtual_addr>是image对应的flash的虚拟地址
        
        - manifest.json提供加密的AES key和RSIP_IV
        
        - [app|boot]，根据该信息使用manifest.json中对应的AES key和RSIP_IV
        

        .. code::

        
           $ ./elf2bin.exe rsip $KM4_IMG_DIR/km0_image2_all.bin $KM4_IMG_DIR/km0_image2_all_en.bin 0x0c000000 $MANIFEST_JSON app
        3. RDP加密
        
        .. code::
        
           $ ./elf2bin rdp enc <src.bin> <dst.bin> <manifest.json>
        其中：
        
        - <src.bin>是需要加密的原始bin文件
        
        - <dst.bin>是加密后的bin文件
        
        - manifest.json提供加密的AES key和RSIP_IV
        

        需要注意的是，RDP加密使用的IV是由 APP中的RSIP_IV + RDP_IV
        

        4 cert生成
        

        .. code::
        
           $ ./elf2bin.exe cert <manifest.json> < manifest.json> <out_file> <key_id1> <key1_name> <key_id2> <key2_name>...
        其中
        
        - manifest.json提供version/imgid等讯息，如果enable secure boot，则会使用manifest.json中的key info(algorithm, private key, public key, public key hash)，cert的信息app保持一致
        
        - key info in manifest.json提供用于生成cert.bin中signature字段所需的key相关信息
        
        - key_id和key_name则是将manifest.json中对应的public key hash存放进cert.bin中
        

        For example, you can type in the following command to generate app image的cert.bin:
        
        .. code::
    
       $ ./elf2bin.exe cert $MANIFEST_JSON $MANIFEST_JSON $KM4_IMG_DIR/cert.bin 0 app
    About RMA
    ~~~~~~~~~~~~~~~~~~
    During the RMA process, Realtek needs to run its own test code to locate the problem.
    

    Because the entire OTP security zone is not readable and writable in RMA mode, the confidential information would not be leaked out. It is safe to run other code on the SoC.
    

    To provide a safer way, we realize the implementation of RMA secure boot.
    
    - If the RMA PK hash is programmed, the secure boot is force to be enabled in RMA mode. Only the image signed with RMA key can be executed.
    
    - If the RMA PK hash is not programmed, the secure boot is disabled in RMA mode. Any code can run on the SoC.
    

    It is recommended that RMA PK is different from ROTPK.
    
