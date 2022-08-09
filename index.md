<h1 align="center">AES Synthesizer System Design</h1>

# 0.0 The Goal (And What To Expect)
Building a synthesizer is difficult. I'm not saying this to scare you away, but to prepare you: this will be a challenge. That said, it will be an extremely rewarding challenge, and a great accomplishment. Synthesizers combine knowledge from across an electrical engineering education, and building one requires applying your knowledge in a way that will solidify your understanding of electronics and make you a far better engineer. Of course, any difficult journey deserves proper motivation.

By the end of this series, you will have a synthesizer consisting of four modules: an oscillator, amplifier, filter, and envelope generator (ADSR).

The filter consists of a simple high- and low-pass stage, controllable via potentiometer knobs. This system applies the concepts of impedance and frequency-dependence, and we will have to make decisions as designers about sensitivity and filter range. The filter is not voltage-controlled, although you will be able to do this by the end of this series, by applying the fundamentals we cover in other subsystems.

 The oscillator, by contrast, _is_ voltage-controlled. This means that the pitch can be set via an adjustable power supply, an Arduino, a set of mechanical switches, or any other voltage output we can control. It is capable of generating sawtooth waves from 20Hz - 3kHz, and you will have the knowledge to modify it to produce triangle and pulse-width signals, as well. In the process, you'll learn about operational amplifiers and comparators, relaxation oscillators, negative and positive feedback, hysteresis, transistor switches, as well as parasitics and other real-world implementation concerns.
 
 The amplifier is also voltage-controlled, and will amplify tiny signals (~10mV) so that they can be played on our speakers. This subsystem will teach you about differential pairs, AC analysis, the transistor "linear" operation, and component matching.

 Finally, the envelope generator is a "triggered" circuit that uses resistor-capacitor (RC) networks to create predictable, controllable time-varying voltage signals, giving the full ADSR control. By the time we get here, you'll have a lot of knowledge and practice at building circuits, so this is the most "flexible" design — you'll get to make a lot of decisions that will shape the sound and how your synth responds to user input. This section will teach concepts of ADSR, RC time constants, inverting and summing op-amps, and transistor discrete logic.

 In total, this series will familiarize you with content from the introductory Circuits series, as well as EE 306/307/308, and 409. The list of topics covered makes this sound like a project for 3rd- or 4th-year students in Electrical Engineering, but the whole point of this project is that you don't need those classes (yet!) to get started building synthesizers. Most people, with a bit of effort, can understand the fundamentals of how these systems work. In the process, you'll get to build something really special: your own DIY synth.  

  
# 1.0 Fundamentals

## Electronics
- Simulation & Breadboarding
	- Simulations are not perfect. Falstad < SPICE < Reality
	- Breadboards are not perfect
- Ohm's Law: V=IR. Definitions & relationships
- Voltage dividers
- AC vs DC
- Reactive Components (X, Z), Energy Storage, Frequency Dependence
	- RC Time Constants
	- Resonance / Tank Circuits
- Diodes

## Basic Waveforms
- Square / Saw / Tri / Sine
- Harmonics

## Sound & Audio Fundamentals
- Pitch as oscillation, frequency & period
- Timbre
- Range of human hearing


# The Synthesizer

## Bill of Materials (BOM)

## System Overview

On any synthesizer, we want the core functionality to create a pitch, change its volume, and shape the sound (or "timbre"). This leads every synthesizer to be made up of at least three basic subsystems:

1. Oscillator: Produces a tone at a given frequency
2. Filter: Shapes the sound
3. Amplifier: Provides gain (amplification) to change volume

One optional, yet common, additional feature found in most synths is an envelope generator, also called "ADSR". This allows for the generation of low-frequency, time-varying voltage signals which can be used as input to other subsystems.

4. ADSR: Creates a time-varying voltage envelope to modulate other subsystems

Control of each subsystem is accomplished either by modifying the circuit directly, or by introducing an external CV "control voltage". Subsystems which can be controlled by CV are referred to as "voltage-controlled", meaning we can have voltage-controlled oscillators (VCOs), filters (VCFs), and amplifiers (VCAs). 

The diagram below shows how all of these subsystems interact in a basic synthesizer. Notice the main signal path flows from oscillator → filter → amplifier, while the ADSR envelope is passed as an additional (optional) input to some of the subsystems.

<img src="res/synth-signal-flow.png" height=300 />

_Typical synthesizer subsystems and signal flow._ [^mit_syw]*

The diagram also gives us some other information about our system, such as the existence of input control voltage, an offset, and a gate. The significance of these terms isn't extremely important right now, but will become clear as we examine each subsystem in detail. For now, simply knowing that these inputs and outputs exist is enough to inform our design process.

Finally, the diagram also indicates possibilities for more advanced synthesizers, including multiple oscillators (VCO2) and noise inputs. These add additional layers of sound, at the cost of additional complexity. These features are not included in this synthesizer build as they are not essential to sound synthesis; however, they are common modifications which enhance a synth's feature set, and are briefly discussed as [future work](#future-work).


### Design Constraints

We need to build each subsystem separately, yet each one relies on others in order to function properly. This presents a challenge because it means that decisions we make in one part of the system design can affect other systems, causing design changes, adjustments, and compromises down the line. 

One way to avoid this chaos is to outline some design constraints. For this project, we'll want to consider the following:
- Power levels (aka "rails")
- Signal levels
- Subsystem inputs & outputs

DIY synthesizer design can adhere to many arbitrary constraints; however, the popularity of synth gear has led to some conventions which we can use to inform our design. For example, the Eurorack "standard" (popularized by the company Doepfer) has led much of the synthesizer community to adopt a split-supply ±12V / GND for power; signals range between ±5V or 0-10V; envelopes typically adhere to 0-8V. [^pc_eurorack]

For simplicity, our design will use the following constraints across the entire system:
- Power Rails: ±12V & GND
- Control Voltage (CV) Signals: 0-5V 
- Output Signal: 0-5Vpp

Power and signal levels typically have the most impact to the overall system, so with these constraints, we should be able to avoid any big design problems.

## 2.1 A Basic Speaker Output Buffer
The final output stage of our synth will be ready for a speaker to play, but throughout the design process, we'll want to listen to the signal that we have. Sometimes, this signal isn't safe for a speaker to play — for example, there may be a DC offset, which desktop speakers aren't built to handle. Other times, listening to the signal might disturb the circuit, altering its behavior and changing the sound significantly. We can solve both of these problems by building an audio output buffer, which isolates the signal and removes any unsafe DC offsets without _much_ distortion.

<img src="res/speaker-buffer-falstad.png" height=300 />

_[Speaker buffer schematic [Falstad]](https://tinyurl.com/22qtc32k)_

We haven't explored filters, op-amps, or negative feedback yet, so don't expect to understand this circuit right away. (After we build an oscillator, though, you'll understand it completely.) For now, building the circuit and testing it using lab instruments will be challenging enough if it's your first circuit, so we'll use this as an opportunity to get familiar with building circuits, breadboarding, and troubleshooting. 

### Building the Speaker Buffer
Using the schematic above, try to draw up your own physical layout (eg. how it will look on a breadboard) before implementing the circuit on a breadboard. (You'll want to draw layouts by hand, by the way.)Notice that some things are missing from the schematic: 

- power connections
- IC pinout
- capacitor type
- I/O connectors (How do you plug in an aux cable for speakers?)

#### Pinouts
Here's an example breadboard layout using the LM358 and a generic TRS jack. Layouts are part-specific, so if you use different parts, you'll need to check the datasheets and adjust the pins to match!

#### Capacitor Type
If there is no signal offset, or if the offset is negative, the capacitor could be _reverse-biased_, meaning that a negative voltage is applied to the capacitor. This is a possibility because we are building a general-purpose buffer — we don't know what signals we'll listen to! Our signals will frequently have a positive DC offset, but this isn't always the case.

Therefore, the capacitor used in this circuit **must** be _non-polarized_. Avoid using electrolytics — if one lead is longer than the other, this is a likely sign that you've got a polarized capacitor!

#### I/O Connector Jack
I recommend using the STX-3000 barrel jack connector from the [BOM](#bom). (With the disclaimer that the pinout is *not* the same as in the layout below!) 

However, how did we get to this particular connector? It turns out that, even if you know that you want to use a 3.5mm audio connector, component selection is not straightforward! Check out the section on [component selection](#component-selection) for considerations on how we chose this particular part.

#### Breadboard Layout
<img src="res/speaker-buffer-layout.png" height=300 />

Notes:
- Signal_In is an AC signal (eg. sine wave) and can be created using a function generator. 
	- Frequency: [20 Hz, 10 kHz]
	- Amplitude: 1Vpp
	- Offset: [0V, 8V]
- Check the TRS jack datasheet to verify the ordering of connections. The STX-3000, for example, uses the middle pin as "Sleeve", with "Tip" and "Ring" on top & bottom.
- Always test op-amp buffers to verify that the output (**Out1**) has the same voltage / signal as the input (**In1+**).
- The left "+" rail of the breadboard is unused. You could choose to connect the **Signal_In** to it for convenience.

Check the output using an oscilloscope to verify it is not too large (1Vpp) and has its DC offset removed. Do this _before_ connecting the speakers!

This is what your breadboarded circuit might look like:
<img src="res/speaker-buffer-bb.png" height=300 />

You're ready to listen to signals!


## The Filter
Given that synthesizer signal flow originates with the oscillator, it might seem that this is the best place to start. For our system design, though, we'll want to start with something simpler which lays the groundwork for more complex subsystems. The filter of a synthesizer represents an excellent starting point. As we build the filter, we will become familiar with electronics fundamentals and engineering design principles which will apply to future subsystem development, and even future system design.

## The Oscillator
The oscillator is the core of any synthesizer, generating periodic signals that we recognize as sound. There are many circuits which produce oscillations, but in general, oscillators can be categorized as either harmonic or relaxation.

| | Harmonic | Relaxation |
|---|----|----|
| **Feedback Mechanism:** | Amplification | Switching |
| **Method:** | Excites oscillations in a resonator (eg. crystal) | Repetitively charges/discharges an energy-storage device to a threshold |
| **Output:** | Sinusoidal | Non-linear (square / triangle / saw) |
| **Example:** | ![](res/signal-example-harmonic.png) | ![](res/signal-example-relaxation.png) |

While many synthesizers can output sinusoidal signals, sinusoids lack harmonics and therefore provide limited options for shaping the sound using subtractive filters. Therefore, we will build a relaxation oscillator, which can produce a waveform that is rich in harmonics. 

Sawtooths are frequently preferred for synthesis because they contain both even and odd harmonics, giving additional flexibility when shaping signals through a filter.[^mit_syw] (For a review of waveform harmonics, refer to [Basic Waveforms](#basic-waveforms).) A sawtooth core also provides flexibility for further development down the line, as it can be processed to produce other waveforms such as rectangular and triangle waves. 

A quick Google search will provide many relaxation oscillator designs. We will use a relatively simple sawtooth core design sourced from [Moritz Klein's YouTube channel](https://www.youtube.com/watch?v=QBatvo8bCa4). In order to understand how it works, we need to learn about comparators and hysteresis.

### Comparators
Comparators can be thought of as a subset of a more general device we'll examine first, called an "operational amplifier." These devices have two inputs (the non-inverting **In+**, and the inverting input **In-**) and a single output (**Out**), and are represented in schematics by a triangle like so: 

<img src="res/ideal-opamp.jpg" height=200 />

The name "operational amplifier" (aka "opamp") originates from a component that can do mathematical _operations_ with amplification. Depending on its placement and configuration in the circuit, the operational amplifier can be made to perform a variety of computations, from basic arithmetic (eg. addition and subtraction) to advanced operations such as integration or differentiation. However, the output follows a simple formula which, in isolation, provides a principle of operation for us to understand:

$$V_{out} = A_{opamp}*(V_+ - V_-)$$

In the ideal case, op-amps have an _amplification_ ( $A_{opamp}$ ) that is infinite, but even practical op-amps have enormous internal gain ( $A$ ≈ $10^5$ ). Therefore, the opamp functions like so: take the difference of the two input voltages, multiply it by a really big number, and output that voltage. 

You might imagine a few ways this could go:

| $V_+$ | $V_-$ | $A*(V_+ - V_-)$ | $V_{out}$ |
|-------|-------|---------------|-----------|
| 5 | 3 | $10^5 * (2)$ | $2*10^5$ |
| 3 | 5 | $10^5 * (-2)$ | $-2*10^5$ |
| -5 | 3 | $10^5 * (-8)$ | $-8*10^5$ |
| 5 | -3 | $10^5 * (8)$ | $8*10^5$ |
| -5 | -3 | $10^5 * (-2)$ | $-2*10^5$ |

In reality, the op-amp will _try_ to output the necessary voltage, but it is limited by the power supplies (the "rails"). In our case, our circuits run on ±12V, so instead of reaching voltages ~ $10^5$, the opamp in each given case above outputs either a positive or negative 12V. 

| $V_+$ | $V_-$ | $A*(V_+ - V_-)$ | $V_{out}$ |
|-------|-------|---------------|-----------|
| 5 | 3 | $10^5 * (2)$ | +12V |
| 3 | 5 | $10^5 * (-2)$ | -12V |
| -5 | 3 | $10^5 * (-8)$ | -12V |
| 5 | -3 | $10^5 * (8)$ | +12V |
| -5 | -3 | $10^5 * (-2)$ | -12V |

Notice that the output depends solely on the sign of $(V_+ - V_-)$. This setup is called a "comparator" topology, and it is this operation that the comparator excels at: the output is only "HIGH" or "LOW", and depends solely on a comparison of one input vs the other. 

Look at the figure below, and check out [this Falstad simulation](https://tinyurl.com/2pw6lrbk) of the comparator topology, to check your intuition about how this works.

<img src="res/opamp-twoinputs.png" height=300 />

_Different inputs cause the opamp output to hit either rail, depending on ( $V_+ - V_-$ ) [[Falstad]](https://tinyurl.com/2jkduams)_

(Side-note: Falstad doesn't have a "comparator" part, and in fact, these simulations all use op-amps. However, because a comparator is a subset of operational amplifiers (where the "operation" is comparison), you can sometimes use an op-amp to represent a comparator. In fact, some designers even use op-amp ICs in place of comparators! (As we'll eventually discover, though, this can be bad practice, and if your op-amp is only being used to compare (by swinging to the rails), a comparator is a better choice.)

#### Comparators With Reference
While comparator setups could be used to compare any two voltages, typical implementations set one of the inputs as a fixed value, to which the other input is compared. For example, we can check whether a signal is higher than 3V by setting the inverting (-) input to a "trip point" of 3V. This makes $V_- = 3$, such that the output swings "high" to the positive rail when $(V_+ - V_-) = (V_+ - 3) > 0$.

<img src="res/opamp-comparator-ref.png" height=300 />

_Comparator with reference at 3V. Input sweeps [-5V,5V]. [[Falstad]](https://tinyurl.com/2kqvkuxf)_

Typically, the reference level doesn't simply *exist* — that is, we have ±12V rails, but we wouldn't want to get a new power supply every time we need to make a comparison! (Eg. We might want to compare against 1V, 3V, 5V, etc.) Instead, common practice is to use [voltage dividers](#voltage-divider) or [zener diodes](#diodes) to set the comparator reference, while the other input is variable. This allows us to set any voltage within the rails as our reference.

<img src="res/opamp-comparator-ref-vdiv.png" height=300 />

_Setting the reference to 3V via voltage dividers [[Falstad]](https://tinyurl.com/2f42ud68)_

#### Hysteretic Comparators
What if we wanted more than one trip point? That is, what if we wanted the comparator output to swing "high" at one voltage, but swing "low" at a different voltage? This is known as "hysteresis", and will take some setup conceptually; however, physically it is simple to achieve. Importantly, this is the key to creating a relaxation oscillator!

First, we need to develop our intuition about what a voltage divider does. Imagine a voltage divider as "dividing" the voltage between two points. Typically, this is a fixed voltage rail, such as our 12V VCC, and GND; the output exists between the two resistors and depends on their ratio. However, what if we moved this second side, so that it wasn't connected to GND? In that case, the output would still be set "between" the two resistors, but with the endpoints shifted. You can see this principle demonstrated in the three voltage dividers below, where (due to the equal resistances) the output voltage is always halfway between the endpoints.

<img src="res/voltage-div-shifting.png" height=300/>

In general, given two endpoints $V_2$ and $V_1$:

$$V_{out} = V_1\frac{R_1}{R_1+R_2} + V_2\frac{R_2}{R_1 + R_2}$$

[(Click here for the simple proof!)]()

This is important because we already have two possible outputs from our comparator: "high" & "low" (±12V). If we tie the comparator output to one end of the voltage divider, then the voltage divider will similarly shift its output. We can use this to set a changing reference value (or "trip" point) at the comparator's input.

<img src="res/hysteretic-divider.gif" height=300 />

This technique of using the output to influence one of the inputs is called "feedback". We'll dive into feedback more when we look further into op-amps, but for now, know that hysteresis uses _positive_ feedback, meaning that we must use the non-inverting (+) input as our reference.

Applying this to the comparator circuit, we arrive at a *hysteretic* comparator, with two different trip points. Just like the voltage divider above, you can see below that the non-inverting (+) reference input (red line) maintains a "divided" value between the comparator output and GND, creating a variable trip point. Open the simulation below to change the resistor ratio and see how the trip points move!

<img src="res/hysteretic-comparator.png" height=300 />

_Hysteretic comparator with trip points at ±6V. [[Falstad]](https://tinyurl.com/2k9ohuqg)_

As you might guess, because the voltage divider is symmetric about 0V, changing the resistor values will shift where this trip point occurs on both the "high" and "low" sides. These thresholds form a "hysteretic window", which are the transition threshold values that the input must reach in order to "trip" the comparator output to the other extreme. 

In addition to moving trip points nearer or further apart (expanding or collapsing the window), we can also bias the resistive divider about a point other than 0V to further alter the behavior. 

This makes the complete formula describing our trip point thresholds:

$$V_{trip,low} = V_{bias}\frac{R_{in}}{R_{in}+R_f} + V_{comp,low}\frac{R_f}{R_{in}+R_f}$$

$$V_{trip,high} = V_{bias}\frac{R_{in}}{R_{in}+R_f} + V_{comp,high}\frac{R_f}{R_{in}+R_f}$$

where
- $V_{comp} =$ comparator output voltage (at low or high, respectively)
- $R_{in} =$ input resistor (nearest the bias voltage)
- $R_f =$ feedback resistor (connecting output to input)

For example, if we set the input bias to 6V instead of 0V, then (using the formula from before) our trip points will move to:

$$V_{out,high} = 12\frac{50k}{100k} + 6\frac{50k}{100k} = 9V$$

$$V_{out,low} = -12*\frac{50k}{100k} + 6\frac{50k}{100k} = -3V$$

<img src="res/hysteretic-comp-6v-offset.png" height=300 />

_Hysteretic comparator with biased reference about 6V. [[Falstad]](https://tinyurl.com/2hyqy4zb)_

Finally, we can modify the comparator output by changing its power connections from ±12V to something else. Note that this changes $V_{comp}$ appropriately, and is yet another factor altering the trip points.

### First Oscillator: The Triangle Core

It's time to make our first oscillator! We have most of the circuit already — in fact, this is the complete schematic for a basic triangle core.

<img src="res/tri-core-falstad.png" height=300 />

_The triangle oscillator core [[Falstad]](https://tinyurl.com/2qcdnmht)_

Importantly, the above circuit uses +12V and 0V, and **not** -12V, for powering the comparator. This is important because it shifts the point of symmetry for the hysteretic window from 0V to 6V.

 You should recognize the hysteretic comparator circuit from before. As noted in the previous section, the change in power to +12V/0V alters $V_{comp,low}$ and affects the trip points. However, with the bias point set at 6V, the voltage divider in the positive feedback path is once again symmetric, so trip points are evenly spaced about 6V. 

 It may take a bit more time to process how the new circuitry creates an oscillator, but you already know everything you need to understand it.
 
 Previously, we supplied a sine input at the inverting (-) terminal, which the comparator responded to. However, we could supply any input, as long as it eventually hits the trip points that the comparator needs to change its output. (Go back and run the simulation, changing the input source to a triangle or sawtooth.) 
 
 The comparator always outputs either "high" or "low" — in this case, +12V or 0V — so we can think of it as creating a rudimentary voltage supply. If we connect this output to an RC network, we'll get a capacitor that charges and discharges according to an RC time constant, τ, proportional to the values of R and C. This looks roughly the same as a voltage source input, and therefore provides the hysteretic comparator with the ability to oscillate!
 
 You'll find that $R_{charge}$ in the feedback path sets the oscillation frequency, by limiting current flow to the capacitor. However, the trip points _also_ affect oscillation frequency by controlling how far the capacitor must charge/discharge to trip the comparator. That said, they should not be used to vary the oscillation frequency. Larger hysteretic windows cause the RC response to become more "exponential" as the trip points move closer to the comparator rails. For a better "triangle" shape that looks linear, set the trip points closer together, so that they occupy a smaller fraction of the RC charging/discharging graph. For example, charging a capacitor from 5V to 7V has an imperceptible curve and will appear perfectly linear.

 <img src="res/rc-charge-discharge.png" height=300 />

You can (and should) try to build this oscillator before moving on. Although we are using it as a stepping stone to a sawtooth oscillator, you could implement it as an oscillator to use in your final synth!

Despite the achievement of the triangle oscillator core, it does have some shortcomings. These don't make it useless — you could do a lot with it, and could certainly include it in the final build — but it does limit the final synth design.

Most notably, one of our goals for an oscillator should be to have voltage control, and the triangle oscillator makes this difficult because it both charges *and* discharges along the same resistor. Our path to voltage control will use transistors, and having a resistor with a single direction of current flow will make this upgrade possible.

In the next section, we'll turn the triangle core into a sawtooth oscillator.

### The Sawtooth Core



The sawtooth core begins with the simple circuit below, which consists of a single capacitor, a diode, a comparator, and a few resistors. Depending on component values, it can reach frequencies as low as 10-20Hz, and as high as several kHz.

<img src="res/saw-core-falstad.png" height=300 />

_[Sawtooth core [Falstad]](https://tinyurl.com/279leub2)_

We want the ability to produce sound at a variable frequency; therefore, what we want is a variable oscillator which changes output frequency over the audible range of 20Hz - 20kHz. 

The term "variable" implies that something is changing, but what? There are several options when it comes to the "variable" part of a variable oscillator:
- Component values (eg. resistors, capacitors)
- Voltage
- Current

Most music synthesizers use either component values or voltages as inputs to determine the output frequency of a signal. 


Spiked capacitor: https://tinyurl.com/24br28bv

## The Amplifier


## The Envelope Generator


## Future Work
### Additional Oscillators & Noise

### Additional Modules
> Low frequency oscillators (LFOs), white noise generators, ring modulators, and glides (portamento). LFOs are used to modulate the control voltages to the other modules (VCO, VCF, and VCA) to provide tremolo or vibrato effects. White noise generators can be used as additional signal source, usually mixed with the output of the VCO to add the whoosh of a wind instrument or the the crash of a cymbal or drum. Ring modulators are basically analog multipliers that create very weird effects, but are important for synthesizing things like bell sounds. Glides are used on the inputs of VCOs to provide trombone-like slides from note to note. [^nate]

# References
[^mit_syw]: So You Want to Build a Synthesizer MIT http://web.mit.edu/klund/www/weblatex/node2.html
[^pc_eurorack]:Perfect Circuit Eurorack https://www.perfectcircuit.com/signal/eurorack-line-level
[^nate]: Nate Hatch Fender Component Selection https://www.youtube.com/watch?v=FacBtCPez2U&t=7s
