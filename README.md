# 2-Bit Arithmetic Logic Unit with 4 Operations

<image src="picture.jpeg" width="800px"/>

## 1. Description

This 2-bit ALU (Arithmetic Logic Unit) is built using 74-series logic gates and multiplexers, and performs four basic operations: Logical AND, Logical OR, Addition, and positive-result Subtraction, on two 2-bit input words (A1A0 and B1B0). It uses a 2-bit opcode (I1I0) to select the operation, and displays the result as a 3-bit output (R2R1R0). Logical AND and OR operate only on the least significant bits (A0 and B0), with the result being displayed on R0. Arithmetic operations use the full words on input and output, and the output R2 is reserved for indicating overflow during addition.

## 2. Block Schematic
<image src="block.png" width="600px"/>

## 3. Logical Schematic and Simulation
Logisim Evolution was used for the prototyping of the circuit and for simulating its functionality. (file provided in repository)
<image src="logical.png" width="1000px"/>

## 4. Electrical Schematic and PCB
The project’s schematic and PCB were designed in KiCAD. (files provided in repository)
<image src="schematic.png" width="1000px"/>
<image src="pcb.png" width="800px"/>

## 5. Operation
The ALU takes two 2-bit words (A1A0 and B1B0) as input for its operations, and one 2-bit word (I1I0) for selecting the opcode. The results of the operations are displayed using LEDs in the form of a 3-bit word (R2R1R0). The operations and their opcodes are as follows:
<br>
<br>Logical AND		00
<br>Logical OR		01
<br>Addition		10
<br>Subtraction		11
<br>
<br>For the logical operations, only the least significant bits of the inputs - A0 and B0 - are operated upon, and the result is displayed on the result bit R0. The arithmetic operations employ the full 2-bit input word as well as two output bits, R1 and R0, while the third, R2, is reserved for displaying overflows from summations, and will remain LOW otherwise.

## 6. In-depth analysis of functionality
The general idea is that the input words A and B are forwarded to the operation-performing combinatorial logic gate circuits. Several multiplexers then select from among the results of the gate arrays, and based on the instruction opcode, the result of an operation is displayed. There are two arrays of multiplexers, corresponding to the 2-bit instruction word - I1 selects the arithmetic operations if 1, or the logic operations if 0. I0 further differentiates between the AND/OR, and SUM/DIFF operations.
<br>The logic unit is quite simple, being composed of just an AND and an OR gate, handling the AND and OR operations, respectively. Both gates are active at the same time - the desired result is chosen using a 2-input multiplexer.
<br>The arithmetic unit uses a 2-bit carry-lookahead adder (CLA for short) to perform addition and subtraction. The CLA adder works by taking 5 inputs - one for the carry-in of the summation of the LSB digits, as well as 4 more inputs consisting of the two 2-bit input words, A and B. The adder then outputs bits labeled to be the “sum” of each pair digits.

#### Timing considerations and adders
Generally, addition can be implemented with full adders, which work by taking as inputs two operators A, B, and a third input for the carry-in of the summation, and give as outputs one bit as the sum of the digits, and a second, which acts as the carry for a cascading summation. One full adder adds together one pair of digits, and many can be put in a cascading connection to produce a ripple-carry adder.
<br>Initially, the design of this ALU employed, for its arithmetic unit, two full adders with ripple carry, rather than a CLA adder. Upon further analysis, however, it was determined that the two full adders, which operated without any synchronicity signal from a clock (the only way of avoiding timing issues with such a design), would have produced wrong results due to timing issues caused by propagation delays inherent in the logic gates. A pseudo-timing diagram follows:
<image src="logi_wrong.png" width="1000px"/>
As can be seen from the schematic, the ripple-carry is sent from the first adder when the gate U4C fires its output, namely, in the 5th time period. This carry is meant to arrive at gate U3D – however, U3D fires in the 4th period, just before U4C, from which it is meant to receive its input. This is an obvious problem – U3D’s output will not account for the carry it was supposed to receive, since gate U4C had no time to send its output just yet.
<image src="logi_cla.png" width="1000px"/>
For a carry lookahead adder, however, no such issue exists – instead of waiting for the adders of less significant digits to finish their execution and send a ripple carry, the CLA adder uses extra logic circuitry that determines, based on the digits in the input words, whether a carry will be generated, before the adders dealing with said digits finish their execution. This absolves each individual adder of their dependency upon previous adder stages, allowing every individual pair to be summed up at the same time as, and without having to wait for, the others.
<br>The design principle of the CLA adder is the avoidance of successive propagation, by predicting whether an operation will give a carry, before the operation occurs in the first place. This is done by introducing extra logic gates into the design, accounting for all possible situations that could result in a stage receiving a carry. Whether one of the adder stages in a CLA adder receives a carry depends on whether a previous stage has either generated a carry by itself, or propagated one from a stage before it. This ALU implements a 2-bit CLA adder, so the situations that it accounts for w.r.t. each carry are:

Carry-out 0:
- do A0 and B0 generate a carry?
- is there a carry (generated externally) AND A0 and B0 propagate it?

Carry-out 1:
- do A1 and B1 generate a carry?
- is there a carry (generated by A0 and B0) AND A1 and B1 propagate it?
- is there a carry (generated externally) AND A0 and B0 propagate it AND A1 and B1 propagate it?

To further illustrate the principle, if we had to implement a 3-bit CLA adder, we would also have accounted for:

Carry-out 2:
- do A2 and B2 generate a carry?
- is there a carry (generated by A1 and B1) AND A2 and B2 propagate it?
- is there a carry (generated by A0 and B0) AND A1 and B1 propagate it AND A2 and B2 propagate it?
- is there a carry (generated externally) AND A0 and B0 propagate it AND A1 and B1 propagate it AND A2 and B2 propagate it?

... and so on, recursively, for any increase in width.
<br>
<br>A further consideration regarding timing issues would be the propagation delays caused by the lengths of the PCB route traces – in high frequency circuits, the signals must arrive at the same time to their destinations. However, this ALU was implemented without clock timing in mind, and as such, the PCB route lengths do not matter, because given enough time, all signals will arrive, and the output will eventually be displayed correctly.

#### Addition
In the addition mode, the first adder stage is a simple full adder, which starts with a carry-in of zero and adds the digits as normal. The “sum” output is sent to the result LED output R0, while the carry is used with look-ahead in mind for the addition of the second pair of digits.
<br>The second adder will similarly send the sum to the LED output on R1, but the process is slightly modified, and also, the carry-out of this stage will have accounted for the carry-out of the first adder, by checking all possible situations. 
<br>Since there is no third adder for the second carry-out to cascade to, any potential overflow will need to be displayed in order to produce the accurate result of the summation. The output R2 has been designated for this, and for all other situations, a circuit using two logic gates has been put in place to keep the state of R2 low.

#### Subtraction
Binary subtraction is performed using the two’s complement summation. This requires computing the two’s complement of the B input, which is done by inverting the digits of B and adding 1. For instance, for a B = 01, the two’s complement is obtained by inverting B, giving us “10”, and adding 1. Thus we have: 10 + 01 = 11.
<br>The inversion of B can be done by applying inverters to each of B’s digits, this being implemented using NAND gates that are tied to B’s normal values. Seeing as addition and subtraction require the normal and the inverted values of B respectively, two multiplexers are used to select from among them, depending on the desired operation.
<br>In the case of 2-bit operands, binary subtraction of A and the two’s complement of B therefore means:
<br>
<br>A1 A0 +
<br>B1’B0’+
<br>0  1
<br>
<br>With the normal inputs of A, as well as the inverted B which is selectable by the multiplexers, all that remains is providing an additional 1 to the summation. This can be conveniently done by passing the carry-in to the addition of A0 and B0 as 1 whenever subtraction is chosen as the operation; the carry-in would otherwise remain 0 when addition is performed. It so happens that the instruction bit I0 behaves exactly in this manner, allowing us to use it as the carry-in to the first adder stage.
<br>It should be noted that the subtraction operation comes with some inherent caveats due to its employment of the two’s complement. For instance, without the logic gates that condition the output R2 solely to display the overflow of addition operations, the subtraction operation - itself employing summations, potentially overflowing - would also display overflows on the pin. The logic gates fix this, and the outputs on the other two LEDs are otherwise correct, if no overflow is displayed on R2.
<br>Another functional incompleteness is found when performing subtractions that would give negative results. For instance, the operation 00 - 11 gives the result 100. Even without accounting for the output on R2, we can see how this is actually an integer overflow, arising as the result of 111 - 11. So, because we have no way of converting such overflows, nor otherwise handling signed integers, subtraction operations resulting in negative-signed results are completely unsupported by the circuit and will give an incorrect result.


## 7. Bill of Materials
SMD ICs:
- NAND:	1x MM74HC00M (SO-14)
- AND:	2x MM74HC08M (SO-14)
- OR:	1x MM74HC32M (SO-14)
- XOR:	1x MM74HC86M (SO-14)
- MUX:	2x SN74LS157D (SO-16)

Other components, THT:
- 3x LEDs
- 3x 470Ω resistors
- 8x 1kΩ resistors
- 1x 10uF electrolytic capacitor
- 1x 10nF ceramic capacitor
- 1x 2-pin female socket connector