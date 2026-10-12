# Boot Sequence

### According to datasheet, boot sequence is divided into two parts:
#### 1. Hardware-controlled boot sequence
#### 2. Processor-controlled boot sequence

### Hardware controlled boot sequence:

#### Hardware controller boot sequence is not user controlled. It happens automatically. User cannot make any changes to this process except add a watchdog to restart the power-on state machine. 

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

#### 
