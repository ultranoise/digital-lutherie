# digital-lutherie Bela Workshop
Code examples and tutorials for programming Bela boards (bela.io) during the Digital Lutherie course (Postdigital Lutherie Master - Tangible Music Lab)

Contents of this page:
* [General Information](#general-information-on-bela)
* [Supercollider Cheatsheet](#supercollider-cheatsheet)
* [Pure Data Cheatsheet](#pure-data-cheatsheet)
* [Trill Craft examples](#trill-craft-essentials)
* [Audio Input Hardware](#audio-input-hardware)
* [Audio Input Software](#audio-input-software)

# General information on bela

Website: https://bela.io/

Get Started Guide: https://learn.bela.io/get-started-guide/

Learning how to code: https://bela.io/learn/

Writing Code with Supercollider: https://learn.bela.io/using-bela/languages/supercollider/

Writing Code with Pure Data: https://learn.bela.io/using-bela/languages/pure-data/

Technical reference (terminal, network, SD, etc): https://learn.bela.io/using-bela/

# Connect

```
http://bela.local/
```
or (Mac OS X)

```
192.168.7.2
```

or (Windows)

```
192.168.6.2
```

through terminal
```
ssh root@bela.local
```

# MIDI test

```
amidi -l
```

# Supercollider Cheatsheet

## Overview

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

## Digital in/out

In the previous example, notice the functions ```DigitalIn.ar() and DigitalOut.ar() ``` for accessing the digital pins.  

```var button = DigitalIn.ar(inPin);``` reads the digital input and stores the read value in ```var button```. This value is used to change the ```outPin``` value with ```DigitalOut.```

## Analog input

The function to read from an analog pin (e.g. a potentiometer) is ```AnalogIn.ar()```. 

Here an example on how to use analog input in a SynthDef:

```
	
	SynthDef('ledFade', {
	
		var rate = AnalogIn.ar(0).exprange(0.3, 20);  //use Analog 0 to define rate
	
		var amp = AnalogIn.ar(1);  //use Analog 1 to define amplitude
	
		// returns a value from 0-1
	
		rate.poll(1); amp.poll(1);  //Print the current output value of rate and amp, useful for debugging SynthDefs.
	
		AnalogOut.ar(0, SinOsc.ar(rate).range(0.0, amp)); //use rate and amplitude to control an oscillator
	
		// send to Analog Output 0
	
	}).add;

```
In this case two analog pins are read (0 and 1) using two ```AnalogIn.ar()``` functions. 

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

## Analog Out

In the Bela Mini there are no analog out pins, only in the Bela (standard) board. So you cannot use the function ```AnalogOut.ar()``` or you will get the error ```AnalogOut Error: the UGen needs BELA analog outputs enabled```


## Debugging and Plotting

The particular server-client architecture of supercollider requires special management of messages if you want to for example print sensor values in the console. The easiest is sending OSC messages from the server to the IDE running in your laptop. This can be done with the following a couple of ```SendReply.kr``` and ```OSCdef``` functions:

```
s = Server.default;

s.options.numAnalogInChannels = 2;
s.options.numAnalogOutChannels = 2;
s.options.numDigitalChannels = 16;
s.options.maxLogins = 4;  	   // set max number of clients
s.options.bindAddress = "0.0.0.0"; // allow anyone on the network connect to this server

s.options.blockSize = 16;
s.options.numInputBusChannels = 2;
s.options.numOutputBusChannels = 2;

s.waitForBoot{
	SynthDef('buttonControl', {arg inPin, outPin;
		// read the value at the input
		var sensor = AnalogIn.ar(0);
		
		var pitch = AnalogIn.ar(0).exprange( 200, 5000 );
		// analog input 1 controls the amplitude
		var gain = AnalogIn.ar(1); // returns a value from 0-1
		Out.ar(0, SinOsc.ar(pitch).dup * gain);
		//send the value to the console
		SendReply.kr(Impulse.kr(2), "/sensor", sensor); 		
		
	}).add;	
	
	s.sync;
	
	a = Synth('buttonControl', ['inPin', 1, 'outPin', 0]);
	OSCdef(\trill, {|msg| msg[3..].postln }, "/sensor");
};

ServerQuit.add({ 0.exit }); // quit if the button is pressed
```

In Supercollider we can also plot to the IDE Scope sending a signal to the BelaScpe bus (e.g. ```.belaScope(0)```), but you first need to define the number of Scope channels with ```s.options.belaMaxScopeChannels = 8;``` 

Here there is an example of plotting Audio Input:

```
s = Server.default;

s.options.numAnalogInChannels = 8;
s.options.numAnalogOutChannels = 8;
s.options.numDigitalChannels = 16;
s.options.numInputBusChannels = 2;
s.options.numOutputBusChannels = 2;
s.options.maxLogins = 4;
s.options.bindAddress = "0.0.0.0"; // allow anyone on the network connect to this server

s.options.belaMaxScopeChannels = 8;

s.waitForBoot({
	SynthDef("help-scope",{ arg out=0;
		var in = SoundIn.ar([0,1]).belaScope(0);
		Out.ar(out, in);
	}).play;
});

ServerQuit.add({ 0.exit }); // quit if the button is pressed
```

## Audio Processing
Example of dynamic pitch shifting
```
s = Server.default;

s.options.numAnalogInChannels = 8;
s.options.numAnalogOutChannels = 8;
s.options.numDigitalChannels = 16;
s.options.numInputBusChannels = 2;
s.options.numOutputBusChannels = 2;
s.options.maxLogins = 4;
s.options.bindAddress = "0.0.0.0"; // allow anyone on the network connect to this server

s.options.belaMaxScopeChannels = 8;

s.waitForBoot({
	SynthDef("help-scope",{ arg out=0;
		var in = SoundIn.ar([0,1]).belaScope(0);
		var proc = PitchShift.ar(in, 0.02, Line.kr(0.1,4,20), 0, 0.001);
		Out.ar(out, proc);
	}).play;
});

ServerQuit.add({ 0.exit }); // quit if the button is pressed
```

Example input periodic granulator

```
s = Server.default;

s.options.numAnalogInChannels = 2; // can only be 2, 4 or 8
s.options.numAnalogOutChannels = 2;
s.options.numDigitalChannels = 16;
s.options.maxLogins = 4;  	   // set max number of clients
s.options.bindAddress = "0.0.0.0"; // allow anyone on the network connect to this server

s.options.blockSize = 16;
s.options.numInputBusChannels = 2;
s.options.numOutputBusChannels = 2;

s.waitForBoot{
	
	//b = Buffer.alloc(s, 44100 * 4.0, 1);  // 4 seconds buffer
	
	
	SynthDef(\sagrain, {arg amp=1, grainDur=0.05, grainSpeed=10, panWidth=0.5;
    	var pan, granulizer;
    	pan = LFNoise0.kr(grainSpeed, panWidth);
    	granulizer = GrainIn.ar(2, Impulse.kr(grainSpeed), grainDur, SoundIn.ar([0,1]), pan);
    	Out.ar(0, granulizer * amp);
	}).add;
	
	x = Synth(\sagrain);

};

ServerQuit.add({ 0.exit }); // quit if the button is pressed
```

Play some tweets by Mr Magnusson

```
s = Server.default;

s.options.numAnalogInChannels = 8; // can be 2, 4 or 8
s.options.numAnalogOutChannels = 8; // can be 2, 4 or 8
s.options.numDigitalChannels = 16;
s.options.maxLogins = 8;

s.options.pgaGainLeft = 5;     // sets the pregain for the left audio input (dB)
s.options.pgaGainRight = 5;    // sets the pregain for the right audio input (dB)
s.options.headphoneLevel = -3; // sets the headphone level (-dB)
s.options.speakerMuted = 1;    // set true to mute the speaker amp and draw a little less power
s.options.dacLevel = 0;       // sets the gain of the analog dac to (dB)
s.options.adcLevel = 0;       // sets the gain of the analog adc to (dB)

s.options.blockSize = 16;
s.options.numInputBusChannels = 10;
s.options.numOutputBusChannels = 2;

//some tweets by Mr Magnusson
s.waitForBoot {
	//play{a=Saw;b=220;c=0.3;a.ar(b,c)+a.ar(LFNoise2.ar(1).range(1.1892,1.2599)*b,c)+a.ar(b*1.5,c)!2}
	//play{i=Dust.ar(4);a=0.5;b=5e-3;q=Decay2;p=PulseDivider;n=WhiteNoise.ar;(SinOsc.ar(80)*q.ar(p.ar(i,2),a,b)+(n*q.ar(p.ar(i,4),b,a)))!2}
    play{{a=SinOsc;l=LFNoise2;a.ar(666*a.ar(l.ar(l.ar(0.5))*9)*RLPF.ar(Saw.ar(9),l.ar(0.5).range(9,999),l.ar(2))).cubed}!2} 	
};

```

## Extensions and SC3 Plugins

Finally, you can use SC3 plugins. The board has them installed. For example, the following code uses the FM7 plugin:

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

## General

  * Workflow: Pd patch can only edited in your computer. You cannot edit Pd patches in the IDE. After every edit one needs to manuall upload (drop) the patch to the IDE.
  
  * Usually we first create a new Pd project and then we directly Drag-&-Drop the Pd patch into the browser.
  
  * Requirements for a patch to compile: 
  
  	1) the top level patch has to have the name ```_main.pd```
  	2) in case you are using an existing project, remove any render.cpp from the project folder (usually you will not notice this, only in case of errors)
  
## Inputs and Outputs

### Audio in/out

  * Stereo AUDIO IN/OUT objects: **[adc~ 1 2] [dac~ 1 2]** 
  
### Analog in/out pins

  * Analog in/out pins for sensor reading or actuators writing are also accessed using ```[adc~]``` and ```[dac~]``` objects from channels 3 to 10. Sensor data is obtained at audio signal rate. Therefore we usually do not use control rate objets but signal rate object (use *~ instead of *).
  
  * There are 8 analog input pins. Read Analog IN pins using [adc~ 3 4 5 6 7 8 9 10] . 
  
  * Write values to the analog OUT pins (LEDs, motors, etc) using [dac~ 3 4 5 6 7 8 9 10]. Remember that Bela mini does not have analog out pins (only the bigger version).
  * Often we make use of the [snapshot~] object to convert audio signals to numbers and [sig~] to convert numbers to signals.  
   
### Digital in/out pins

  * Digital in/out pin are accessed through messages at control rate (but it is also possible to deal with them at audio rate as we will see). 
  
  * Examples of how to deal with digital pins (in/out configuration & read/write):  
  
  ![This is an image](/images/digital-output-1.png) 
  ![This is an image](/images/digital-output-2.png) 
  ![This is an image](/images/digital-input-1.png) 
  ![This is an image](/images/digital-input-2.png) 
  
  * It is also possible to init a digital pin as AUDIO RATE using a tilde in the message afte the pin nr:
	
	![This is an image](/images/distance-sensor-1.png) 

### Debuggin and plotting
  
  * The Bela IDE Scope is accessed through 4 channels **[dac~ 27 28 29 30]**
  * You can use [print] to write messages to the console.
  * Use the [snapshot~] object to convert audio signals to numbers (which can be printed).
  

  
## Usage examples: 
  
  ![This is an image](/images/analog-input-4.png) 
  ![This is an image](/images/analog-input-7.png) 
  ![This is an image](/images/analog-output-1.png) 
  ![This is an image](/images/analog-output-3.png) 
  ![This is an image](/images/analog-output-5.png) 
  
	
* Sensor handling techniques
  
    ![This is an image](/images/sensor1.png)
    ![This is an image](/images/sensor2.png)
    ![This is an image](/images/sensor3.png)
    ![This is an image](/images/sensor4.png)
    
## Tips and extended techniques

    * Reading files from sdcard: [open /mnt/sd/sample01.wav( message to [readsf~] (similar to writing with writesf~ objects)
    
    * Communicating Pd and C++ is possible, e.g. C++ (sensor capture) -> Pd (receive messages to synth) : https://learn.bela.io/tutorials/pure-data/advanced/custom-render/
    
  
  * Crafting GUIs: https://learn.bela.io/the-ide/crafting-guis/
  
   


# Trill Craft Essentials

  * Bela guide: https://learn.bela.io/tutorials/trill-sensors/working-with-trill-craft/
  * How to connect it (I2C bus):
  
  ![This is an image](https://learn.bela.io/assets/images/fritzing/pd//capacitive-sensing.png)
  
  You only have to connect four cables: V+ to 3.3V, GND and the pins SDA and SCL of Trill to the respective pins of Bela. 
  
  * Changing the settings and sensitivity: https://learn.bela.io/using-trill/settings-and-sensitivity/ (check the example Trill -> general-settings to understand the parameters)
  
  ## Trill Craft in Pure Data: 
  https://learn.bela.io/tutorials/pure-data/sensors/capacitive-sensing/ (remember that in the IDE the Pd patch is visulized with a bug, it uses the first 4 inputs)
 
  ![This is an image](/images/pd-trill-example.png) 	
  
  ## Trill Craft in Supercollider: 
  We need to copy a UGen to the Bela. General Information:
  
  https://github.com/jreus/Trill_SC/
  
  How to install:
  
  1) Go to https://github.com/jreus/Trill_SC/ click on the green "Code" button and download a ZIP of this repository to your computer
  
  2) Extract the contents of the ZIP and rename the folder "Trill_SC" (remove the "-master" part).
  
  3) Open a Terminal in your computer and go to the folder where you downloaded Trill_SC (make use of "cd" commands)
  
  3.1) Mac and Linux computers: you simply open "Terminal"
  
  3.2) Windows: try typing "scp" in the terminal. If it does not exist, the easiest is downloading and installing the open source apps WindSCP (https://winscp.net/eng/index.php) or PuTTy https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
  
  4) In the terminal type "scp -r Trill_SC root@192.168.7.2:" to copy the folder to the Bela
  
  5) In the terminal type "ssh root@192.168.7.2" to remotely access the Bela board
  
  6) type "ln -s ~/Trill_SC/ext/Trill ~/.local/share/SuperCollider/Extensions/TrillUGens"
  
  7) type "exit" to close the ssh connection with the Bela board
  
  
  
  ## Example in Supercollider:
  
```
/**
Functional example of using TrillRaw UGen.

(C) 2019 Jonathan Reus

Refactored by Gorka Egino Arroyo (2024)

**/

// SETTINGS
~init = {
	postln("Setting SuperCollider initial configuration...");
	s.options.numAnalogInChannels = 8; // can be 2, 4 or 8
	s.options.numAnalogOutChannels = 8; // can be 2, 4 or 8
	s.options.numDigitalChannels = 16;
	s.options.maxLogins = 8;
	
	s.options.pgaGainLeft = 5;     // sets the pregain for the left audio input (dB)
	s.options.pgaGainRight = 5;    // sets the pregain for the right audio input (dB)
	s.options.headphoneLevel = -1; // sets the headphone level (-dB)
	s.options.speakerMuted = 1;    // set true to mute the speaker amp and draw a little less power
	s.options.dacLevel = 0;       // sets the gain of the analog dac to (dB)
	s.options.adcLevel = 0;       // sets the gain of the analog adc to (dB)
	
	s.options.blockSize = 16;
	s.options.numInputBusChannels = 10;
	s.options.numOutputBusChannels = 2;
	postln("SuperCollider configured successfully");
};


~trill_settings = ( // This is data structure is called "event"
    numTouchPads: 30, // maximum 30
    i2c_bus: 1, // I2C bus to use on BeagleBone, usually you want this to be 1
	i2c_address: 0x30, // the i2C address of Trill sensor
	noiseThresh: 0.01, // float: 0-0.0625, with 0.0625 being the highest noise thresh
	prescalerOpt: 2, // sensitivity option, int: 1-8 (1=highest sensitivity, play with this for complex Trill Craft setups)
);

// UTILITIES
OSCdef(\trill, {|msg| msg[3..].postln }, "/trill"); // utility for allowing trill values to be posted

// START
s = Server.default;
~init.();

s.waitForBoot {
	var postTrillValues = true;
	
	{|t_updateTrill = 1.0| // time between sensor readings, in seconds
		var raw_vals;
		var sig;

		raw_vals = TrillRaw.kr(~trill_settings[\i2c_bus], ~trill_settings[\i2c_address], ~trill_settings[\noiseThresh], ~trill_settings[\prescalerOpt], t_updateTrill);
		raw_vals = raw_vals[(0..~trill_settings[\numTouchPads]-1)]; // only keep the specified number of touch pads
		
		if (postTrillValues) {SendReply.kr(Impulse.kr(2), "/trill", raw_vals)};
		
		sig = SinOsc.ar(freq: (1..~trill_settings[\numTouchPads]) * 50); // harmonic sound with fundamental @ 50Hz; harmonicsN == touchPadsN
		sig = sig * Lag.kr(raw_vals, lagTime: 0.1); // multiply amp of each harmonic by the smoothed out value of the corresponding touch pad
		sig = sig * 0.6; // turn down the signal
		sig = Splay.ar(sig); // pan the harmonic sound in the stereo field
		sig = CombL.ar(sig.sum, maxdelaytime: 0.2, delaytime: 0.2, decaytime: 3.0) * 0.4 + sig; // add a comb delay line

		sig;
	}.play;
};
```

# Audio Input Hardware

You can connect any type of audio input signal, from contact microphones, electret capsules, condenser microphones, etc. You only have to take care of amplifying it correctly. 

A typical example is connecting an electret capsule through a 2.2KOhm pull up resistor (electret capsules need a bit of power supply to work): 
![This is an image](https://learn.bela.io/assets/images/fritzing/pd//recording-samples.png)

or do the same with the popular MAX4466 electret amplifier: 

![This is an image](/images/gy-max4466.jpg) 

Link: https://eckstein-shop.de/GY-MAX4466SoundSensorModuleElectretMicrophoneAmplifier-MAX4466EN

# Audio Input Software

In pure data: take a look at patch examples from: 

https://guitarextended.wordpress.com/audio-effects-for-guitar-with-pure-data/

In Supercollider:

Write microphone input to output using a bus in Supercollider:

```
s = Server.default;

s.options.numAnalogInChannels = 2; // can only be 2, 4 or 8
s.options.numAnalogOutChannels = 2;
s.options.numDigitalChannels = 16;
s.options.maxLogins = 4;  	   // set max number of clients
s.options.bindAddress = "0.0.0.0"; // allow anyone on the network connect to this server

s.options.blockSize = 16;
s.options.numInputBusChannels = 2;
s.options.numOutputBusChannels = 2;

s.waitForBoot{

		SynthDef("help-In", { arg out=0, in=0;
    		var input;
        	input = SoundIn.ar(in, 1);
        	Out.ar(out, input);
		}).add;

	//read the input and play it out on the left channel
	Synth("help-In", [\out, 1, \in, 1]);
};

ServerQuit.add({ 0.exit }); // quit if the button is pressed
```
Example of audio input processing (pitch shift), can you add more effects?

```
s.waitForBoot{

	SynthDef("help-In", { arg out=0, in=0;
    		var input;
    		var proc;
        	input = SoundIn.ar(in, 1);
        	proc = PitchShift.ar(input, 0.02, Line.kr(0.1,4,20), 0, 0.0001);
        	Out.ar(out, proc);
	}).add;

	//read the input and play it out on the left channel
	Synth("help-In", [\out, 1, \in, 1]);
};

ServerQuit.add({ 0.exit }); // quit if the button is pressed
```

Record 4 seconds of sound and play it back continously, can you change the playing rate?

```
s.waitForBoot{
	
	b = Buffer.alloc(s, 44100 * 4.0, 1);  // 4 seconds buffer
	
	SynthDef('looper', {arg buf, outChan=0;   //plays from buffer
		var sig = PlayBuf.ar(1, buf, 1, loop: 1);
		Out.ar(outChan, sig.dup);
	}).add;

	SynthDef('record', {arg buf;   //records in buffer
		var in = SoundIn.ar(1);
		RecordBuf.ar(in, buf);
	}).add;

	l = Synth('looper', ['buf', b]);
	r = Synth('record', ['buf', b]);
};

ServerQuit.add({ 0.exit }); // quit if the button is pressed
```
Randomly read the buffer
```s.waitForBoot{
	
	
	b = Buffer.alloc(s, 44100 * 4.0, 1);  // 4 seconds buffer
	
	SynthDef('looper', {arg buf, outChan=0;   //plays from buffer
		var sig = PlayBuf.ar(1, buf, 1, LFNoise0.ar(12)*BufFrames.ir(buf), loop:1)!2; //random read
		Out.ar(outChan, sig.dup);
	}).add;

	SynthDef('record', {arg buf;   //records in buffer
		var in = SoundIn.ar(1);
		RecordBuf.ar(in, buf);
	}).add;

	l = Synth('looper', ['buf', b]);
	r = Synth('record', ['buf', b]);
};

ServerQuit.add({ 0.exit }); // quit if the button is pressed
```


Granularize incoming audio with GrainIn.ar

```
s.waitForBoot{
	
	b = Buffer.alloc(s, 44100 * 4.0, 1);  // 4 seconds buffer
	
	
	SynthDef(\sagrain, {arg amp=1, grainDur=0.1, grainSpeed=10, panWidth=0.5;
    		var pan, granulizer;
    		pan = LFNoise0.kr(grainSpeed, panWidth);
    		granulizer = GrainIn.ar(2, Impulse.kr(grainSpeed), grainDur, SoundIn.ar(1), pan);
    		Out.ar(0, granulizer * amp);
	}).add;
	
	x = Synth(\sagrain);

};

ServerQuit.add({ 0.exit }); // quit if the button is pressed
```
