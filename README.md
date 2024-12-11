# STM32G474RE `printf` Error on UART Callback

This project is to demonstrates the error in BSP of Nucleo-G474RE for ease of investigation for the maintainer on [this post](https://community.st.com/t5/stm32-mcus-embedded-software/stm32g4-bsp-s-virtual-com-port-fails-to-print-character-when/m-p/749213#M30327).
On line `386` in file `Drivers/BSM/STM32G4xx_Nucleo/stm32g4xx_nucleo.c`:

```c
#if (USE_HAL_UART_REGISTER_CALLBACKS == 0)
    /* Init the UART Msp */
    COM1_MspInit(&hcom_uart[COM]);
#else
    if (IsComMspCbValid == 0U) /// THIS IS THE ISSUE!!!!!!!!!!!!!!!!!!!!!!!!!!!
    {
      if (BSP_COM_RegisterDefaultMspCallbacks(COM) != BSP_ERROR_NONE)
      {
        return BSP_ERROR_MSP_FAILURE;
      }
    }
#endif
```

The `IsComMspCbValid` is an array, which is basically an address.
Here, it is believed that someone forget to index it, so `IsComMspCbValid == 0U` will always evaluate to `false`.
This causes `BSP_COM_RegisterDefaultMspCallbacks(COM)` not called, which means the `COM` is never initialized correctly, thus the lack of response in `printf`.

To fix this, `IsComMspCbValid` should be indexed, it should be `IsComMspCbValid[COM] == 0U`.
