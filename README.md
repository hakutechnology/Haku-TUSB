# TinyUSB Implementation:

## Files Required:
Configure and place these files in App/USB.
Examples of these files are available via existing projects and TinyUSB

### tusb_config.h
Defines various required configs for TinyUSB.
Important configurations to consider for specific setup include: 
  BOARD_TUD_RHPORT - sets RHPORT (default 0)
  CFG_TUSB_RHPORT0_MODE - sets usb mode; device or host
Example for CDC configuration    


    #define USB_VID   0xCafe
    #define USB_BCD   0x0200
    
    //--------------------------------------------------------------------+
    // Device Descriptors
    //--------------------------------------------------------------------+
    tusb_desc_device_t const desc_device =
            {
                    .bLength            = sizeof(tusb_desc_device_t),
                    .bDescriptorType    = TUSB_DESC_DEVICE,
                    .bcdUSB             = USB_BCD,
    
                    // Use Interface Association Descriptor (IAD) for CDC
                    // As required by USB Specs IAD's subclass must be common class (2) and protocol must be IAD (1)
                    .bDeviceClass       = TUSB_CLASS_CDC,
                    .bDeviceSubClass    = MISC_SUBCLASS_COMMON,
                    .bDeviceProtocol    = MISC_PROTOCOL_IAD,
    
                    .bMaxPacketSize0    = CFG_TUD_ENDPOINT0_SIZE,
    
                    .idVendor           = USB_VID,
                    .idProduct          = USB_PID,
                    .bcdDevice          = 0x0100,
    
                    .iManufacturer      = 0x01,
                    .iProduct           = 0x02,
                    .iSerialNumber      = 0x03,
    
                    .bNumConfigurations = 0x01
            };
      
  
  Class - Pick which functionalities are enabled using 1 for enabled and 0 for disabled
  
    CFG_TUD_CDC = 1 // Communications Device Class
    CFG_TUD_MSC = 0 // Mass Storage Device
    CFG_TUD_HID = 0 // Hardware Interface Device
    CFG_TUD_MIDI = 0 // Musical Instrument Digital Interface 

  Rest of the file should be relativly standard from the example

  ### usb_descriptors.c
Defines USB standards configurations
#### Important configs to set:
    
tusb_desc_device_t - Sets general settings for USB protocol including advertised IDs
desc_fs_configuration - Sets USB full speed specific configurations

    //General config description
    // Config number, interface count, string index, total length, attribute, power in mA
    TUD_CONFIG_DESCRIPTOR(1, ITF_NUM_TOTAL, 0, CONFIG_TOTAL_LEN, 0x00, 100),

    //Class sepcific config description, required for each class enabled in tusb_config.h
    // Interface number, string index, EP notification address and size, EP data address (out, in) and size.
    TUD_CDC_DESCRIPTOR(ITF_NUM_CDC, 4, EPNUM_CDC_NOTIF, 8, EPNUM_CDC_OUT, EPNUM_CDC_IN, 64),
    

## CMake:
  Iclude to enable TinyUSB in code:
  
    #TinyUSB Enable
    add_definition(${target} -DUSES_TINYUSB)

  Include compile definition CFG_TUSB_MCU as supported MCU, example here configuring for STM32U5
  
      target_compile_definitions(${target} PRIVATE
            CFG_TUSB_MCU=OPT_MCU_STM32U5
    )

  Link TinyUSB library

       # Link libraries
    target_link_libraries(${target}
            TinyUSB
    )

Include App/USB in target_include_directories

    # Add include paths for App
    target_include_directories(${target} PRIVATE
            App/USB # Needed to source tusb_config.h
    )

Include App/USB/usb_descriptors.c in target_sources

    target_sources(${target} PRIVATE
            App/USB/usb_descriptors.c   #needed for tinyUSB configuration
    )

Fetch TinyUSB from Haku repos

    fetch_haku_library(Haku-TinyUSB TinyUSB R2025.03.01)


## CubeMX:
  Enable USB_DRD_FS:
Pinout & Configuration -> Connectivity -> USB_DRD_FS -> Mode: Device_Only_FS / Host_Only_FS
MAKE SURE TO ALSO ENABLE "usb global interupt" in NVIC Settings
    
<img width="1266" alt="image" src="https://github.com/user-attachments/assets/427fe28e-39f8-444b-bf1f-ae5493324976" />



## ST Interrupts:
Enableing "usb global interrupt" in the CubeMX NVIC Settings will generate a USB_IRQHandler in interrupt file (stm32*_it.c).
Need to call tud_int_handler(RHPORT) in user code section.
The arg set to RHPORT defined in tusb_config.h (default 0)


    /**
      * @brief This function handles USB global interrupt.
      */
    void USB_IRQHandler(void)
    {
      /* USER CODE BEGIN USB_IRQn 0 */
      tud_int_handler(RHPORT);
      /* USER CODE END USB_IRQn 0 */
      HAL_PCD_IRQHandler(&hpcd_USB_DRD_FS);
      /* USER CODE BEGIN USB_IRQn 1 */
    
      /* USER CODE END USB_IRQn 1 */
    }

  
