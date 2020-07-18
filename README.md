This repository consists of code for generating a lookup table (LUT) to be used for correcting ADC non-linearity issue. An example is also given on how to apply the LUT in your program.

The code is based on original work from [Helmut Weber](https://github.com/MacLeod-D/ESP32-ADC) that he first described at [ESP32 discussion forum](https://esp32.com/viewtopic.php?f=19&t=2881&start=30#p47663), but modified with bug-fixed by me.

### ESP32 Linearity issue

To address the ESP32 ADC non-linear issue, a lookup table is used to correct the non-linearity. You may need to generate your own lookup table as it varies from device to device due to the variation of ESP32 internal reference voltage.

[![ESP32 ADC linearity](https://github.com/e-tinkers/esp32-adc-calibrate/blob/master/images/esp32_ADClinearity.png)](https://github.com/e-tinkers/esp32-adc-calibrate/blob/master/images/esp32_ADClinearity.png)

### Things need to know before using LUT

It is recommended to read the [ESP32 Analog to Digital Converter](https://docs.espressif.com/projects/esp-idf/en/latest/api-reference/peripherals/adc.html) before using the LUT approach for solving the ESP32 linearity issue. According to documentation, ESP32 chips(ESP32-D0WD and ESP32-D0WDQ6) that manufactured after 1st week of 2018 have been individual measurement and burned with the eFuse Vref value on to the chip. The eFuse Vref can be read using the function call `read_efuse_vref(void)` which you can find the source code at [esp_adc_cal.c](https://github.com/espressif/esp-idf/blob/f91080637c054fa2b4107192719075d237ecc3ec/components/esp_adc_cal/esp_adc_cal.c#L153).

There is a function [`calculate_voltage_linear()`](https://github.com/espressif/esp-idf/blob/f91080637c054fa2b4107192719075d237ecc3ec/components/esp_adc_cal/esp_adc_cal.c#L246) in the same library which uses a polynomial formula to correct the linearity.

It is your choice on whether you want to implement the polynomial fitting method or LUT method to address your ESP32 ADC linearity problem. The LUT take up a lot of memory (which is not a big issue for ESP32) but it is faster and more accurate than the polynomial fitting approach.

### How to generate the LUT?

This program use the ESP32 DAC to generate a value as the reference and feed it into the ADC for calibration. Remember to use a short jumper wire to connect DAC output(GPIO 25) to ADC Channel 1 ADC7(GPIO 35) on your ESP32.

By default, the program will generate a `float ADC_LUT` which take up much more memory but with better precision, if you want to have an `int ADC_LUT` table, comment out the line `#define FLOAT_LUT` to get the integer table.

1. Connect a jumper wire between GPIO 25 and GPIO 35 (i.e. A7);
2. decide on what table (float or integer) you'd want to generate;
3. Start the `main.cpp` sketch from PlatformIO or Arduino IDE;
4. When the program stops, copy and paste the entire table from Serial Monitor to your sketch to use it.

### How to use the LUT?

Refer to the sketch in example directory on how to apply the LUT in your program. Run the sketch and see the result on Serial Plotter.

Here is the results between raw reading from ESP32 ADC against the calibrated reading with LUT.

[![rawReading versus calbratedReading](https://github.com/e-tinkers/esp32-adc-calibrate/blob/master/images/rawReading_versus_calibratedReading.png)](https://github.com/e-tinkers/esp32-adc-calibrate/blob/master/images/rawReading_versus_calibratedReading.png)

and here is the DAC output against calibrated ADC reading.

[![DAC output versus ADC calbratedReading](https://github.com/e-tinkers/esp32-adc-calibrate/blob/master/images/DAC_output_versus_adcCalibratedReading.png)](https://github.com/e-tinkers/esp32-adc-calibrate/blob/master/images/DAC_output_versus_adcCalibratedReading.png)
