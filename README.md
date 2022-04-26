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

Printing sensor values at the IDE console has to be done with the function ```poll``` because values are converted to audio signals. For example:

```pitch.poll(1); gain.poll(1);```

from the code in the last example. 

Finally, if you want to can call SC3 plugins. The board has them installed. For example, the following code uses the FM7 plugin:

```
s.waitForBoot({

SynthDef(\fm7BelaTest,
	{|freq = 220, attackTime = 0.5, releaseTime = 4, gate = 1|
    var ctls = [
        // freq, phase, amp
		[ freq, 0,    1   ],
		[ freq*2, pi/2, 1],
		[ 0,    0,    0   ],
		[ 0,   0,    0   ],
		[ 0,   0,    0   ],
		[ 0,   0,    0   ]
    ];
	var mods = [
		[SinOsc.kr(0.001,0, 1.3), SinOsc.kr(2,0,1), 0, SinOsc.kr(0.015,0,1), 0, 0],
		[SinOsc.kr(0.0012,0, 0.3), SinOsc.kr(1.5,0,0.5), 0, SinOsc.kr(0.025,0,1), 0, 0],
		[0, 0, 0, 0, 0, 0],
		[0, 0, 0, 0, 0, 0],
		[0, 0, 0, 0, 0, 0],
		[0, 0, 0, 0, 0, 0]
    ];
	var env = EnvGen.kr(Env.adsr(attackTime, 1, 1, releaseTime),gate,doneAction:2);
	var fm = FM7.ar(ctls, mods).at([0,1])*env;
	Out.ar(0,fm);
}).send(s);

  s.sync;
  ~synth = Synth(\fm7BelaTest, [\freq, 110, \attackTime, 3], target: s);
});
```

# Pure Data Cheatsheet

  * The workflow: first edit a Pd patch in your computer then upload it to the Bela board. You cannot edit Pd patches in the IDE. 
  * Create a new project and directly Drag-&-Drop Pd patch into the browser
  
  * Requirements for a patch to compile: 
  
  	1) the top level patch has to have the name ```_main.pd```
  	2) in case you are using an existing project, remove any render.cpp from the project folder
  
  * Stereo AUDIO IN/OUT through: **[adc~ 1 2] [dac~ 1 2]** 
  
  * Analog in/out pins are accessed through adc~ and dac~. Audio rate signals and not control rate data are used to deal with them.
  	
  * Digital in/out pin are accessed through messages at control rate and it is also possible to deal with them at audio rate. 
  
  * Read Analog IN pins through [adc~ 3 4 5 6 7 8 9 10] (8 analog inputs). Write analog OUT pins through [dac~ 3 4 5 6 7 8 9 10]. Remember that Bela mini does not have analog out.
  
  * How to deal with digital pins (in/out configuration & read/write):  
  
  ![This is an image](/images/digital-output-1.png) 
  ![This is an image](/images/digital-output-2.png) 
  ![This is an image](/images/digital-input-1.png) 
  ![This is an image](/images/digital-input-2.png) 
  
  
  * The Bela IDE Scope is accessed through 4 channels **[dac~ 27 28 29 30]**
  
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
    





