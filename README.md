# Phase-Locked Loop using Open-Source Google-Skywater 130nm 
![image](https://user-images.githubusercontent.com/90971641/133891914-a342895d-2a64-4b2f-b9a4-cb4efac3ac32.png)
# Index
- [Day 1: Basics of PLL and Lab Setup](https://github.com/ikhindria/PLLWorkshop/blob/main/README.md#day-1-basics-of-pll)  
    [1. Introduction to PLL](1-introduction-to-pll)  
    [2. Components of PLL](2-components-of-pll)  
    [3. Some Common Terms](3-some-common-terms)  
    [4. Tools and Development Flow](4-tools-and-development-flow)  
    [5. PLL Specifications](5-pll-specifications)  
    [6. Pre-Layout Circuits](6-pre-layout-circuits)  
    [7. Setting up Magic and NGSPICE](7-setting-up-magic-and-ngspice)  
 - [Day 2: Lab](#day-2-lab)  
    [1. Circuit Design and Pre-Layout Simulations](1-circuit-design-and-pre-layout-simulations)  
    [2. Layouts](2-layouts)  
    [3. Extracting Parasitics](3-extracting-parasitics)  
    [4. Post-Layout Simulations](4-post-layout-simulations)  
    [5. Tapeout Theory](5-tapeout-theory)  
  - [Acknowledgements](acknowledgements)
# Day 1: Basics of PLL
## 1. Introduction to PLL
Phase-Locked Loop (PLL) is a control system which compares the frequency and phase of the on-chip oscillator with the reference signal and modifies the voltage of the oscillator in order to match the frequency of the oscillator with reference signal. PLL can be used to generate a signal of frequency equal to or multiple of the frequency of the reference signal. PLL reduces noise in the signal genrated by the oscillator.      
Some of the applications of PLL are for clock synchronization, in demodulation circuits and data converters.   

### BASIC BLOCK DIAGRAM
![image](https://user-images.githubusercontent.com/90971641/133893361-c99f729b-f5e3-4c9a-8e32-14e9b578dd41.png)

## 2. Components of PLL  
![image](https://user-images.githubusercontent.com/90971641/133893707-3e4a4a03-50dd-4181-9f52-037e71aa4eb3.png)
## i. Phase Frequency Detector (PFD)
Phase frequency detector is used to compare the phase and frequency of the reference signal with the feedback signal and generate a digital output. The output is generated  depending on whether the feedback signal is lagging or leading w.r.t the reference signal in phase. It also detects the frequency difference between the signals using the same functionality.  
This functionality can be represented in a state machine format:
 ![image](https://user-images.githubusercontent.com/90971641/133895518-a2b4192e-d893-4304-b993-e486fa9c5a47.png)
**Explaination:**
Initially, both UP and DOWN signals are equal to 0. 
When falling edge of reference signal arrives, UP=1. Then, when falling edge of output signal arrives, UP = 0 and the system goes back to initial state. 
When rising edge of reference signal arrives, DOWN=1. Then, when rising edge o f output signal arrives, DOWN = 0 and the system goes back to initial state.

 **Implementation using flipflops:** When both flipflops are triggered at the negedge, both the outputs become 1 and the AND gate activates the CLR signal, making output 0.

![image](https://user-images.githubusercontent.com/90971641/133895747-ba1a2984-2459-447d-93f2-dd71e3c65ea6.png)

**Dead Zone:** Range where difference in phase and frequency cannot be detected as the edges of REF and OUT signal are very close and the UP or DOWN signals are not high for enough time. More precise the PFD, better the stability of the phase-locked loop.
## ii. Charge Pump(CP)
A charge pump converts a digital signal which denotes the phase and frequency difference into an analog signal that can be used to control the oscillator. 
This can be done using a current steering circuit. It directs the current from VDD to output or from output to ground, depending on the given UP and Down signals. 

![image](https://user-images.githubusercontent.com/90971641/133915347-f432a0e2-b512-4196-b520-1c7cd9183595.png)

The charging and discharging of capacitor depends on UP and DOWN signals. When average UP signal active time> average DOWN signal, the output voltage rises and vice versa. 
This output will control the VCO. Increasing the output voltage speeds up the VCO whereas reduction slows it.   
 **Implementation using MOSFETs:** Mncsr and Mpcsr are current sources. Even when the current sources are off, there is some leakage current which keeps charging the capacitor even when there is no UP or DOWN signal.
  ![image](https://user-images.githubusercontent.com/90971641/133915215-5c84d431-6f53-4977-9bd3-44b15f80ff9f.png)
  
  ## iii. Loop Filter
  Capacitor of charge pump smoothens the output however it is not that smooth. There are still fluctuations due to UP and DOWN signals. The output capacitor can be replaced by a LPF(Low Pass Filter) so that any fluctuations of high frequency can be smoothened out. Adding LPF increses the stability of the PLL by adding a zero to the frequency domain.   
**For stability of the loop: Thumb Rule - (a)Cx~=C/10 (b) Loop Filter B.W. ~= Highest Output Frequency/10  else the stability of PLL reduces    
Loop Filter B.W.=1/(1+RC1) where C1=CCx/(C+CX)**                                             
                                            
![image](https://user-images.githubusercontent.com/90971641/133915452-2e502302-d5e1-435a-ab86-bfffa27cacb7.png)

## iv. Voltage-Controlled Oscillator (VCO)
It is the most common on-chip oscillator design. It is a ring oscillator which consusts of a series combination of odd number of inverters with a delay. It causes the output signal to flip after a delay. It operates at a fixed frequency. 
### Period = 2 x delay(inverter) x inverter_count
To vary frequency, delay must be vaired and consequently, current supplied must be varied. Delay is the time taken to charge the output. 
We can control the frequency of the oscillator by 'starving' the oscillator of current. 

![image](https://user-images.githubusercontent.com/90971641/133915812-5e28b4bd-edda-4933-a800-b27efec26336.png)
Two current sources at top and bottom used as supplies. This enables control of frequency by input control voltage. We need to ensure that range of PLL is within the range of frequencies the VCO can generate.  

## v. Frequency Divider (FD)

freq/2 consists of a D flipflop and an inverter. For freq/8 we can stack 3 such D flipflops and inverters. 
![image](https://user-images.githubusercontent.com/90971641/133915904-aede1cfb-1c67-4b3e-87d4-e22222aeb410.png)

## 3. Some Common Terms
### i. Lock Range :
It is the range of frequencies for which PLL is able to follow input frequency variations once locked.  
### ii. Capture Range :
It is the range of frequencies for which PLL is able to attain a lock-in when starting from an unlocked condition. Is is usually smaller than the lock range and will depend on loop filter bandwidth. 
### iii. Settling Time :
The time in which the PLL is able to attain a lock state from an unlocked state. It depends on how quickly the charge pump rises to its stable value. 

##  4. Tools and Development Flow
Tools being used : Ngspice and Magic

### Development Flow
1. SPICE-level Circuit development
2. Pre-Layout Simulation
3. Layout Development
4. Parasitics Extraction
5. Post-Layout Simulation

##  5. PLL Specifications
- Corner = 'TT' (nominal case where doping process led to just right MOS devices)
- Supply Voltage = 1.8V
- Room temperature
- VCO mode(control voltage given directly to VCO using a VCOin pin) and PLL mode
- Input Fmin=5Mhz; Fmax = 12.5Mhz
- Multiplier = 8x
- Jitter(RMS)< ~20ns
- Duty Cycle = 50%

### Approximate pin location specs: 
![image](https://user-images.githubusercontent.com/90971641/133916550-59e1f2ca-7cc0-4173-a148-2f7415034d45.png)

##  6. Pre-Layout Circuits
### i. FD
![image](https://user-images.githubusercontent.com/90971641/133916592-d6c1f83f-83a2-4069-8599-7ee1ced9b02a.png)

### ii. PFD
![image](https://user-images.githubusercontent.com/90971641/133916623-3ac38f19-f84c-42e3-86cf-c93787b1aeac.png)

### ii. CP
![image](https://user-images.githubusercontent.com/90971641/133916684-a286b199-061e-42b4-9cfb-687a48f771da.png)

### iii. VCO

![image](https://user-images.githubusercontent.com/90971641/133916703-4dfda7ae-9dfe-4b83-a606-1a8f18e39755.png)

##  7. Setting Up Magic and NGSPICE
NGSPICE and MAGIC were pre-installed in the lab assistance.   
**Cloning repo to download technology file for Google Skywater 130nm for NGSPICE**  
![image](https://user-images.githubusercontent.com/90971641/133918315-7d406b7f-56b5-4fbb-a855-b19d8e5afefe.png)


**Creating a library with required cells and parameters**  
![image](https://user-images.githubusercontent.com/90971641/133918396-c14d3ead-b870-4d12-a2a0-b300be812d1b.png)


**Downloading 130nm technology file for Magic**  
![image](https://user-images.githubusercontent.com/90971641/133918422-85dbbd35-13ec-4320-b0ac-71a541380a4f.png)

# Day 2: Lab
## 1. Circuit Design and Pre-Layout Simulations
### i. FD
- x denotes sub circuit and in sky 130nm library mos models are deoted as sub circuits  
m - mos  
c-capacitance  
r-resistance  
v-voltage source   
and so on  
- 3 2 1 1 drain gate source body respectively names of net  
- width w=360 x 2 =720n; 360 is the min width in skywater for nfet. Double taken for transistor sizing purpose.  
- nets 1=vdd, 0=gnd, 2= gate (input) 3 output    
**Command for simulation:**    
![image](https://user-images.githubusercontent.com/90971641/133921410-5411689f-0de1-4c1d-a27a-a800bfdcb181.png)

![image](https://user-images.githubusercontent.com/90971641/133921421-cda4000c-acc8-4207-8c6d-0f78d0f3fb0f.png)

### ii. CP
**When no voltage given, to check leakage current, Leakage Current = 40uV**  
![image](https://user-images.githubusercontent.com/90971641/133921469-52b018ff-6977-4df0-b12e-2442223a9111.png)

**When PULSE given**  
![image](https://user-images.githubusercontent.com/90971641/133921490-da202b97-b8f6-45ef-aade-938b05162a92.png)

### iii. VCO  
![image](https://user-images.githubusercontent.com/90971641/133921584-6b1bb453-cd55-4f02-a10d-9c8f4584d4b6.png)

### iv. PFD  
![image](https://user-images.githubusercontent.com/90971641/133921605-adb2d215-a2a5-4297-a94e-ee4946ff71f5.png)

### v. PLL
Combining all the components in one file, including loop filter and instantiating it.  
![image](https://user-images.githubusercontent.com/90971641/133921993-158a4256-343e-4684-acd4-fce44169a260.png)

**Close up**
![image](https://user-images.githubusercontent.com/90971641/133922905-db782cc5-65b7-46dc-9738-24e5c0f225b5.png)
## 2. Layouts
**Commands:**  
![image](https://user-images.githubusercontent.com/90971641/133924475-67ab98fe-cb5c-4e54-8d43-17bf590f2bac.png)

### i. FD
![image](https://user-images.githubusercontent.com/90971641/133924484-a2b14d2f-34f7-4461-9bea-e7c11504cb50.png)

### ii. PFD
![image](https://user-images.githubusercontent.com/90971641/133924497-51146526-d17b-48ff-b692-6f2257975db3.png)

### iii. CP
![image](https://user-images.githubusercontent.com/90971641/133924512-82a8ddcf-1c24-4f20-acd3-21ace1c23364.png)

### iv. VCO
![image](https://user-images.githubusercontent.com/90971641/133924528-e3d38dbc-49bb-4a7d-b965-2621b6c1b1ac.png)

### v. MUX
MUX used to select between charge pump and external source to control the VCO. This enables the VCO mode.  
![image](https://user-images.githubusercontent.com/90971641/133924545-70f68757-7aed-4981-b7ab-1d9f63f4f065.png)

### vi. PLL 
![image](https://user-images.githubusercontent.com/90971641/133924562-e2f24ac3-9963-43d6-a315-82aa2c8aa60d.png)  

**To generate GDS file after post-layout simulations, click on File->Write GDS.** 
![image](https://user-images.githubusercontent.com/90971641/133927503-a448c753-0ac8-4c9b-ba15-ce3f26d9c075.png)

## 3. Extracting Parasitics
![image](https://user-images.githubusercontent.com/90971641/133927158-43608f0b-285b-4735-9ead-0427b76ade8b.png)

## 4. Post-Layout Simulations
### i. PFD
![image](https://user-images.githubusercontent.com/90971641/133927164-898b540c-84df-415f-b0c8-126ead07162d.png)
![image](https://user-images.githubusercontent.com/90971641/133927167-f6be659c-c26a-4ac8-93bb-5463d3cc6f0c.png)

### ii. VCO
![image](https://user-images.githubusercontent.com/90971641/133928666-728898f2-2b8d-40ac-aa4a-5fc1dd0b0c0c.png)

### iii. FD
![image](https://user-images.githubusercontent.com/90971641/133928737-d18a4d38-4ca1-4524-aa47-6a01aae86103.png)
### iv. CP
**Charging**  
![image](https://user-images.githubusercontent.com/90971641/133928783-90ad860e-e7d0-4e35-9c1f-8444f547a77c.png)
**Discharging**  
![image](https://user-images.githubusercontent.com/90971641/133928814-8a7d9489-fa05-4840-bc85-be9b8b5136ef.png)
### v. PLL  
![image](https://user-images.githubusercontent.com/90971641/133928857-f1041fb7-ab41-4e4a-aaaf-eefde206d41b.png)

## 5. Tapeout Theory
Tapeout means sending the final design to the Fab. 
It will contain
- I/O pads
- Peripherals
- Memory
- Testing Mechanisms
- Others  
**Caravel SoC by efabless**  
![image](https://user-images.githubusercontent.com/90971641/133928966-0249dad6-2de9-4f62-8f1c-8ffda8bf2eb6.png)

# Acknowledgements
1)Mr. Kunal Ghosh From VSD  
2)[Ms. Lakshmi Sathi](https://github.com/lakshmi-sathi/avsdpll_1v8)

