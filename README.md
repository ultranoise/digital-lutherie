# digital-lutherie
Code examples and tutorials for Digital Lutherie (Postdigital Lutherie Master - Tangible Music Lab)

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

Digital In/Out from Buttons or Sensors:

```
	
SynthDef('buttonControl', {
	
		arg inPin, outPin;	
		
		var button = DigitalIn.ar(inPin);
	
		DigitalOut.ar(outPin, button);
	
	}).add;
	
```

Analog In:

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

# Pure Data Cheatsheet

  * Drag-&-Drop patch into the browser
  
  * Requirements: 
  
  1) _main.pd is the top level patch 
  2) Remove any render.cpp from the project folder
  
  * AUDIO IN/OUT: **[adc~ 1 2] [dac~ 1 2]** 
  
  * Analog IN/OUT: **[dac~ 3 4 5 6 7 8 9 10] [adc~ 3 4 5 6 7 8 9 10]** -> Analog in/out 0-7
  
  * Digital pins:  {{ :tamlab:lectures:digital:digital-output-1.png?nolink&400 |}} {{ :tamlab:lectures:digital:digital-output-2.png?nolink&400 |}} {{ :tamlab:lectures:digital:digital-input-1.png?nolink&400 |}} {{ :tamlab:lectures:digital:digital-input-2.png?nolink&400 |}}
  
  * Bela Scope: 4 channels **[dac~ 27 28 29 30]**
  
  * Init digital inputs or outputs at audio rate: **[out 11 ~ , in 12~ ( - [s bela_setDigital]** {{ :tamlab:lectures:digital:distance-sensor-1.png?nolink&600 |}}
  *
  * Examples: {{ :tamlab:lectures:digital:analog-input-1.png?nolink&600 |}} {{ :tamlab:lectures:digital:analog-input-4.png?nolink&600 |}} {{ :tamlab:lectures:digital:analog-input-5.png?nolink&600 |}} {{ :tamlab:lectures:digital:analog-input-7.png?nolink&600 |}} {{ :tamlab:lectures:digital:analog-output-1.png?nolink&600 |}} {{ :tamlab:lectures:digital:analog-output-3.png?nolink&600 |}} {{ :tamlab:lectures:digital:analog-output-5.png?nolink&600 |}}
  
  * Communicating Pd and C++, e.g. C++ (sensor catch) -> Pd (receive messages to synth) : https://learn.bela.io/tutorials/pure-data/advanced/custom-render/
  
  * Crafting GUIs: https://learn.bela.io/the-ide/crafting-guis/
  
  * Reading files from sdcard: **[open /mnt/sd/sample01.wav( message to [readsf~]** (similar for writing with writesf~)
  
  * Sensor handling techniques: {{ :tamlab:lectures:digital:sensor1.png?nolink&800 |}} {{ :tamlab:lectures:digital:sensor2.png?nolink&800 |}} {{ :tamlab:lectures:digital:sensor3.png?nolink&800 |}} {{ :tamlab:lectures:digital:sensor4.png?nolink&800 |}}






