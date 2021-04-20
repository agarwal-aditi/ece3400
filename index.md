# Aditi Agarwal ECE 3400 SP 21

This is Aditi Agarwal's (aa2224) Wiki Page for ECE 3400 SP 21. 

# Table of Contents
- [Lab 1](#lab1)
- [Lab 2](#lab2)
- [Lab 3](#lab3)

## Lab1

In Lab 1, I learned how to program the Arduino, how to assemble a circuit with photoresistors, and how to view the input of photoresistors through using Arduino pins. 

### Blinking the LED

As an introduction to programming an Arduino, the first task of the lab was to blink an LED. After installing the Arduino IDE, I used the file given to us, `blink_LED.ino`. This code helped me learn how programming a microcontroller works, in which there is a setup sequence, and then a loop that continuosly runs on the Arduino everytime it is turned on until a new program gets loaded onto the Arduino or you press the RESET button. In the setup part of the code, the `LEDBUILTIN` pin, which is a "pin" that controls the LED that is already on the Arduino, is declared as an output so that the compiler can tell the Arduino that it is writing values to it to output onto the connected I/O device, which in this case is the built-in LED. In the loop part of the code, we first write a `HIGH` signal to the LED, which is a digital value that is high enough power to turn on the LED, then there is a delay, then we write `LOW` which turns the LED off, and then there is another delay. The delay between the writes to the LEDs is required in order to see any visible change since the clock speed of the Arduino is faster than we can see. After understanding how the code works, I programmed it to the Arduino, and saw that how I changed the delay time affected how fast the LED was blinking. 

### Building the Photosensor Circuits

After understanding how to program the Arduino, we got started on the first part of developing a Light Following robot. To get the Light Following Robot to follow light, I first needed to connect the photoresistors to the Arduino to sense light intensity. The CdS Photosensors are photoresistors in which as more light is detected (indcident light) at the sensor, the resistance value decreases so that the output voltage increases. This is because the photoresistor is a voltage divider circuit with the resistor that has incident light is on one side and a 10 kilo-ohms pull-down resistor is on the other side. The output voltage is connected to an analog input of the Arduino Nano so that its value can be read (from 0 to 1023) by the Arduino as a digital value. 

 Since it reads a value upto 1023, it was important to first calibrate the photoresistor to the regular lighting surroundings so that it will be able to detect a flashlight approximiatley 30 cm away. To do this, I first created a simple circuit with just one photosensor connected. 
 
![one photosensor circuit](single_photoresistor_circuit.png)


After creating this circuit, I used the `CdS_ReadA0.ino` code to read the value to the Serial Monitor. In the code, first in set up I set the baud rate to 9600 and then match that on the serial monitor so that I can read the outputs. Then in the loop, I read the pinvalue using an `analogRead` to pin A0, which is which pin the photoresistor output is. Below is what got printed out on the serial monitor without a flashlight near it, so it is the value based on just surrounding light.

![img2](one_photoresistor_serial.png)


After callibrating with one photosensor, I created a circuit with two photosensors, one pointing to the left and one to the right of the robot so that it will turn to the right or left based on where the flashlight is. I used two analog pins, A0 and A1 for the output of the photoresistors. The full circuit below. 

![full circuit](full_circuit.png)

To detect which side the flashlight is, I used a Normalized Measurement for the left and the right. This is needed because if the flashlight was on the right, for example, the left would still detect it. The normalized measurement for the left is the sensor reading on the left divided by the sum of both sensor readings and the normalized measurement for the right is the sensor reading on the right divided by the sum of both sensors. I wrote the code `CdS_ReadA0A1.ino` so that it prints the normalized measurement values on both the left and right side. This is the first step to eventually making the robot follow light.

Serial Output when flashlight is on the right:

![img4](photoresistor_right.png)


Serial Output when flashlight is on the left:

![img5](photoresistor_left.png)


In Lab 2, I will connect the motors to the circuit and write code so that the robot moves based on the light.


## Lab2

In this lab, I integrated the work from Lab 1 where I callibrated and created the circuits for the photoresistors with the actual motors and wheels to fully create a light following robot. 

### Materials
<img width="611" alt="Screen Shot 2021-03-29 at 5 34 50 PM" src="https://user-images.githubusercontent.com/45053255/112903091-11614600-90b5-11eb-9e58-9acc08dc3e81.png">


### Analog-to-Digital Conversion on Arduino

In order to read voltage values, the Arduino is able to convert an Analog input into a digital reading from 0 to 1023. 

In this lab, we had to examine the timing of the ADC process as we use it to read the photoresistor values for light following. The CLK_ADC value generates a clock signal as described by the following table from the ATMega4809 Datasheet. 

<img width="818" alt="Screen Shot 2021-03-29 at 5 09 50 PM" src="https://user-images.githubusercontent.com/45053255/112900484-93e80680-90b1-11eb-86ad-5e7c60726b41.png">


Using the bitRead() function and from reading the ATMega Datasheet for the correct bits to access, I was able to access the prescalar and PDIV values from the ADC0 CTRLC and CLKCTRL MCLKCTRLB registers respectively. The CLK_ADC values is generated from the CLK_PER value which is the CPU clock divided down by the prescaler value that is accessed in the CTRLC register in ADC0. 

![Uploading Screen Shot 2021-03-29 at 5.10.31 PM.png…]()
<img width="925" alt="Screen Shot 2021-03-29 at 5 10 19 PM" src="https://user-images.githubusercontent.com/45053255/112900543-a5c9a980-90b1-11eb-9078-3747716fa2a3.png">


The prescalar value of 6 means that the prescalar is 128. There is also the PDIV value which is either enabled or un-enabled for use to futher divide the CLK_PER value by. 

<img width="927" alt="Screen Shot 2021-03-29 at 5 10 42 PM" src="https://user-images.githubusercontent.com/45053255/112900586-b2e69880-90b1-11eb-8a7b-ee9b9755a173.png">


Since the PDIV was not enabled (PEN = 0), it was not factored into the CLK_ADC value.

The CLK_ADC is hence 16 MHz / 128 = 125 kHz

### H-Bridge

To run the robot's motors, we used an H-Bridge. We were given a chip for the H-Bridge (L293D). To power and control the motors, we used the following pinout from Lecture 7.


<img width="694" alt="Screen Shot 2021-03-29 at 4 47 34 PM" src="https://user-images.githubusercontent.com/45053255/112898132-782f3100-90ae-11eb-8d56-598d2129cb8e.png">

The internal logic for the L293D chip to work requires 5V from the Arduino and the motors use the 4.5V AA Battery pack. I connected Arduino output pins to the enables for the 2 motors to control the speed and then digital connections to the motor. 

<img width="618" alt="Screen Shot 2021-03-29 at 5 11 14 PM" src="https://user-images.githubusercontent.com/45053255/112900649-c5f96880-90b1-11eb-881e-1e840a76af20.png">

The chip at the bottom of the image (back of the robot) is the L293D chip.

To send a digital signal that can control power, the Arduino uses Pulse Width Modulation (PWM). PWM works by sending either a high or low signal, but in different time intervals. The longer the high signal is in each cycle, the higher the voltage is. 

<img width="414" alt="Screen Shot 2021-03-29 at 5 07 00 PM" src="https://user-images.githubusercontent.com/45053255/112900226-2dfb7f00-90b1-11eb-8916-f4a08e43d875.png">

I used 2 Arduino digital pins with PWM for the enables for each motor to control the speed. The other outputs for the direction did not need to be connected to PWM pins because they were either just high or low. To show wheel actuation, I first just made the robot's wheels move forward together, move backward together, turn in opposite direction to each other, and then come to a full stop. Each of these are for one second until the stop. To get the timing accurate, I used a millisecond timer and compared the time in the loop to 0 to switch to the different wheel directions based on time. 

video link: https://drive.google.com/file/d/1U6MJ6ScXGQpIi-ZrKf22N5cd0a4_htNr/view?usp=sharing

<figure class="video_container">
  <iframe src="https://drive.google.com/file/d/1U6MJ6ScXGQpIi-ZrKf22N5cd0a4_htNr/preview" width="640" height="480"></iframe>
</figure>

### Calibrating the Motors

The wheels are impossible to attach identically, hence to move the robot in a straight line, I had to find the different PWM speed values to write to the motors to go straight. I found that my left wheel moved faster than my right wheel so I adjusted for both fast and slow speeds.

Fast in straight line video: https://drive.google.com/file/d/1BnMHmPCJJ87rW69YuH_DytN6lfREscw7/view?usp=sharing

<figure class="video_container">
  <iframe src="https://drive.google.com/file/d/1BnMHmPCJJ87rW69YuH_DytN6lfREscw7/preview" width="640" height="480"></iframe>
</figure>

Slow in a straight line video: https://drive.google.com/file/d/1GYDUXnUgZSVViRshvOcERzb2nhsQlAwe/view?usp=sharing

<figure class="video_container">
  <iframe src="https://drive.google.com/file/d/1GYDUXnUgZSVViRshvOcERzb2nhsQlAwe/preview" width="640" height="480"></iframe>
</figure>

### Light Following using Photosensors

After calibrating the motors and understanding how to move the wheels, I combined the code from lab 1 normalized measurements identifying if the light is right or left with the code for moving the wheels. To turn the robot, I set the wheel in the direction the robot is turning to a slow speed, and the opposite wheel a fast speed. 

For light following, the conditions are that if there is no extra light source (flashlight), the robot spins in circles and the LED blinks repeatedly with 500 ms on and then off, if there is a light on one side then the robot turns that direction, and if the flashlight is not on one side the robot goes in a straight line. 

To make the code more organized, I made multiple functions. One function to move forward, one to move left, one to move right, and one for the blinking LED. This helped me debug the code a lot easier. In the main loop, I check if the left normalized value is between 0.4 and 0.6 and if it is, then it checks if there is extra light by checking if the actual photosensor value is greater than 100. If it is greater than 100 it goes straight and if not then it spins in circles with the blinking LED. If the left normalized value is not between 0.4 and 0.6 then it sees with normalized value is higher and turns in that direction. 

Final Video: https://drive.google.com/file/d/1JJEkW6YtUpRud0T0AVR1O1OzvPztSUCq/view?usp=sharing

<figure class="video_container">
  <iframe src="https://drive.google.com/file/d/1JJEkW6YtUpRud0T0AVR1O1OzvPztSUCq/preview" width="640" height="480"></iframe>
</figure>


## Lab3

In this lab we learned about Filtering and FFT by wiring circuits for a microphone and doing FFT analysis on MATLAB and then the Arduino. I implemented and tested passive and active filters. For the final circuit, I implemented a bandpass filter with the microphone which will be used in the next lab.

### Materials
<img width="636" alt="Screen Shot 2021-04-20 at 4 59 01 PM" src="https://user-images.githubusercontent.com/45053255/115463350-b6c19280-a1f9-11eb-804f-711e73fe2602.png">

### Microphone Circuit without Amplification

The first iteration of the microphone circuit was without any amplification or filtering and was a simple circuit, as shown in the picture below:
<img width="639" alt="Screen Shot 2021-04-20 at 6 11 12 PM" src="https://user-images.githubusercontent.com/45053255/115470109-cf36aa80-a203-11eb-8e62-ed7a1517c44b.png">

<img width="412" alt="Screen Shot 2021-04-20 at 7 20 17 PM" src="https://user-images.githubusercontent.com/45053255/115475369-71a75b80-a20d-11eb-9ceb-47631f3c872f.png">

After implementing this circuit, I wrote the Arduino code to recieve it. Since the data needed to be read quickly, I could not just simply use the analogRead() function. I had to manually code the ADC in Free Running mode to obtain the values. Then after getting the ADC value, which is from 0 to 1023, I had to convert it to a signed 16 bit integer value so I could graph the spectrum. I converted it by subtracting 512 (the 0 value from 0 to 1023) and then shifting 6 bits to accomodate for the subtraction. 

Frequency vs Amplitude for 500 Hz on MATLAB
<img width="533" alt="Screen Shot 2021-04-20 at 7 26 21 PM" src="https://user-images.githubusercontent.com/45053255/115475734-4b35f000-a20e-11eb-80a4-ecdce731cd07.png">



### LT Spice Simulations vs Actual Results

To test if the passive and active filters I implement are correct, I simulated them using LTSpice.

Low Pass Filter Schematic
<img width="884" alt="Screen Shot 2021-04-20 at 5 01 10 PM" src="https://user-images.githubusercontent.com/45053255/115463608-030cd280-a1fa-11eb-819d-4a78f2a035a7.png">

Simulated Low Pass Cutoff Frequency Zoomed-In
<img width="886" alt="Screen Shot 2021-04-20 at 5 02 39 PM" src="https://user-images.githubusercontent.com/45053255/115463786-38192500-a1fa-11eb-93fa-8c2e188887a6.png">

High Pass Filter Schematic
<img width="1123" alt="Screen Shot 2021-04-20 at 5 01 46 PM" src="https://user-images.githubusercontent.com/45053255/115463675-1881fc80-a1fa-11eb-962e-687bd7b69308.png">

Simulated High Pass Cutoff Frequency Zoomed-In
<img width="885" alt="Screen Shot 2021-04-20 at 5 03 58 PM" src="https://user-images.githubusercontent.com/45053255/115463967-672f9680-a1fa-11eb-8ed1-769f5c862c51.png">

The Low Pass and High Pass filters were not accurate on the actual board. The professor explained that the inaccurate results are likely a result of faulty capacitors.

Bandpass Filter Schematic
<img width="861" alt="Screen Shot 2021-04-20 at 5 38 55 PM" src="https://user-images.githubusercontent.com/45053255/115467295-4a499200-a1ff-11eb-92ed-32615bd4de9a.png">

Bandpass Filter Simulation 
<img width="982" alt="Screen Shot 2021-04-20 at 5 46 07 PM" src="https://user-images.githubusercontent.com/45053255/115467936-4bc78a00-a200-11eb-91a7-7d4e037a11d4.png">

The Bandpass Filter real results matched the simulation trend for the most part. The real does not have as sharp of cutoffs outside the bandpass frequency range because of noise that is not simulated. 

### FFT on Arduino

The final part of the lab used the Bandpass filter we implemented and performing Fourier analysis on it. To perform Fourier analysis, I used the FFT on Arduino library, and then stored 257 samples from the microphone in an array using TCA interrrupt to make it only 257. Then, I performed Fourier analysis on those samples.

Spectrum for 500 Hz

<img width="576" alt="Screen Shot 2021-04-19 at 5 26 30 PM" src="https://user-images.githubusercontent.com/45053255/115476078-0ced0080-a20f-11eb-99b5-6314afc3d882.png">

Spectrum for 700 Hz
<img width="519" alt="Screen Shot 2021-04-20 at 7 33 47 PM" src="https://user-images.githubusercontent.com/45053255/115476183-550c2300-a20f-11eb-8bce-4d79b2af6ac4.png">

Spectrum for 900 Hz
<img width="500" alt="Screen Shot 2021-04-20 at 7 34 11 PM" src="https://user-images.githubusercontent.com/45053255/115476204-635a3f00-a20f-11eb-9bbc-f96589e74423.png">

