# Boot Sequence

### According to datasheet, boot sequence is divided into two parts:
#### 1. Hardware-controlled boot sequence
#### 2. Processor-controlled boot sequence

### Hardware controlled boot sequence:

#### Hardware controller boot sequence is not user controlled. It happens automatically. User cannot make any changes to this process except add a watchdog to restart the power-on state machine. 
![](/home/andresiniesta/Pictures/Screenshots/Screenshot from 2026-10-12 17-27-45.png)
1. Power is applied to the chip - This means RUN pin is high, chip is in RESET state as long as RUN pin is LOW
2. The on-chip voltage regulator turns ON. The system waits until the digital supply (DVDD) is stable
3. The Power-On State Machine starts. The required hardware blocks and peripherals need to be turned on in a particular order since, one's presence would be required by the other. (Refer to _**Figure 27**_ in datasheet)
   1. The ring oscillator is started
   2. The crystal oscillator is reset (not started yet, happens later in _runtime_init_)
   3. _clk_ref_ and _clk_sys_ are clock generators. They are bought out of reset. _clk_ref_ runs using ring oscillator with no divider and _clk_sys_ runs from _clk_ref_
   4. Reset controller resets all non-boot peripherals
   5. System performs a boot check on the flash and sanity of the program in it
   6. The system brings bus fabric, ROM/SRAM and XIP (execution in place) out of reset
   7. The system brings processors out of reset. They inturn begin executing bootcode from ROM.

### Processor controlled boot sequence:

#### Processor controlled boot sequence is also "baked in" and it is not something that is user programmed. It runs from the ROM. Refer to _**Figure 15**_ in the datasheet for the flow chart.
![](/home/andresiniesta/Pictures/Screenshots/Screenshot from 2026-10-12 17-25-20.png)
The flow chart is described below:
1. System reads the CPUID from the SIO_CPUID register in the SIO block. If it reads 0 (means core 0), the process continues. Else, it sleeps until an entry point is provided.
2. System checks if PoR rescue flag is set. If yes, the execution halts here and waits for aN instruction from a debugger
3. System checks for the watchdog scratch registers for magic numbers to see if it needs to load some code from SRAM. If it finds those magic numbers, it loads the code from that vector address. Otherwise, it jumps to main() in bootrom_main.c
4. If BOOTSEL pin is not being held LOW, the system will attempt a flash boot. For this purpose, it initializes the SSI channel, initializes the GPIOs required and communicates with the flash. It loads the first 256 bytes of the flash and checks for the checksum. If checksum passes, the system enters boot2 (flash second stage). Else, the crystal oscillator is started, PLLs are initialized, USB is clocked at 48MHz and the system enters USB device mode bootcode.

### Boot stage 2


