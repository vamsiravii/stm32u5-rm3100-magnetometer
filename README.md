# RM3100 Magnetometer Driver for STM32U5

A small I2C driver for the PNI **RM3100** geomagnetic sensor, written against the STM32 HAL for an **STM32U575ZI**, plus a set of test routines you can watch live in STM32CubeIDE.

## What it does

- Reads all three axes (X, Y, Z) over I2C
- Single-shot and continuous measurement modes, with nine selectable update rates from about 2.3 Hz to 600 Hz
- Adjustable cycle count (resolution vs. speed trade-off)
- Optional DRDY pin for interrupt-style readiness, with automatic fallback to polling the status register
- Converts raw 24-bit counts to microtesla

## Files

| File | Purpose |
|---|---|
| `Core/Inc/rm3100.h`, `Core/Src/rm3100.c` | The driver: register map, handle struct and public API |
| `Core/Src/rm3100_example.c` | Copy-paste usage snippets and wiring notes |
| `Core/Inc/rm3100_test.h`, `Core/Src/rm3100_test.c` | Three bring-up tests (single-shot, continuous stream of 50 samples, raw vs. microtesla), results exposed as globals for CubeIDE *Live Expressions* |
| `mag.ioc` | CubeMX configuration (I2C1 on PB6/PB9, UART4 on PA0/PA1) |

The rest of `Core/` and `Drivers/` is CubeMX-generated HAL code.

## Quick start

1. Open the project in **STM32CubeIDE** and let it build.
2. Wire the sensor to I2C1 (SCL on PB6, SDA on PB9, both with pull-ups). The RM3100's `I2CEN` pin must be pulled high to select I2C mode. Full pin notes are at the top of [`rm3100_example.c`](Core/Src/rm3100_example.c).
3. Call the test suite from `main()` after `MX_I2C1_Init()`:

   ```c
   RM3100_Test_RunAll(&hi2c1, RM3100_I2C_ADDR_00);
   ```

   Note that `main.c` does not call the driver yet. As committed it only initializes the peripherals.
4. Start a debug session, add the variables listed at the top of `rm3100_test.c` to *Live Expressions*, and watch `test_status`.

## Using the driver

```c
RM3100_HandleTypeDef mag = {
    .hi2c     = &hi2c1,
    .i2cAddr  = RM3100_I2C_ADDR_00,   // both address pins to GND
    .drdyPort = NULL,                 // NULL = poll the status register
};

if (RM3100_Init(&mag) == RM3100_OK) {
    RM3100_SetCycleCount(&mag, RM3100_CC_200);
    RM3100_StartCMM(&mag, RM3100_TMRC_37HZ);

    RM3100_Data raw;
    if (RM3100_DataReady(&mag) && RM3100_ReadData(&mag, &raw) == RM3100_OK) {
        float x_uT = RM3100_ToMicroTesla(raw.x, 200);
    }
}
```

## Status

Work in progress. The driver, an example and bring-up tests are in place. The project's `main.c` does not use them yet.

## Hardware

- MCU: STM32U575ZITxQ
- Sensor: PNI RM3100 (I2C, address `0x20` with both address pins grounded)
- Toolchain: STM32CubeIDE
