.. _sdk_examples:

SDK Examples
------------------------
There are two kinds of examples in the SDK of |CHIP_NAME|: application examples and peripheral examples. This chapter illustrates the contents of examples and how to build example source code. The path and description of SDK examples are listed in Table 1\-1.

Table 1\-1 Examples in SDK

+---------------------+------------------------------------+-------------------------------+
| Items               | Path                               | Description                   |
+=====================+====================================+===============================+
| Application example | {SDK}\component\example            | xml, ssl, …                   |
+---------------------+------------------------------------+-------------------------------+
| Peripheral example  | {SDK}\component\example\peripheral | ADC, UART, I2C, SPI, Timer, … |
+---------------------+------------------------------------+-------------------------------+

Application Example
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
In each folder of application example, there are C source files, header files and README.txt. You should check for detailed configurations of the example according to README.txt.



.. note::
   The application examples are shared by all Realtek SoC, so you need to refer to README.txt for detailed information of |CHIP_NAME|.



The application examples normally run on KM4. The entry function of application example is app_example() in \ ``main.c``\  under \ ``{SDK}\``\  \ ``amebadplus_gcc_project\project_km4\src``\ . Each application example has its own app_example(), and the app_example() in \ ``main.c``\  will be replaced automatically when the application example is built.

.. image:: ../_static/sdk_example_rst/25a5147b5a9e3fc69169961460ab167ab5b68b20.png
   :width: 241
   :align: center


Figure 1\-2 Entry function of application example

To run application example, you only need to:

1. Check software and hardware settings in README.txt of the example.

2. Add compile options "\ ``EXAMPLE\={example folder``\  \ ``name}``\ " when building the project, and replace \ ``{example``\  \ ``folder``\  \ ``name}``\  with the specific folder name of this example.


For example, if you want to build xml example to start an xml example thread, you need to set the macro in SDK according to README.txt in \ ``{SDK}\component\example\xml``\ . Then enter "\ ``make EXAMPLE\=xml``\ " for KM4 on MSYS2 MinGW 64\-bit (Windows) or terminal (Linux).

.. image:: ../_static/sdk_example_rst/5609433edd23d6866671138b31d8ba392edfc78f.png
   :width: 894
   :align: center


Figure 1\-3 Building XML application example

Peripheral Example
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The peripheral examples are demos of peripherals. Most examples consist of raw and mbed folders, you can choose raw or mbed demos as you like.

Table 1\-2 Comparison of raw and mbed examples

+-------+------------------------------------------------------+----------------------------------+
| Items | Path                                                 | Description                      |
+=======+======================================================+==================================+
| mbed  | {SDK}\component\example\peripheral\{peripheral}\mbed | mbed APIs are used.              |
+-------+------------------------------------------------------+----------------------------------+
| raw   | {SDK}\component\example\peripheral\{peripheral}\raw  | Low\-level driver APIs are used. |
+-------+------------------------------------------------------+----------------------------------+


Each example folder has main.c and README.txt. There are example descriptions, required components, HW connection and expected behavior in README.txt.


The peripheral examples normally run on KM4. To run peripheral examples, you only need to:

1. Check software and hardware settings in README.txt of the example.

2. Replace the original \ ``main.c``\  under \ ``{SDK}\amebadplus_gcc_project\project_km4\src``\  with \ ``main.c``\  in the example directory.

3. (Optional) Copy other header files that are depicted in the README.txt to \ ``{SDK}\amebadplus_gcc_project\project_km4\src``\  if needed.

4. Re\-build the project by command "\ ``make``\ ".

