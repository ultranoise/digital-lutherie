# digital-lutherie
Code examples and tutorials for Digital Lutherie (Postdigital Lutherie Master - Tangible Music Lab)

# General information on bela

Website: https://bela.io/

Get Started Guide: https://learn.bela.io/get-started-guide/

Writing Code with Supercollider: https://learn.bela.io/using-bela/languages/supercollider/

Writing Code with Pure Data: https://learn.bela.io/using-bela/languages/pure-data/

# Supercollider Cheatsheet


Only write your code within this block:

<code>
s.waitForBoot{
	
};
</code>

The general idea to add interaction is defining Synths (with Synthdef) where some parameter depends on the digital or analog input readings. 
For example: 

<code>
	
s.waitForBoot{
	
	SynthDef('buttonControl', {arg inPin, outPin;
	
		var button = DigitalIn.ar(inPin);     //read button
	
		DigitalOut.ar(outPin, button);        //switch on/off a LED
	
	}).add;	
	
	s.sync;
	
	a = Synth('buttonControl', ['inPin', 7, 'outPin', 6]);
};
</code>

How to Digital In/Out from Buttons or Sensors:
<code>
SynthDef('buttonControl', {arg inPin, outPin;
		var button = DigitalIn.ar(inPin);
		DigitalOut.ar(outPin, button);
	}).add;
</code>

Analog In
<code>
	SynthDef('ledFade', {
		var rate = AnalogIn.ar(0).exprange(0.3, 20);
		var amp = AnalogIn.ar(1);
		// returns a value from 0-1
		rate.poll(1); amp.poll(1);
		AnalogOut.ar(0, SinOsc.ar(rate).range(0.0, amp));
		// send to Analog Output 0
	}).add;

</code>


