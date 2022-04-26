# digital-lutherie
Code examples and tutorials for programming Bela boards (bela.io) during the Digital Lutherie course (Postdigital Lutherie Master - Tangible Music Lab)

# General information on bela

Website: https://bela.io/

Get Started Guide: https://learn.bela.io/get-started-guide/

Writing Code with Supercollider: https://learn.bela.io/using-bela/languages/supercollider/

Writing Code with Pure Data: https://learn.bela.io/using-bela/languages/pure-data/

# Supercollider Cheatsheet


Only write your code within this block:

```
s.waitForBoot{
	
};
```

The general idea to add interaction is defining Synths (with Synthdef) where some parameter depends on the digital or analog input readings. 
For example: 

```
s.waitForBoot{
	
	SynthDef('buttonControl', {arg inPin, outPin;
	
		var button = DigitalIn.ar(inPin);     //read button
	
		DigitalOut.ar(outPin, button);        //switch on/off a LED
	
	}).add;	
	
	s.sync;
	
	a = Synth('buttonControl', ['inPin', 7, 'outPin', 6]);
};
```

In this example ```var button = DigitalIn.ar(inPin);``` reads the digital input and stores the read value in ```var button```. This value is used to change the ```outPin``` value with ```DigitalOut.```

Notice the functions ```DigitalIn.ar() and DigitalOut.ar() ``` for accessing the digital pins. 

Here an example on how to read analog input:

```
	
	SynthDef('ledFade', {
	
		var rate = AnalogIn.ar(0).exprange(0.3, 20);
	
		var amp = AnalogIn.ar(1);
	
		// returns a value from 0-1
	
		rate.poll(1); amp.poll(1);
	
		AnalogOut.ar(0, SinOsc.ar(rate).range(0.0, amp));
	
		// send to Analog Output 0
	
	}).add;

```
The function to read from an analog pin is ```AnalogIn.ar()```. In this case two pins are read (0 and 1) with two ```AnalogIn.ar()```

This could be an example of use: 

```
s.waitForBoot{
	"Server Booted".postln;
	(
	SynthDef("AnalogIn-test",{ arg out=0;
		// analog input 0 controls the pitch
		var pitch = AnalogIn.ar(0).exprange( 200, 5000 );
		// analog input 1 controls the amplitude
		var gain = AnalogIn.ar(1); // returns a value from 0-1
		Out.ar(0, SinOsc.ar(pitch).dup * gain);
	}).send(s);
	);

	s.sync;
	Synth.new("AnalogIn-test", target: s);
};
```

In the Bela mini there are no analog out pins, only in the Bela (standard) board. So you cannot use the function ```AnalogOut.ar()``` or you will get the error ```AnalogOut Error: the UGen needs BELA analog outputs enabled```

Printing sensor values at the IDE console has to be done with the function ```poll``` because the values as read as audio signals. For example:

```rate.poll(1); amp.poll(1);```


# Pure Data Cheatsheet

  * Drag-&-Drop patch into the browser
  
  * Requirements: 
  
  1) _main.pd is the top level patch 
  2) Remove any render.cpp from the project folder
  
  * AUDIO IN/OUT: **[adc~ 1 2] [dac~ 1 2]** 
  
  * Analog IN/OUT: **[dac~ 3 4 5 6 7 8 9 10] [adc~ 3 4 5 6 7 8 9 10]** -> Analog in/out 0-7
  
  * Digital pins:  
  
  ![This is an image](/images/digital-output-1.png) 
  ![This is an image](/images/digital-output-2.png) 
  ![This is an image](/images/digital-input-1.png) 
  ![This is an image](/images/digital-input-2.png) 
  
  
  * Bela Scope: 4 channels **[dac~ 27 28 29 30]**
  
  * Init digital inputs or outputs at audio rate: **[out 11 ~ , in 12~ ( - [s bela_setDigital]** 
  
  ![This is an image](/images/distance-sensor-1.png) 

  
  * Examples: 
  ![This is an image](/images/analog-input-1.png) 
  ![This is an image](/images/analog-input-4.png) 
  ![This is an image](/images/analog-input-5.png) 
  ![This is an image](/images/analog-input-7.png) 
  ![This is an image](/images/analog-output-1.png) 
  ![This is an image](/images/analog-output-3.png) 
  ![This is an image](/images/analog-output-5.png) 
  
  
  * Sensor handling techniques:
  
    ![This is an image](/images/sensor1.png)
    ![This is an image](/images/sensor2.png)
    ![This is an image](/images/sensor3.png)
    ![This is an image](/images/sensor4.png)
    
    * Communicating Pd and C++, e.g. C++ (sensor catch) -> Pd (receive messages to synth) : https://learn.bela.io/tutorials/pure-data/advanced/custom-render/
  
  * Crafting GUIs: https://learn.bela.io/the-ide/crafting-guis/
  
  * Reading files from sdcard: **[open /mnt/sd/sample01.wav( message to [readsf~]** (similar for writing with writesf~)
    





