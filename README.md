# Phase-Locked Loop using Open-Source Google-Skywater 130nm 
![image](https://user-images.githubusercontent.com/90971641/133891914-a342895d-2a64-4b2f-b9a4-cb4efac3ac32.png)
# Index
- Day 1: Basics of PLL and Lab Setup 
    1. Introduction to PLL
    2. Components of PLL
    3. Some Common Terms
    4. Tools and Development Flow
    5. PLL Specifications
    6. Pre-Layout Circuits 
    7. Setting up Magic and NGSPICE
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
### For stability of the loop: Thumb Rule - (a)Cx~=C/10 (b) Loop Filter B.W. ~= Highest Output Frequency/10  else the stability of PLL reduces
 ### Loop Filter B.W.=1/(1+RC1) where C1=CCx/(C+CX)                                             
                                            
![image](https://user-images.githubusercontent.com/90971641/133915452-2e502302-d5e1-435a-ab86-bfffa27cacb7.png)

## iv. Voltage-Controlled Oscillator (VCO)
It is the most common on-chip oscillator design. It is a ring oscillator which consusts of a series combination of odd number of inverters with a delay. It causes the output signal to flip after a delay. It operates at a fixed frequency. 
### Period = 2xdelay(inverter)x inverter_count
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
NGSPICE and MAGIc were pre-installed in the lab assisstance.   
**Cloning repo to download technology file for Google Skywater 130nm for NGSPICE**  
![image](https://user-images.githubusercontent.com/90971641/133918315-7d406b7f-56b5-4fbb-a855-b19d8e5afefe.png)


**Creating a library with required cells and parameters**  
![image](https://user-images.githubusercontent.com/90971641/133918396-c14d3ead-b870-4d12-a2a0-b300be812d1b.png)


**Downloading 130nm technology file for Magic**  
![image](https://user-images.githubusercontent.com/90971641/133918422-85dbbd35-13ec-4320-b0ac-71a541380a4f.png)
