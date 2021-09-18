# Phase-Locked Loop using Open-Source Google-Skywater 130nm 
![image](https://user-images.githubusercontent.com/90971641/133891914-a342895d-2a64-4b2f-b9a4-cb4efac3ac32.png)
# Index
- Day 1: Basics of PLL   
    1. Introduction to PLL
    2. Components of PLL
    - 
    
# Day 1: Basics of PLL
## 1. Introduction to PLL
Phase-Locked Loop (PLL) is a control system which compares the frequency and phase of the on-chip oscillator with the reference signal and modifies the voltage of the oscillator in order to match the frequency of the oscillator with reference signal. PLL can be used to generate a signal of frequency equal to or multiple of the frequency of the reference signal. PLL reduces noise in the signal genrated by the oscillator.      
Some of the applications of PLL are for clock synchronization, in demodulation circuits and data converters.   

### BASIC BLOCK DIAGRAM
![image](https://user-images.githubusercontent.com/90971641/133893361-c99f729b-f5e3-4c9a-8e32-14e9b578dd41.png)

## 2. Components of PLL  
![image](https://user-images.githubusercontent.com/90971641/133893707-3e4a4a03-50dd-4181-9f52-037e71aa4eb3.png)
### i. Phase Frequency Detector (PFD)
Phase frequency detector is used to compare the phase and frequency of the reference signal with the feedback signal and generate a digital output. The output is generated  depending on whether the feedback signal is lagging or leading w.r.t the reference signal in phase. It also detects the frequency difference between the signals using the same functionality.  
This functionality can be represented in a state machine format:
 ![image](https://user-images.githubusercontent.com/90971641/133895518-a2b4192e-d893-4304-b993-e486fa9c5a47.png)
**Explaination:**
Initially, both UP and DOWN signals are equal to 0. 
When falling edge of reference signal arrives, UP=1. Then, when falling edge of output signal arrives, UP = 0 and the system goes back to initial state. 
When rising edge of reference signal arrives, DOWN=1. Then, when rising edge o f output signal arrives, DOWN = 0 and the system goes back to initial state.

 **Implementation using flipflops:** When both flipflops are triggered at the negedge, both the outputs become 1 and the AND gate activates the CLR signal, making output 0.
![image](https://user-images.githubusercontent.com/90971641/133895747-ba1a2984-2459-447d-93f2-dd71e3c65ea6.png)
