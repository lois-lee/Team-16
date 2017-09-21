
# Lab 2: 
For this lab, we split up into 2 subteams blah blah blah

[Original Lab Manual Here](https://cei-lab.github.io/ece3400/lab2.html) 




### Acoustic Team:

The goal of this sublab was to enable our robot to detect a 660hz frequency. In order to do this we had to:
  * do a correct FFT analysis, 
  * have a working amplifier circuit and 
  * also be able to distinguish a 660 hz from a 585hz and 735 hz. 

This acoustic aspect of the lab is important because later on the robot will use this 660Hz signal as its start signal.

**Introduction**

For this lab, we needed a microphone, arduino uno, and some resistors and capacitors. We did not have to assemble the microphone circuit as in past years since the microphone we used (MAX4466) already came with a low pass filter, which meant that we would not have to use the capacitor and 3 kohm resistor. 

However, we still wanted to check if the microphone worked properly, so we connected it to the arduino (don't forget to power the arduino) and then we hooked it up to the oscilloscope, turned on the 660 Hz (moving around the potentiometer to change gain) and adjusted the scale. In the video, in the oscilloscope measurement you can see it says 660 Hz, reconfirming that it has read the correct value.

video

**FFT analysis**

For this part of the lab we decided to use analogRead. Note that when we use fft_input, each index i that is even represents a real signal, whereas the odd signals are the imaginary components. Thus in the loop:

```
 for (int i = 0 ; i < 512 ; i += 2) { // save 256 samples
      fft_input[i] = analogRead(A0); // put real data into even bins
      fft_input[i+1] = 0; // set odd bins to 0
```

We have the the even index 1 equal to the analogRead from pin A0, an i+1 index (the odd one) equal to zero.
Then the for loop is incremented by two for each bin.

To check the code and make sure it was working correctly, we hooked it up to the function generator. We also hooked up the oscilloscope to make sure the function generator was outputting as well. We had the arduino print out the values, then we transfered that data to excel and created a bar graph for each bin and signal. 

Bar graph image

*Bar graph image explanation:*

To start off, note first that the bar graph has a peak in either the 19th or the 20th bin, this is supposed to be around the 17th or 18th, but in our case, it does not change our process or matter that much since everything will be relative, and since we have measured with the oscilloscope that it was recieving the same 660Hz created by the function generator. We have the other peaks that are the multiples of 660Hz, so that we can be sure the spacing between each peak is equidistant and that the 20 bin number is correct after all.

However, this was not good enough to distinguish from the 585hz and the 735hz. If you look closely at the following plot:

*plot of 585 and 735 with 660

you can see that the 585hz and 735hz has overlap somewhat with the 660hz. Thus it is necessary for us to further filter and amplify the signal.


**Amplifier Circuit**

Rather than making a filter and an amplifier separately, we decided to make a bandpass filter with gain.

Using [this website](analog.com/designtools/en/filterwizard/) we mapped out what we wanted our Bode plot to look like, such that our 660 Hz signal would be amplified but all others would be minimized. We first started off with a gain of 40db, or 100. The website then outputs a bandpass filter circuit schematic.

*image of bode plot
*image of circuit schematic

Note that on the website it will output a circuit for voltage range from 5V to -5V, however we want from 5V to 0V. If you change this value, they will give you a REF schematic as well. However this is unnecessary. We simply used a voltage divider to connect the REF and give each 2.5V.

*image of actual physical circuit


**Distinguishing the 660hz from 585hz and 735hz**

In review, the signal that inputs into the microphone gets preliminarily filtered and amplified by our bypass filter with gain. Then the filter outputs into the A0 input pin of the arduino. The arduino code has different amplitudes in each bin, and we can recall that the 660hz amplitude was around the bin 19 and 20. We then added the following code:

```
    for (byte i = 0 ; i < FFT_N/2 ; i++) { 
      Serial.println(fft_log_out[i]); // send out the data
      if (fft_input[38] > 60)
        digitalWrite(LED_BUILTIN, HIGH);      

```

The purpose of this was to be able to show physically that the arduino responded to the 660hz, rather than the 585hz or the 735hz. We did this by making the arduino LED light up when it detected an amplitude higher than 60 in the 19th bin. Recall that the function fft_input allocates two different indexes for each bin, one real and one imaginary. Thus when we want to reference the 19th bin, we actually have to call the 38th as we did in the above code.

You can see in [this video](add video link)that the LED does not light up during the 585 and 735hz tones. This is because we filtered and amplified only the 660hz, so that the amplitude of the 660hz would be the only one with a significantly high amplitude. The 60 value can be adjusted, however we found that our coe worked best with the 60.

**Full code for acoustic team**


