# USB Joystick Mouse with STM32

This project demonstrates how to use an STM32 microcontroller to emulate a USB mouse using a **joystick module**.  
Two ADC channels are used to read the joystick position, and DMA is configured in **circular mode** for continuous sampling.  
A push button is also implemented to simulate the mouse click.  

The STM32 USB peripheral is configured as a **HID device**, and reports are sent to the PC to control the cursor.

---

## Features
- USB HID (Human Interface Device) Mouse implementation
- Joystick control using **2 ADC channels** with DMA (circular mode)
- Button press via **external interrupt** (GPIO with EXTI)
- Cursor movement with joystick axes
- Mouse click simulated by pressing the button

---

## Hardware Requirements
- STM32 microcontroller with USB FS support (e.g., STM32F103C8T6)
- Joystick module (with X and Y analog outputs)
- Push button
- USB cable

**Connections:**
| Component      | STM32 Pin |
|----------------|-----------|
| Joystick X     | PA0 (ADC1_IN0) |
| Joystick Y     | PA1 (ADC1_IN1) |
| Button         | PA2 (EXTI interrupt) |
| USB D+ / D-    | USB FS pins |

---

## Code Highlights

### HID Mouse Structure
```c
typedef struct
{
    uint8_t button;
    int8_t mouse_x;
    int8_t mouse_y;
    int8_t wheel;
} mouseHID;

mouseHID myMouse = {0,0,0,0};
````

### ADC with DMA

```c
uint16_t adc_val[2];
HAL_ADC_Start_DMA(&hadc1, (uint32_t*)adc_val, 2);
```

### Joystick Movement Mapping

```c
if((adc_val[0] - MID_POINT) > 20 || (adc_val[0] - MID_POINT) < -20)
    myMouse.mouse_x = (adc_val[0] - MID_POINT) / 400;
else 
    myMouse.mouse_x = 0;

if((adc_val[1] - MID_POINT) > 20 || (adc_val[1] - MID_POINT) < -20)
    myMouse.mouse_y = (adc_val[1] - MID_POINT) / 400;
else 
    myMouse.mouse_y = 0;
```

### Button Interrupt

```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    if(GPIO_Pin == GPIO_PIN_2)
    {
        btn_flag = 1;
    }
}
```

### Sending HID Report

```c
USBD_HID_SendReport(&hUsbDeviceFS, (uint8_t*)&myMouse, sizeof(myMouse));
HAL_Delay(20);
```

---

## How It Works

1. **Joystick Movement** → Changes in ADC values are mapped to `mouse_x` and `mouse_y`.
2. **DMA in Circular Mode** → Continuously updates ADC values without CPU load.
3. **Button Press (PA2)** → Sets `button` flag, sends HID report for a click, then resets.
4. **USB HID Report** → Sends joystick movements and button state to PC as a mouse input.

---

## Usage

1. Connect the STM32 board to the PC via USB.
2. Move the joystick → The PC cursor moves accordingly.
3. Press the button → A left-click is simulated.
