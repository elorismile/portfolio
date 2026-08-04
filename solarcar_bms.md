# SolarCar Battery Management System
I'm the electrical lead for the SolarCar at VT team, where we are building a solar race car that is capable of racing for hundreds of miles without recharging. If you want to learn more about what we do, Virginia Tech published a [<ins>cool magazine article about us.</ins>](https://eng.vt.edu/magazine/stories/fall-2024/solar-powered-cars.html)

![EVG05086](https://github.com/user-attachments/assets/1576897c-a041-4c0e-a7fc-1910f41a6329)

# Background

Two days prior to leaving for the 2025 Formula Sun Grand Prix, we fried our Prohelion Gen 1 BMU’s high voltage sense. We immediately ordered a new one, but Prohelion ships from Australia and the package would not arrive until the end of the race. 

Miraculously, I managed to make a replacement BMU using an ADC borrowed from my teammate’s RC car and a few lucky finds from our electrical junk drawer. After spending 4 straight days awake, I managed to fulfil the scrutineering requirements and our team was finally able to race on the track. 

# Goal

Learning from what happened during FSGP 2025, I set out to design a proper custom BMS with the following goals:

- Repairable without buying a whole new module  
- Use generic components to maximize repairability  
- Cheap enough to allow budget for an entire spare BMS  
- Designed explicitly to pass the scrutineering process  
- Use larger component package sizes to increase robustness against physical wear/moisture/carbon dust and make in-field repair easier  
- As close to millivolt measurement accuracy as possible  
- More thermistors to meet regulatory requirements  
- Make the BMS operation independent from other car systems. 

Ultimately, the goal for the new BMS was to be as reliable as possible and get us through scrutineering. 

# CMU Design

There are 4 Cell Management Units (CMUs) in our car. They sit on top of the battery modules (7s-8s), and are responsible for measuring cell voltage, temperature, and balancing. 

I considered using off-the shelf cell monitor ICs like the LTC6811 or L9963E, but these options were not reliably in stock during development and I ultimately decided that a custom design would offer greater flexibility in the long run, and would support active balancing in the future. 

At the heart of the CMU is the analog to digital converter used to measure the cell voltage. I wanted 24 bit precision to help get me as close to \+/-1 mV accuracy as possible. I initially tested using ADS131, but settled on using the MCP356xR family due to better performance in tests and better pricepoint. 

For the measurement architecture, I initially went with 2 muxes to feed cell positive and negative taps to a difference amplifier, which would produce a single-ended signal for the ADC. I went with two MUX508s from TI paired with an INA146 difference amp, which could both handle up to 36V. On paper, the design should have worked very well, as the INA146 datasheet promised 0.025% gain error. However, once the CMU was assembled, the output of the INA146 wandered by 20mV on its own. 

By the time I realized that the difference amp/mux approach wasn’t going to work, it was already March. With little time left, I did some prototyping and decided to go with a custom switch matrix and flying ADC. 

Compared to mux/diff amp, flying switch matrix offers the following advantages:

- Generic parts (no available good alternatives for the hv difference amp or hv mux)  
- Cheaper cost (mux and difference amp were expensive parts)  
- \<1 ohm switch matrix resistance compared to 170 ohm mux resistance  
- No inaccuracy introduced by an additional amplifier stage  
- Uses component types already found in CMU BOM

The switch matrix works by having an optoisolator control a pair of source-to-source PMOS to block current in both directions. Each cell tap has one of these controlled switches, which either connects it or disconnects it from the differential measurement rail. The main electronics and ADC of the CMU are isolated both from the battery cells and the rest of the car. When a cell tap is switched on, the CMU’s ground level “flys” to the approximate level of the cell tap thanks to the ESD diodes in the ADC. 

Ordinarily, there would need to be a separate positive and negative rail for the cell measurement, and each cell tap would have two switches, one to connect to the positive rail and one to connect to the negative rail. This flying design takes advantage of the differential mode on the ADC to reduce component count by cutting the number of switches needed in half. Instead of each tap having a positive and negative rail switch, odd number voltage taps are connected to rail A and even numbered taps are connected to rail B. This means that as the CMU measures different cells, the ADC will see positive differential input, then negative, then positive. 

# BMU Design

The Battery Management Unit (BMU) is responsible for contactor control, precharge, current measurement, and managing all of the CMUs. 

The BMU is split into 3 modules: BMU\_Main, BMU\_HVSense, and BMU\_Power. 

## BMU\_Power

Has high-side NMOS for contactor control. It also has the mechanism to switch the car’s 12V rail from the supplemental battery to the HV-\>12V converter. 

## HV\_Sense

Contains the ADC used for pack current measurement, and contains an MCP3562R (4 channel version) used for voltage measurement during precharge. When laying out the PCB, I had to consider voltage creep, so the voltage dividers were spread out over several series resistors. The most important part is that the current shunt and pack voltage measurement are entirely isolated from each other, so the failure that happened last year is impossible. 

## BMU\_Main

Contains the ESP32S3 that runs the show, as well as an SD card for storing SOC data. This one is the least likely to fry, but in the event that it does, the SD card can be swapped to the new module to retain the SOC data. 

## Tangents about BMU Design

A note about the board-board interconnects: The simplest choice is 2.54mm dupont headers like Arduino and Raspberry Pi have classically used. I’m not a huge fan of these, because alignment isn’t guaranteed. We’ve had issues with these accidentally getting plugged in 1 over and frying things. The only thing worse than 2.54mm dupont is 1mm dupont, which is a story for another time. I found the perfect connector by accident. I was using the Sullins SBH11 series for the CMU interconnect ribbon cable, and I discovered that there is a female board-mounted version, the SFH11 series. This turned out to be perfect for my application because it had keyed alignment, cost effective compared to other board-board connectors, has robust contacts, and uses 2.54mm spacing and could be swapped out with dupont headers if it goes out of stock. 

A crucial change compared to the Prohelion system is my decision to make the switch panel directly wired to the BMU. The Prohelion system relied on the ignition switch signal to be transmitted over CAN multiple times per second. We ran into issues during FSGP2025 where our CAN bus would go down, and our car would shut off on the track. I had to reboot the car over 20 times just to get it back to the pit. This year, I decided to go with a direct-wired switch panel. After adding some strong EMI filtering to the switch wires, it performed absolutely flawlessly during FSGP2026

# Results

The BMS performed exceptionally well during scrutineering tests. We passed BPS scrutineering first try, which was a big step up from last year. Additionally, it performed extremely reliably with no unexpected faults or fried components out on track. 

## What worked well:

- Pre-added test points helped scrutineering speed  
- \+/- 10mV cell sense accuracy lets us push all the way to the 4.2V-2.5V cell limits  
- Good EMI resistance, measurements remain accurate even when the motor controller is under heavy load  
- Direct-connect switch panel worked reliably  
- RS485 bus connecting CMUs to BMU proved sufficiently reliable  
- Modular BMU design was easy to service  
- No fried components\!

## What could be improved: 

- During testing, I encountered the problem of the measured cell voltage being different by about 40mV upon power up. It turned out that the internal voltage reference on the MCP3561R wasn’t up to the job, and I overnight shipped some external voltage references rated for 0.05% and they performed much better.   
    
- Thermistors accuracy could be better. Mystery thermistors from Amazon proved to be good enough for this year’s race (+/-5C), but for the next version I am planning on improving the quality of the thermistor setup to let us push closer to thermal limits. 

- The MCP3561’s differential input is more susceptible to common-mode noise than I was hoping for. I observed that the common mode of the differential inputs would continue to settle for around 20ms after the switch matrix made its switch. I used as long of a settling delay as I could, but measuring before it was completely settled resulted in drift in the reading. In the next version, I plan to pull one of the differential inputs to ground and then run the ADC in single-ended mode. This method adds complexity and more risk of frying the ADC, but will hopefully let me push closer to the \+/-1mV that I am striving for. 


[return to main page](index.md)
