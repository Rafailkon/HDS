


<details>
  <summary>Super Collider code</summary>

``` sclang
s.meter;
s.plotTree;
(
SynthDef(\synthP, {
	arg atk =2 , sus =0, rel=5, c1=1, c2=(-1),
	freq = 500, cfmin = 500, cfmax=2000, rqmin = 0.1,rqmax = 0.2, amp =1, detune=0.2,pan=0, out=0;
	var sig, env;
	env = EnvGen.kr(Env([0,1,1,0],[atk, sus, rel],[c1,0,c2]),doneAction:2);
	sig = Pulse.ar(freq* {LFNoise1.kr(0.5,detune).midiratio});
	sig = BPF.ar(sig,{LFNoise1.kr(0.2).exprange(cfmin,cfmax)},
		             {LFNoise1.kr(0.1).exprange(rqmin,rqmax)});
        sig = Pan2.ar(sig, pan);
	sig = sig * env * amp;
	Out.ar(out, sig);
}).add;

SynthDef(\pad,{
	arg freq = 440, atk= 0.005, rel = 0.3, amp = 1, pan=0, out = 0;
	var sig, env;
	sig = SinOsc.ar([freq,freq-223,4]);
	env = EnvGen.kr(Env.new([0,1,0],[atk,rel],[1,-1]),doneAction:2);
	sig = Pan2.ar(sig, pan, amp);
	sig = sig * env;
	Out.ar(out, sig);
}).add;

SynthDef(\noise,{
	arg out = 0 ;
        var sig;
	sig = PinkNoise.ar(0.5!2);
	2.do{sig = RLPF.ar(sig,LFNoise2.kr(0.5).exprange(100,5000))};
    Out.ar(out,sig);
}).add;

SynthDef(\reverb,{
	arg in, predelay = 0.1, revtime = 3, lpf = 4500, mix = 1, amp=1, out=0;
	var dry, wet, temp, sig;
	dry = In.ar(in,2);
	temp = In.ar(in,2);
	wet = 0;
	temp = DelayN.ar(temp, 0, 2, predelay);
	16.do{
		temp = AllpassN.ar(temp, 0.05, {Rand(0.001,0.05)}!2, revtime);
		temp = LPF.ar(temp, lpf);
		wet = wet + temp;
	};
	sig = XFade2.ar(dry,wet,mix*2-1,amp);
	Out.ar(out,sig);
}).add;
)
~reverbBus = Bus.audio(s,2);
~reverbSynth = Synth(\reverb,[\in,~reverbBus]);

(
~ambient = Pbind(
    \instrument, \pad,
    \dur, Pwhite(0.05, 0.5, inf),
	\midinote, 33,
	\harmonic, Pexprand(1, 80, inf).round,
	\atk, Pwhite(2.0, 3.0, inf),
	\rel, Pwhite(5.0, 3.0, inf),
	\atk, Pwhite(2.0, 3.0, inf),
	\amp, Pkey(\harmonic)*0.5,
	\pan, Pwhite(-0.8, 0.8, inf),
	\out, ~reverbBus
).play;
)

~ambient.stop;

~wind = Synth(\noise,[\out, ~reverbBus]);

~wind.free;

(
~drum =  Pbind(
	\instrument, \synthP,
	\dur,Prand([1,0.5],inf),
	\freq,Prand([0.4,8],inf),
	\detune,Pwhite(0,0.1,inf),
	\rqmin,0.005,
	\rqmax,0.008,
	\cfmin,Prand((Scale.minor.degrees+64).midicps,inf)*Prand([0.5,1,2,4],inf),
	\cfmax,Pkey(\cfmin)*Pwhite(1.008,1.025,inf),
	\atk, 3,
	\sus,1,
	\rel,5,
	\amp,7,
	\pan, Pwhite(-0.8, 0.8, inf),
	\out, ~reverbBus
).play;
)
~drum.stop;

s.record;
s.stopRecording;
 ``` 
  </details>
 
 <details>
  <summary>Processing code</summary>

```java 
import processing.sound.*;
SoundFile track;
FFT fft;       
int bands = 8192;
float[] spectrum = new float[bands]; 
float angle= -QUARTER_PI*3;

void setup()
{
  surface.setLocation(200,height/5);
  size(1000, 1000);
  background(0);
  fft = new FFT(this, bands);
  track = new SoundFile(this, "SuperC.aiff");
  track.loop();
  fft.input(track);
}
void draw() {
  fft.analyze();
  translate(width/2, height/2);
  rotate(angle);
  for (int i = 0; i<bands; i++)
    {
    strokeWeight(3);
    spectrum[i] = fft.spectrum[i]*25;
    stroke(255,spectrum[i]*255);  
    point(i,i);
    }
  angle += PI/(256*4)*0.1605; 
   
  
  if( angle >= QUARTER_PI*5){
    //background(0);
    //angle = -QUARTER_PI*3;
    noLoop();
    track.stop();
  }
}
  ```
  </details>
