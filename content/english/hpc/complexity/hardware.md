---
title: Modern Hardware
weight: -1
ignoreIndexing: true
---

The main disadvantage of the supercomputers of 1960s wasn't that they were slow — relatively speaking, they weren't — but that they were giant, complex to use, and so expensive that only the governments of world superpowers could afford them. Their size was in fact the reason they were so expensive: they required a lot of custom components that had to be very carefully assembled in the macro-world by people holding advanced degrees in electrical engineering, in a process that couldn't be up-scaled for mass production.

The tipping point was the development of *microchips* — single, tiny, complete circuits — which revolutionized the industry and turned out to be probably the most important invention of the 20th century. What was a multimillion-dollar cupboard of computing machinery in 1965 could in 1975 fit on a [4×4 mm slice of silicon](https://en.wikipedia.org/wiki/MOS_Technology_6502)[^size] that you can buy for $25. This dramatic improvement in affordability started the home computer revolution during the following decade, with computers like Apple II, Atari 2600, Commodore 64 and IBM PC becoming available to the masses.

[^size]: Actual sizes of CPUs are about centimeter-scale because of power management, heat dissipation, and the need to plug it into the motherboard without excessive swearing.

### Fabrication Process

Microchips are "printed" on a slice of crystalline silicon using a process called [photolithography](https://en.wikipedia.org/wiki/Photolithography), which involves

1. growing and slicing a [very pure silicon crystal](https://en.wikipedia.org/wiki/Wafer_(electronics)),
2. covering it with a layer of [substance that dissolves when photons hit it](https://en.wikipedia.org/wiki/Photoresist),
3. hitting it with photons in a set pattern,
4. chemically [etching](https://en.wikipedia.org/wiki/Etching_(microfabrication)) the now exposed parts,
5. removing the remaining photoresist,

…and then performing another 40-50 steps over the span of several months to complete the rest of the CPU.

![](../img/lithography.png)

Consider now the "hit it with photons" part. For that, we can use a system of lenses that projects a pattern onto a much smaller area, effectively making a tiny circuit with all the desired properties. This way, the optics of the 1970s were able to fit a few thousand transistors on the size of a fingernail, which gives microchips several key advantages that macro-world computers didn't have:

- higher clock rates (that were previously limited by the speed of light);
- ability to scale the production;
- much lower material and power usage, translating to much lower cost per unit.

Apart from these immediate benefits, photolithography enabled a clear path to improve performance further: you can just make lenses stronger, which in turn will create identically smaller devices with relatively little effort.

### Dennard Scaling

Consider what happens when we scale a microchip down. A smaller, but nearly identical circuit requires proportionally less materials, and smaller transistors take less time to switch (along with all other physical processes in the chip), allowing reducing the voltage and increasing the clock rate.

A more detailed observation, known as the *Dennard scaling*, states that reducing transistor dimensions by 30%

- doubles the transistor density ($0.7^2 \approx 0.5$),
- increases the clock speed by 40% ($\frac{1}{0.7} \approx 1.4$),
- and leaves the overall *power density* the same.

Since the per-unit manufacturing cost is a function of area, and the exploitation cost is mostly the cost of power[^power], each new "generation" should have roughly the same total cost, but 40% higher clock and twice as many transistors, which can be promptly used, for example, to add new instructions or increase the word size — to keep up with the same miniaturization happening in memory microchips.

[^power]: The cost of electricity for running a busy server for 2-3 years roughly equals the cost of making the chip itself.

Due to the trade-offs between energy and performance you can make during the design, the fidelity of the fabrication process itself, such as "180nm" or "65nm", directly translating to the density of transistors, became the trademark for CPU efficiency.

<!--
[^fidelity]: To avoid upsetting and confusing consumers with this new reality, chip makers agreed to stop delineating their chips by the size of their components. A special committee had a meeting every two years where they took the previous node name, divided by the square root of two, rounded to the nearest integer, declared that result to be the new node name, and then drank lots of wine. Chip makers now group components into generations by the time that they where made, and so "nm" doesn't mean nanometer anymore.
-->

Throughout most of the computing history, optical shrinking was the main driving force behind performance improvements. Gordon Moore, the former CEO of Intel, made a prediction in 1975 that the transistor count in microprocessors will double every two years. His prediction held to this day and became known as *Moore's law*.

![](../img/dennard.ppm)

Both Dennard scaling and Moore's law are not actual laws of physics, but just observations made by savvy engineers. They are both destined to stop at some point due to fundamental physical limitations, the ultimate one being the size of silicon atoms. In fact, Dennard scaling already did — due to power issues.

Thermodynamically, a computer is just a very efficient device for converting electrical power into heat. This heat eventually needs to be removed, and there are physical limits to how much power you can dissipate from a millimeter-scale crystal. Computer engineers, aiming to maximize performance, essentially just choose the maximum possible clock rate so that the overall power consumption stays the same. If transistors become smaller, they have less capacity, meaning less required voltage to flip them, which in turn allows increasing the clock rate.

Around 2005–2007, this strategy stopped working because of *leakage* effects: the circuit features became so small that their magnetic fields started to make the electrons in the neighboring circuitry move in directions they are not supposed to, causing unnecessary heating and occasional bit flipping.

The only way to mitigate this is to increase voltage, and to balance off power consumption you need to reduce clock frequency, which makes the whole process progressively less profitable as transistor density increases. At some point, clock rates could no longer be increased by scaling, and then miniaturization trend started to slow down.

<!--

### Power Efficiency

It may come as a surprise, but the primary metric for modern CPUs is not the clock frequency, but rather "useful operations per joule", or, more practically put, "useful operations per dollar".

Thermodynamically, a computer is just a very efficient device for converting electrical power into heat. This heat eventually needs to be removed, and it's not straightforward to do when you are working with a millimeter-scale crystal. There are physical limits of how much power you can consume and then dissipate.

Historically, the three main variables guiding microchip designs are power, performance and area (PPA), commonly defined in watts, hertz and nanometers. Until ~2005, cost, which was mainly a function of area, and performance, used to be the most important criteria. But as battery-driven mobile devices started replacing PCs, power quickly and firmly moved up on top of the list, followed by cost and performance.

Leakage: interfering magnetic fields make electrons move in the directions they are not supposed to and cause unnecessary heating. It isn't bad by itself: it mitigate it you need to increase the voltage, and it won't flick any bits. But the problem is that the smaller a circuit is, the harder it is to cope with this by isolating the wires. So modern chips keep the clock frequency at a level that won't cause overheat, although physically there aren't other reasons why they shouldn't.

-->

### Modern Computing

Dennard scaling ended, but Moore's law has not died yet.

Clock rates plateaued, but the transistor count is still increasing, allowing for creation of new, *parallel* hardware. Instead of chasing faster cycles, CPU designs started to focus on getting more useful things done in a single cycle. Instead of getting smaller, transistors have been changing shape.

This resulted in increasingly complex architectures capable of doing dozens, hundreds or thousands of different things every cycle. For modern computers, the "let's count all operations" approach for predicting algorithm performance isn't just slightly wrong, but is off by several orders of magnitude.

![Die shot of a Zen CPU microarchitecture by AMD. It has 1,400,000,000 transistors per core](../img/die-shot.jpg)

Theoretical Computer Science is now what's called a dead field: really exciting things stopped happening in the 70s; everything past that are just attempts to replace logarithms in the asymptotic with something slightly less than logarithms. If 50 years ago such algorithms had hope that eventually there will be enough computing power to process the large datasets for which they beat their asymptotically inferior, but practical counterparts, nowadays we know for a fact that they never will.

This is what this book is about: accepting the reality and optimizing for the hardware you have, beyond just asymptotic complexity. You will probably not learn a single asymptotically faster algorithm here, but you will learn how to squeeze performance from all of non-exponentially-increasing transistors you have, which is a far more impactful skill.