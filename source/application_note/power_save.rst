.. _power_saving:

Power Saving
------------------------
Power-Saving Mode
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Summary
^^^^^^^^^^^^^^
The |CHIP_NAME| has an advanced Power Management Controller (PMC), which can flexibly power up different power domains of the chip, to achieve the best balance between chip performance and power consumption. AON, SYSON, SOC are three main power domains in digital system. Functions in different power domains will be turned off differently in different power-saving modes.



Figure 1-1 FreeRTOS tickless in an idle task

<<<<<<< HEAD
To simplify the power management for typical scenarios, |CHIP_NAME| supports two low-power modes, which are sleep mode and deep-sleep mode. Tickless is a FreeRTOS low power feature, which just gates the CPU (no clock or power be turned off) when it has nothing to do. Sleep mode flow and deep-sleep mode flow are based on Tickless. \ *Table*\  \ *1-1*\  explains power-saving related terms.
=======
To simplify the power management for typical scenarios, |CHIP_NAME| supports two low-power modes, which are sleep mode and deep-sleep mode. Tickless is a FreeRTOS low power feature, which just gates the CPU (no clock or power be turned off) when it has nothing to do. Sleep mode flow and deep-sleep mode flow are based on Tickless. \ *Table 1-1*\  explains power-saving related terms.
>>>>>>> 0e8663f (update files)

Table 1-1 Power-saving mode

+------------+------------+--------------+----------------------------+--------------------------------------------------------------------------------------------------------------------+
| Mode       | AON domain | SYSON domain | SOC domain                 | Description                                                                                                        |
+============+============+==============+============================+====================================================================================================================+
| Tickless   | ON         | ON           | ON                         | - FreeRTOS low power feature                                                                                       |
|            |            |              |                            |                                                                                                                    |
|            |            |              |                            | - CPU periodically enters WFI, and exits WFI when interrupts happen.                                               |
|            |            |              |                            |                                                                                                                    |
|            |            |              |                            | - Radio status can be configured off/periodically on/always on, which depends on the application.                  |
+------------+------------+--------------+----------------------------+--------------------------------------------------------------------------------------------------------------------+
| Sleep      | ON         | ON           | Clock-gated or power-gated | - A power saving mode on chip level, including clock-gating mode and power-gating mode.                            |
|            |            |              |                            |                                                                                                                    |
|            |            |              |                            | - CPU can restore stack status when the system exits from sleep mode.                                              |
|            |            |              |                            |                                                                                                                    |
|            |            |              |                            | - The system RAM will be retained, and the data in system RAM will not be lost.                                    |
+------------+------------+--------------+----------------------------+--------------------------------------------------------------------------------------------------------------------+
| Deep-sleep | ON         | OFF          | OFF                        | - A more power-saving mode on chip-level.                                                                          |
|            |            |              |                            |                                                                                                                    |
|            |            |              |                            | - CPU cannot restore stack status. When the system exits from deep-sleep mode, the CPU follows the reboot process. |
|            |            |              |                            |                                                                                                                    |
|            |            |              |                            | - The system RAM will not be retained.                                                                             |
|            |            |              |                            |                                                                                                                    |
|            |            |              |                            | - The retention SRAM will not be power off.                                                                        |
+------------+------------+--------------+----------------------------+--------------------------------------------------------------------------------------------------------------------+


Peripherals are also divided into different power domains in order to achieve better power consumption.

- SOC power domain: including peripherals such as GDMA, SPI, I2C, SDIO, PWM, QSPI, PPE, LEDC, etc.

- SYSON power domain: mainly including peripherals which support wakeup, such as Basic Timer, GPIO, UART, Key-Scan, Cap-Touch, etc.

For peripherals in SOC domain, note that they need to be reinitialized after wakeup from PG since the power domain they are on has been power gated during sleep.


FreeRTOS supports a low-power feature called tickless. It is implemented in an idle task which has the lowest priority. That is, it is invoked when there is no other task under running. Note that unlike the original FreeRTOS, the |CHIP_NAME| does not wake up based on the "xEpectedIdleTime" in Figure 1-2.


Figure 1-2 shows idle task code flow. In idle task, it will check sleep conditions (wakelock, sysactive_time, details in 1.6.1, 1.6.2) to determine whether needs to enter sleep mode or not.

- If not, the CPU will execute an ARM instruction “WFI” (wait for interrupt) which makes the CPU suspend until the interrupt happens. Normally systick interrupt resumes it. This is the software tickless.

- If yes, it will execute the function freertos_pre_sleep_processing() to enter sleep mode or deep-sleep mode.



Figure 1-2 FreeRTOS tickless in an idle task



.. note::
      - Even FreeRTOS time control like software timer or vTaskDelay is set, it still enters the sleep mode if meeting the requirement as long as the idle task is executed.

      - configUSE_TICKLESS_IDLE must be enabled for power-saving application because sleep mode flow is based on tickless.

.. only:: nda
    
    Sleep Mode
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    There are two sleep mode types, sleep CG and sleep PG. CG means to turn off the clock of the SOC domain, while PG means to turn off both the power and clock of the SOC domain. So PG has lower power consumption.
    

    Since the CPU is in SOC domain, sleep PG will need to back up and restore the CPU status, thus PG will consume a bit more time during sleep and wakeup flow than CG.
    

    KM0 and KM4 support both CG or both PG since they are in the same power domain. KM4 is normally used as Application Processor (AP), and KM0 is normally used as Network Processor (NP) for Wi-Fi driver and Wi-Fi firmware, thus KM0 can enter sleep mode only if KM4 requests to enter sleep mode first.
    
    - For sleep PG, if a wakeup event occurs, PMC will turn on the SOC domain's power and clock. KM0 will first start from the reset handler, and check the flag to see if it wakes from PG. If so, KM0 will restore the CPU status, continue to execute from where it sleeps, and then check wakeup reasons to see if this wake source is for KM4, then decide whether to release KM4's clock to resume KM4. Figure 1-3 shows the sleep and wake flow of PG.
    
    - For sleep CG, if a wakeup event occurs, PMC will turn on the SOC domain's clock. Since KM0 is not power-gated in sleep CG, it will wake up and continue to execute from where it sleeps, and then check wakeup reasons to see if need to resume KM4. Figure 1-4 shows the sleep and wake flow of CG.
    
    
    
    Figure 1-3 Sleep and wake flow of PG
    
    
    
    Figure 1-4 Sleep and wake flow of CG
    
.. only:: nda
    
    Deep-Sleep Mode
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    Deep-sleep mode has a lower power consumption as only the AON domain is on while the SYSON and SOC domains are off. So only peripherals in the AON domain can wake up the chip. 
    

    When the chip wakes up from deep-sleep mode, it will do the boot process. As system SRAM and CPU are shut down in deep-sleep mode, the corresponding interrupt of the peripheral which is set as the wake source should be registered again after wakeup to process the interrupt handler. Figure 1-5 shows deep-sleep mode flow.
    
    
    
    Figure 1-5 Deep-sleep mode flow
    
Wakeup Source
~~~~~~~~~~~~~~~~~~~~~~~~~~
Table 1-2 lists the wakeup sources that can be used to wake up the system under different power modes.

Table 1-2 Wakeup source

+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| Wakeup source  | Sleep CG | Sleep PG | Deep-sleep | Comment                                                                                            |
+================+==========+==========+============+====================================================================================================+
| WLAN           | √        | √        | X          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| BT             | √        | √        | X          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| IWDG           | √        | √        | X          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| IPC            | √        | √        | X          | Only KM0 can use IPC to wake up KM4                                                                |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| Basic Timer4~7 | √        | √        | X          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| PMC Timer      | √        | √        | X          | For internal usage                                                                                 |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| UART0~2        | √        | √        | X          | Need to keep OSC4M on during sleep, not recommended to use when the baudrate is larger than 115200 |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| LOGUART        | √        | √        | X          | Need to keep XTAL on during sleep                                                                  |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| GPIO           | √        | √        | X          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| I2C            | √        | √        | X          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| Cap-Touch      | √        | √        | X          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| ADC            | √        | √        | X          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| ADC comparator | √        | √        | X          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| SDIO           | √        | √        | X          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| Key-Scan       | √        | √        | X          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| BOR            | √        | √        | X          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| PWR_DOWN       | √        | √        | √          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| AON_TIMER      | √        | √        | √          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| AON_WAKEPIN    | √        | √        | √          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+
| RTC            | √        | √        | √          |                                                                                                    |
+----------------+----------+----------+------------+----------------------------------------------------------------------------------------------------+

Entering Sleep Mode
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Sleep mode is based on FreeRTOS tickless, thus it is recommended to enter sleep mode by releasing the wakelock.

1. Initialize the specific peripheral.

2. Enable and register the peripheral's interrupt.

3. Set sleep_wevent_config[] in ambea_sleepcfg.c, and the interrupt should be registered on the same CPU selected by sleep_wevent_config[].

4. For peripherals that need special clock settings, set ps_config[] in ameba_sleepcfg.c if needed.

5. Register sleep/wakeup callback if needed.

6. Enter sleep mode by releasing the wakelock in KM4 (PMU_OS needs to be released since it is acquired by default when boot).

7. Clear the peripheral's interrupt when wakeup.


For peripherals that need specific clock settings, such as UART and LOGUART, their setting flows are described in 1.3.1 and 1.3.2.

UART
^^^^^^^^
- When using UART as a wakeup source, clock OSC4M should not be closed when the system is in sleep mode.

- When the baudrate is larger than 115200, it is not recommended to use UART as a wakeup source.


Configuration:

1. Initialize UART and enable its interrupt.

2. Set the related wakeup source (WAKE_SRC_UART0/WAKE_SRC_UART1/WAKE_SRC_UART2_BT) in sleep_wevent_ config[] to WAKEUP_KM4 or WAKEUP_KM0 (based on which CPU you want to wake). The interrupt should be registered on the same CPU selected by sleep_wevent_config[].

3. Set the corresponding entry of uart_config[] in ameba_sleepcfg.c to "ENABLE".

4. Set keep_OSC4M_on in ps_config[] to "TRUE" to keep OSC4M enabled during sleep mode.

5. Enter sleep mode by releasing the wakelock in KM4 (PMU_OS needs to be released since it is acquired by default when boot).

6. Clear the UART interrupt when wakeup.

LOGUART
^^^^^^^^^^^^^^
When using LOGUART as a wakeup source, XTAL should not be closed during sleep.


Configuration:

1. Initialize LOGUART and enable its interrupt.

2. Set WAKE_SRC_UART_LOG in sleep_wevent_config[] to WAKEUP_KM4 or WAKEUP_KM0 (based on which CPU you want to wake). The interrupt should be registered on the same CPU selected by sleep_wevent_config[].

3. Set xtal_mode_in_sleep to XTAL_Normal in ps_config[].

4. Enter sleep mode by releasing the wakelock in KM4 (PMU_OS needs to be released since it is acquired by default when boot).

5. Clear the LOGUART interrupt when wakeup.

Entering Deep-Sleep Mode
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Deep-sleep can also be entered from FreeRTOS tickless flow.


When the system boots, KM4 holds the deepwakelock PMU_OS, thus freertos_ready_to_dsleep() will be checked fail and the system does not enter deep-sleep mode in idle task by default. Since freertos_ready_to_dsleep() will be checked only after freertos_ready_to_sleep() is checked pass, both the wakelock and deepwakelock need to be released for entering deep-sleep mode.


Configuration:

1. Initialize the related peripheral and enable its interrupt.

2. Set sleep_wakepin_config[] in ameba_sleepcfg.c when using AON wakepin as a wakeup source.

3. Enter deep-sleep mode by releasing the deepwakelock and wakelock in KM4.

Power-Saving Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Wakeup Mask Setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
For sleep mode, only one CPU is required to wake up to execute the program in some situations. The wakeup mask module is designed to implement this function. By setting a wakeup mask, you can choose to wake up only KM0, or KM4. If KM4 is chosen, KM0 will be waked up first and then KM0 will resume KM4.


Users can set the wakeup attribute in sleep_wevent_config[] in ameba_sleepcfg.c to choose which CPU you want to wake up. The wakeup attribute of each wakeup source can be set to WAKEUP_KM4 or WAKEUP_KM0 or WAKEUP_NULL, respectively indicating that this wakeup source is only to wake up KM4, or wake up KM0, or not used as a wakeup source.

.. code::

   /* Wakeup entry can be set to WAKEUP_NULL/WAKEUP_KM4/WAKEUP_KM0 */
   WakeEvent_TypeDef sleep_wevent_config[] = {
   //     Module              Wakeup
     {WAKE_SRC_SDIO,          WAKEUP_NULL},
     {WAKE_SRC_AON_WAKEPIN,      WAKEUP_NULL},
     {WAKE_SRC_AON_TIM,        WAKEUP_NULL},
     {WAKE_SRC_Keyscan,        WAKEUP_NULL},
     {WAKE_SRC_PWR_DOWN,        WAKEUP_NULL},
     {WAKE_SRC_BOR,          WAKEUP_NULL},
     {WAKE_SRC_ADC_COMP,        WAKEUP_NULL},
     {WAKE_SRC_ADC,          WAKEUP_NULL},
     {WAKE_SRC_RTC,          WAKEUP_NULL},
     {WAKE_SRC_CTOUCH,        WAKEUP_NULL},
     {WAKE_SRC_I2C1,          WAKEUP_NULL},
     {WAKE_SRC_I2C0,          WAKEUP_NULL},
     {WAKE_SRC_GPIOB,        WAKEUP_NULL},
     {WAKE_SRC_GPIOA,        WAKEUP_NULL},
     {WAKE_SRC_UART_LOG,        WAKEUP_NULL},
     {WAKE_SRC_UART2_BT,        WAKEUP_NULL},
     {WAKE_SRC_UART1,        WAKEUP_NULL},
     {WAKE_SRC_UART0,        WAKEUP_NULL},
     {WAKE_SRC_pmc_timer1,      WAKEUP_KM0},  /* Internal use, do not change it*/
     {WAKE_SRC_pmc_timer0,      WAKEUP_KM4},  /* Internal use, do not change it*/
     {WAKE_SRC_Timer7,        WAKEUP_NULL},
     {WAKE_SRC_Timer6,        WAKEUP_NULL},
     {WAKE_SRC_Timer5,        WAKEUP_NULL},
     {WAKE_SRC_Timer4,        WAKEUP_NULL},
     {WAKE_SRC_IWDG0,        WAKEUP_NULL},
     {WAKE_SRC_IPC_KM4,        WAKEUP_KM4},  /* IPC can only wake up KM4, do not change it*/
     {WAKE_SRC_BT_WAKE_HOST,      WAKEUP_NULL},
     {WAKE_SRC_KM4_WAKE_IRQ,      WAKEUP_KM0},  /* Internal use, do not change it*/
     {WAKE_SRC_WIFI_FTSR_MAILBOX,  WAKEUP_KM0},  /* Wi-Fi wakeup, do not change it*/
     {WAKE_SRC_WIFI_FISR_FESR_IRQ,  WAKEUP_KM0},  /* Wi-Fi wakeup, do not change it*/
     {0xFFFFFFFF,          WAKEUP_NULL},
   };
AON Wakepin Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
AON wakepin is one of the peripherals that can be set as a wakeup source. The |CHIP_NAME| has two AON wakepins (PB30 and PB31), which can be configured in sleep_wakepin_config[] in ameba_sleepcfg.c. The config attribute can be set to DISABLE_WAKEPIN or HIGH_LEVEL_WAKEUP or LOW_LEVEL_WAKEUP, meaning not wake up, or GPIO level high will wake up, or GPIO level low will wake up respectively.

.. code::

   /* can be used by sleep mode & deep sleep mode */
   /* config can be set to DISABLE_WAKEPIN/HIGH_LEVEL_WAKEUP/LOW_LEVEL_WAKEUP */
   WAKEPIN_TypeDef sleep_wakepin_config[] = {
   //   wakepin      config
     {WAKEPIN_0,    DISABLE_WAKEPIN},  /* WAKEPIN_0 corresponding to _PB_30 */
     {WAKEPIN_1,    DISABLE_WAKEPIN},  /* WAKEPIN_1 corresponding to _PB_31 */
     {0xFFFFFFFF,  DISABLE_WAKEPIN},
   };


.. note::
      - By default, AON_WAKEPIN_IRQ will not be enabled in sleep_wakepin_config[], and users need to enable it by themselves.


   - The wakeup mask will not be set in sleep_wakepin_config[]. If wakepin is used for sleep mode, WAKE_SRC_AON_ WAKEPIN entry needs to be set in sleep_wevent_config[].

Clock and Voltage Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The XTAL, OSC4M state, and sleep mode voltage are configurable in ps_config[] in ameba_sleepcfg.c. Users can use this configuration for peripherals that need XTAL or OSC4M on in sleep mode.

.. code::

   PSCFG_TypeDef ps_config = {
     .keep_OSC4M_on = FALSE,        /* Keep OSC4M on or off for sleep */
     .xtal_mode_in_sleep = XTAL_OFF,    /* Set XTAL mode during sleep mode, see enum xtal_mode_sleep for details */
     .sleep_to_08V = FALSE,        /* Default sleep to 0.7V, setting this option to TRUE will sleep to 0.8V */
   };
Sleep Type Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
KM4 can set sleep mode to CG or PG by calling the function pmu_set_sleep_type(uint32_t type), and users can get CPU's sleep mode by calling the function pmu_get_sleep_type().



.. note::
      - KM0 and KM4 are in the same power domain, so they will have the same sleep type, thus pmu_set_sleep_type() should be set to KM4, and KM0 will follow KM4's sleep mode type.

      - Sleep mode is set to PG by default. If users want to change the sleep type, pmu_set_sleep_type() needs to be called before sleep.


Power-Saving Related APIs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Wakelock APIs
^^^^^^^^^^^^^^^^^^^^^^^^^^
In some situations, the system needs to keep awake to receive certain events; otherwise, events may be missed when the system is in sleep mode. An idea of wakelock is introduced in that the system cannot enter sleep mode if some module is holding the wakelock.


Wakelock is a 32-bit map. Each module has its own bit in the wakelock bit map (see enum. PMU_DEVICE). Users can also add the wakelock in the enum. If the wakelock bit map equals zero, it means that there is no module holding the wakelock. If the wakelock bit map is larger than zero, it means that some modules are holding the wakelock, and the system is not allowed to enter sleep mode.


Wakelock is a judging condition in the function freertos_ready_to_sleep(). When the system boots, KM4 holds the wakelock PMU_OS, and KM0 holds the wakelock PMU_OS and PMU_KM4_RUN. Only if all wakelocks are released, KM4 or KM0 is permitted to enter sleep mode. The function freertos_ready_to_sleep() will judge the value of wakelock.

.. code::


   typedef enum {
     PMU_OS        = 0,
     PMU_WLAN_DEVICE,
     PMU_KM4_RUN,
     PMU_WLAN_FW_DEVICE,
     PMU_BT_DEVICE,
     PMU_DEV_USER_BASE,
     PMU_MAX
   } PMU_DEVICE;
It is recommended to enter sleep mode by releasing the wakelock. After the wakelock PMU_OS of KM4 is released, KM4 will enter sleep mode in idle task and send IPC to KM0. KM0 will gate KM4's clock first and then release PMU_OS and PMU_KM4_RUN.


When the system wakes, it will enter sleep mode again quickly unless it acquires the wakelock.


Similar to the wakelock for sleep mode, there is a 32-bit deepwakelock map for deep-sleep mode. If the deepwakelock bit map is larger than zero, it means that some modules are holding the deepwakelock, and the system is not allowed to enter deep-sleep mode. When the system boots, KM4 holds the deepwakelock PMU_OS.

Deepwakelock is a judging condition in the function freertos_ready_to_dsleep(). After the deepwakelock PMU_OS of KM4 is released, and all wakelocks of KM4 are released, KM4 will be allowed to enter deep-sleep mode and send IPC to KM0 in idle task. KM0 will send a deep-sleep request and let the chip finally enter deep-sleep mode.

APIs in the following sections are provided to control the wakelock or deepwakelock.

pmu_acquire_wakelock
****************************************
+-------------------+--------------------------------------------------+
| Items             | Description                                      |
+===================+==================================================+
| Introduction      | Acquire the wakelock for one module              |
+-------------------+--------------------------------------------------+
| Parameter         | nDeviceId: Device ID of the corresponding module |
|                   |                                                  |
|                   | .. code::                                        |
+-------------------+--------------------------------------------------+
| typedef enum {    | None                                             |
|      PMU_OS  = 0, |                                                  |
|      …            |                                                  |
|      PMU_MAX      |                                                  |
|    } PMU_DEVICE;  |                                                  |
| Return            |                                                  |
+-------------------+--------------------------------------------------+

pmu_release_wakelock
****************************************
+-------------------+--------------------------------------------------+
| Items             | Description                                      |
+===================+==================================================+
| Introduction      | Release the wakelock for one module              |
+-------------------+--------------------------------------------------+
| Parameter         | nDeviceId: Device ID of the corresponding module |
|                   |                                                  |
|                   | .. code::                                        |
+-------------------+--------------------------------------------------+
| typedef enum {    | None                                             |
|      PMU_OS  = 0, |                                                  |
|      …            |                                                  |
|      PMU_MAX      |                                                  |
|    } PMU_DEVICE;  |                                                  |
| Return            |                                                  |
+-------------------+--------------------------------------------------+

pmu_acquire_deepwakelock
************************************************
+-------------------+--------------------------------------------------+
| Items             | Description                                      |
+===================+==================================================+
| Introduction      | Acquire the deepwakelock for one module          |
+-------------------+--------------------------------------------------+
| Parameter         | nDeviceId: Device ID of the corresponding module |
|                   |                                                  |
|                   | .. code::                                        |
+-------------------+--------------------------------------------------+
| typedef enum {    | None                                             |
|      PMU_OS  = 0, |                                                  |
|      …            |                                                  |
|      PMU_MAX      |                                                  |
|    } PMU_DEVICE;  |                                                  |
| Return            |                                                  |
+-------------------+--------------------------------------------------+

pmu_release_deepwakelock
************************************************
+-------------------+--------------------------------------------------+
| Items             | Description                                      |
+===================+==================================================+
| Introduction      | Release the deepwakelock for one module          |
+-------------------+--------------------------------------------------+
| Parameter         | nDeviceId: Device ID of the corresponding module |
|                   |                                                  |
|                   | .. code::                                        |
+-------------------+--------------------------------------------------+
| typedef enum {    | None                                             |
|      PMU_OS  = 0, |                                                  |
|      …            |                                                  |
|      PMU_MAX      |                                                  |
|    } PMU_DEVICE;  |                                                  |
| Return            |                                                  |
+-------------------+--------------------------------------------------+

pmu_set_sysactive_time
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In some situations, the system needs to keep awake for a period of time to complete a certain process while the system is active.


+--------------+--------------------------------------------------------------+
| Items        | Description                                                  |
+==============+==============================================================+
| Introduction | Set a period of time that the system will keep active        |
+--------------+--------------------------------------------------------------+
| Parameter    | timeout: time value, unit is ms.                             |
|              |                                                              |
|              | The system will keep active for this time value from now on. |
+--------------+--------------------------------------------------------------+
| Return       | 0                                                            |
+--------------+--------------------------------------------------------------+

Sleep/Wake Callback APIs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
These APIs are used to register suspend/resume callback function for <nDeviceId>. The suspend callback function will be called in idle task before the system enters sleep mode, and the resume callback function will be called after the system resumes.


It is a good way to use the suspend and resume function if there is something to do before the chip sleeps or after the chip wakes. A typical application of the resume function is to acquire the wakelock to prevent the chip from sleeping again. Also, if the CPU chooses PG, some peripherals will lose power so they need to be reinitialized. This can be implemented in the resume function.



.. note::
      - Yield OS is not permitted in the suspend/resume callback functions, thus RTOS APIs which may cause OS yield such as rtos_task_yield, rtos_time_delay_ms, or mutex, semaphore related APIs are not recommended to use.

      - pmu_set_sysactive_time is not permitted in the suspend callback function, but permitted in the resume callback function.


pmu_register_sleep_callback
******************************************************
+--------------+--------------------------------------------------------------+
| Items        | Description                                                  |
+==============+==============================================================+
| Introduction | Register the suspend/resume callback function for one module |
+--------------+--------------------------------------------------------------+
| Parameter    | - nDeviceId: Device ID needs suspend/resume callback         |
|              |                                                              |
|              | .. code::                                                    |
|              |                                                              |
|              |    typedef enum {                                            |
|              |      PMU_OS  = 0,                                            |
|              |      …                                                       |
|              |      PMU_MAX                                                 |
|              |    } PMU_DEVICE;                                             |
|              | - sleep_hook_fun: Suspend callback function                  |
|              |                                                              |
|              | - sleep_param_ptr: Suspend callback function parameter       |
|              |                                                              |
|              | - wakeup_hook_fun: Resume callback function                  |
|              |                                                              |
|              | - wakeup_param_ptr: Resume callback function parameter       |
+--------------+--------------------------------------------------------------+
| Return       | None                                                         |
+--------------+--------------------------------------------------------------+

pmu_unregister_sleep_callback
**********************************************************
+--------------+--------------------------------------------------------------+
| Items        | Description                                                  |
+==============+==============================================================+
| Introduction | Register the suspend/resume callback function for one module |
+--------------+--------------------------------------------------------------+
| Parameter    | - nDeviceId: Device ID needs suspend/resume callback         |
|              |                                                              |
|              | .. code::                                                    |
|              |                                                              |
|              |    typedef enum {                                            |
|              |      PMU_OS  = 0,                                            |
|              |      …                                                       |
|              |      PMU_MAX                                                 |
|              |    } PMU_DEVICE;                                             |
|              | - sleep_hook_fun: Suspend callback function                  |
|              |                                                              |
|              | - sleep_param_ptr: Suspend callback function parameter       |
|              |                                                              |
|              | - wakeup_hook_fun: Resume callback function                  |
|              |                                                              |
|              | - wakeup_param_ptr: Resume callback function parameter       |
+--------------+--------------------------------------------------------------+
| Return       | None                                                         |
+--------------+--------------------------------------------------------------+

pmu_set_max_sleep_time
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
+--------------+-----------------------------------------------------+
| Items        | Description                                         |
+==============+=====================================================+
| Introduction | Set the maximum sleep time                          |
+--------------+-----------------------------------------------------+
| Parameter    | timer_ms: system maximum sleep timeout, unit is ms. |
+--------------+-----------------------------------------------------+
| Return       | None                                                |
+--------------+-----------------------------------------------------+



.. note::
      - The system will be woken up after the timeout.

      - The system may be woken up by other wake events before the timeout.

      - This setting only works once. The timer will be cleared after the system wakes up.


Wakeup Reason APIs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
SOCPS_AONWakeReason
**************************************
+--------------+----------------------------------+
| Items        | Description                      |
+==============+==================================+
| Introduction | Get the deep-sleep wakeup reason |
+--------------+----------------------------------+
| Parameter    | None                             |
+--------------+----------------------------------+
| Return       | Bit[0]: CHIP_EN short press      |
|              |                                  |
|              | Bit[1]: CHIP_EN long press       |
|              |                                  |
|              | Bit[2]: BOR                      |
|              |                                  |
|              | Bit[3]: AON Timer                |
|              |                                  |
|              | Bit[5:4]: AON GPIO               |
|              |                                  |
|              | Bit[8]: RTC                      |
+--------------+----------------------------------+

WAK_STATUS0
**********************
The following register can be used to get the sleep wakeup reason.


+-------------+--------------------------+
| Register    | Parameters               |
+=============+==========================+
| WAK_STATUS0 | Bit[1:0]: WLAN           |
|             |                          |
|             | Bit[2]: KM4_WAKE         |
|             |                          |
|             | Bit[3]: BT_WAKE_HOST     |
|             |                          |
|             | Bit[4]: IPC_KM4          |
|             |                          |
|             | Bit[5]: IWDG0            |
|             |                          |
|             | Bit[9:6]: BASIC TIMER4~7 |
|             |                          |
|             | Bit[11:10]: PMC TIMER0~1 |
|             |                          |
|             | Bit[14:12]: UART0~2      |
|             |                          |
|             | Bit[15]: UART_LOG        |
|             |                          |
|             | Bit[16]: GPIOA           |
|             |                          |
|             | Bit[17]: GPIOB           |
|             |                          |
|             | Bit[19:18]:I2C0~1        |
|             |                          |
|             | Bit[20]: Cap-Touch       |
|             |                          |
|             | Bit[21]: RTC             |
|             |                          |
|             | Bit[22]: ADC             |
|             |                          |
|             | Bit[23]: ADC_COMP        |
|             |                          |
|             | Bit[24]: BOR             |
|             |                          |
|             | Bit[25]: PWR_DOWN        |
|             |                          |
|             | Bit[26]: Key-Scan        |
|             |                          |
|             | Bit[27]: AON_Timer       |
|             |                          |
|             | Bit[28]: AON_Wakepin     |
|             |                          |
|             | Bit[29]: SDIO            |
+-------------+--------------------------+


Note that when wakeup, the corresponding peripheral interrupt will be raised; when clearing the interrupt, the corresponding bit in wakeup reason will also be cleared. Thus it is not possible to get the wakeup reason after the interrupt is cleared.

Wi-Fi Power Saving
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
IEEE 802.11 power save management allows the station to enter its own sleep state. It defines that the station needs to keep awake at a certain timestamp and enter a sleep state otherwise.


WLAN driver acquires the wakelock to avoid the system entering sleep mode when WLAN needs to keep awake. And it releases the wakelock when it is permitted to enter the sleep state.


IEEE 802.11 power management allows the station to enter power-saving mode. The station cannot receive any frame during power saving. Thus AP needs to buffer these frames and requires the station to periodically wake up to check the beacon which has the information of buffered frames.

.. image:: ../_static/power_save_rst/4bd9d8f37d0040ab8c65cdb22167bcb3a613e0f1.png
   :width: 1482
   :align: center


Figure 1-6 Timeline of power saving

When the system is active, and Wi-Fi is connected and enters IEEE 802.11 power management mechanism, this is called LPS in SDK; if the system enters sleep mode when Wi-Fi is connected, we call it WoWLAN mode.


In WoWLAN mode, a timer with a period of about 102ms will be set in the suspend function, and KM0 will wake up every 102ms to receive the beacon to maintain the connection. 


Except LPS and WoWLAN modes, there is also an IPS mode, which can be used when Wi-Fi is not connected. Table 1-3 lists all three power-saving modes for Wi-Fi. Table 1-4 describes the relationship between power modes of the system and Wi-Fi.

Table 1-3 Wi-Fi power-saving modes

+--------+----------------+----------------------------------------+-------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------+
| Mode   | Wi-Fi status   | Wi-Fi status                           | Description                                                                               | SDK                                                                                 |
+========+================+========================================+===========================================================================================+=====================================================================================+
| IPS    | Not associated | RF/BB/MAC OFF                          | Wi-Fi driver automatically turns Wi-Fi off to save power.                                 | IPS mode is enabled in SDK by default and is not recommended to be disabled.        |
+--------+----------------+----------------------------------------+-------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------+
| LPS    | Associated     | - RF periodically ON/OFF               | LPS mode is used to implement IEEE 802.11 power management.                               | LPS mode is enabled in SDK by default but can be disabled through API in Table 1-5. |
|        |                |                                        |                                                                                           |                                                                                     |
|        |                | - MAC/BB always ON                     | NP will control RF ON/OFF based on TSF and TIM IE in the beacon.                          |                                                                                     |
+--------+----------------+----------------------------------------+-------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------+
| WoWLAN | Associated     | - RF and BB periodically ON/OFF        | NP is waked up at each beacon early interrupt to receive a beacon from the associated AP. | WoWLAN mode is enabled in SDK by default.                                           |
|        |                |                                        |                                                                                           |                                                                                     |
|        |                | - MAC periodically enters/ exits CG/PG | NP will wake up AP when receiving a data packet.                                          |                                                                                     |
+--------+----------------+----------------------------------------+-------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------+

Table 1-4 Relationship between power modes of system and Wi-Fi

+-------------------+------------------+------------------------------------------------------------------------+
| System power mode | Wi-Fi power mode | Description                                                            |
+===================+==================+========================================================================+
| Active            | IPS              | Wi-Fi is on, but not connected                                         |
+-------------------+------------------+------------------------------------------------------------------------+
| Active            | LPS              | Wi-Fi is connected and enters IEEE 802.11 power management mechanism   |
+-------------------+------------------+------------------------------------------------------------------------+
| Sleep             | Wi-Fi OFF/IPS    |                                                                        |
+-------------------+------------------+------------------------------------------------------------------------+
| Sleep             | WoWLAN           | Wi-Fi keeps associating.                                               |
+-------------------+------------------+------------------------------------------------------------------------+
| Deep-sleep        | Wi-Fi OFF        | Deep-sleep is not recommended if Wi-Fi needs to keep on or associated. |
+-------------------+------------------+------------------------------------------------------------------------+

Table 1-5 API to enable/disable LPS

+------------------------------------+----------------------+
| API                                | Parameters           |
+====================================+======================+
| int wifi_set_lps_enable(u8 enable) | Parameter: enable    |
|                                    |                      |
|                                    | - TRUE: enable LPS   |
|                                    |                      |
|                                    | - FALSE: disable LPS |
+------------------------------------+----------------------+


When Wi-Fi is connected and the system enters sleep mode, WoWLAN mode will be entered automatically, and KM0 will periodically wake up to receive the beacon to maintain the connection, this will consume some power. If you are more concerned about the system power consumption during sleep mode, and Wi-Fi is not a necessary function in your application, it is recommended to set Wi-Fi off or choose Wi-Fi IPS mode.

