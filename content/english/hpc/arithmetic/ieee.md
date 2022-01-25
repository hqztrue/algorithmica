---
title: IEEE 754 Floats
weight: 2
---

When we designed our [DIY floating-point type](../float), we omitted quite a lot of important little details:

- How many bits do we dedicate for the mantissa and the exponent?
- Does a "0" sign bit mean "+", or is it the other way around?
- How are these bits stored in memory?
- How do we represent 0?
- How exactly does rounding happen?
- What happens if we divide by zero?
- What happens if we take the square root of a negative number?
- What happens if we increment the largest representable number?
- Can we somehow detect if one of the above three happened?

Most of the early computers didn't have floating-point arithmetic, and when vendors started adding floating-point coprocessors, they had slightly different visions for what answers to those questions should be. Diverse implementations made it difficult to use floating-point arithmetic reliably and portably — particularly for people developing compilers.

In 1985, the Institute of Electrical and Electronics Engineers published a standard (called [IEEE 754](https://en.wikipedia.org/wiki/IEEE_754)) that provided a formal specification of how floating-point numbers should work, which was quickly adopted by the vendors and is now used in virtually all general-purpose computers.

## Float Formats

Similar to our handmade float implementation, hardware floats use one bit for sign and a variable number of bits for the exponent and the mantissa parts. For example, the standard 32-bit `float` encoding uses the first (highest) bit for sign, the next 8 bits for the exponent, and the 23 remaining bits for the mantissa. 

![](../img/float.svg)

One of the reasons why they are stored in this exact order is so that it would be easier to compare and sort them: you can simply use largely the same comparator circuit as for [unsigned integers](../integer) — except for maybe flipping the bits in the case of negative numbers.

IEEE 754 and a few consequent standards define not one, but *several* representations that differ in sizes, most notably:

|      Type | Sign | Exponent | Mantissa | Total bits | Approx. decimal digits |
|----------:|------|----------|----------|------------|------------------------|
|    single | 1    | 8        | 23       | 32         | ~7.2                   |
|    double | 1    | 11       | 52       | 64         | ~15.9                  |
|      half | 1    | 5        | 10       | 16         | ~3.3                   |
|  extended | 1    | 15       | 64       | 80         | ~19.2                  |
| quadruple | 1    | 15       | 112      | 128        | ~34.0                  |
|  bfloat16 | 1    | 8        | 7        | 16         | ~2.3                   |

Their availability ranges from chip to chip:

- Most CPUs support single- and double-precision — which is what `float` and `double` types refer to in C.
- Extended formats are exclusive to x86, and are available in C as the `long double` type, which falls back to double precision on arm. The choice of 64 bits for mantissa is so that every `long long` integer can be represented exactly. There is also a 40-bit format that similarly allocates 32 mantissa bits.
- Quadruple as well as the 256-bit "octuple" formats are only used for specific scientific computations and are not supported by general-purpose hardware.
- Half-precision arithmetic only supports a small subset of operations and is generally used for machine learning applications, especially neural networks, because they tend to do a large amount of calculation, but don't require a high level of precision.
- Half-precision is being gradually replaced by bfloat, which trades off 3 mantissa bits to have the same range as single-precision, enabling interoperability with it. It is mostly being adopted by specialized hardware: TPUs, FGPAs, and GPUs. The name stands for "[Brain](https://en.wikipedia.org/wiki/Google_Brain) float".

Lower precision types need less memory bandwidth to move them around and usually take fewer cycles to operate on (e. g. the division instruction may take $x$, $y$, or $z$ cycles depending on the type), which is why they are preferred when error tolerance allows it.

Deep learning, emerging as a very popular and computationally-intensive field, created a huge demand for low-precision matrix multiplication, which led to manufacturers developing separate hardware or at least adding specialized instructions that support these types of computations — most notably, Google developing a custom chip called TPU (*tensor processing unit*) that specializes on multiplying 128-by-128 bfloat matrices, and NVIDIA adding "tensor cores", capable of performing 4-by-4 matrix multiplication in one go, to all their newer GPUs.

Apart from their sizes, most of the behavior is exactly the same between all floating-point types, which we will now clarify.

## Handling Corner Cases

The default way integer arithmetic deals with corner cases such as division by zero is to crash.

Sometimes a software crash, in turn, causes a real, physical one. In 1996, the maiden flight of the [Ariane 5](https://en.wikipedia.org/wiki/Ariane_5) (the space launch vehicle that ESA uses to lift stuff into low Earth orbit) ended in [a catastrophic explosion](https://www.youtube.com/watch?v=gp_D8r-2hwk) due to the policy of aborting computation on arithmetic error, which in this case was a floating-point to integer conversion overflow, that led to the navigation system thinking that it was off course and making a large correction, eventually causing the disintegration of a $1B rocket.

There is a way to gracefully handle corner cases like these: hardware interrupts. When an exception occurs, CPU:

- interrupts the execution of a program;
- packs every all relevant information into a data structure called "interrupt vector";
- passes it to the operating system, which in turn either calls the handling code if it exists (the "try-except" block) or terminates the program otherwise.

This is a complex mechanism that deserves an article of its own, but since this is a book about performance, the only thing you need to know is that they are quite slow and not desirable in real-time systems such as navigating rockets.

### NaNs and Infinities

Floating-point arithmetic often deals with noisy, real-world data, and exceptions there are much more common than in the integer case. For this reason, the default behavior is different. Instead of crashing, the result is substituted with a special value without interrupting the executing, unless the programmer explicitly wants to.

The first type of such value is the two infinities: a positive and a negative one. They are generated if the result of an operation can't fit within the representable range, and they are treated as such in arithmetic.

$$
\begin{aligned}
   -∞ < x &< ∞
\\  ∞ + x &= ∞
\\  x ÷ ∞ &= 0
\end{aligned}
$$

What happens if we, say, divide a value by zero? Should it be a negative or a positive infinity? This case is actually unambiguous because, somewhat less intuitively, there are also two zeros: a positive and a negative one.

$$
    \frac{1}{+0} = +∞
\;\;\;\;  \frac{1}{-0} = -∞
$$

Zeros are encoded by setting all bits to zero, except for the sign bit in the negative case. Infinities are encoded by setting all their exponent bits to one and all mantissa bits to zero, with the sign bit distinguishing between positive and negative infinity.

The other type is the "not-a-number” (NaN), which is generated as the result of mathematically incorrect operations:

$$
\log(-1),\; \arccos(1.01),\; ∞ − ∞,\; −∞ + ∞,\; 0 × ∞,\; 0 ÷ 0,\; ∞ ÷ ∞
$$

There are two types of NaNs: a *signaling NaN* and a *quiet NaN*. A signaling NaN raises an exception flag, which may or may not cause immediate hardware interrupt depending on the FPU configuration, while a quiet NaN just propagates through almost every arithmetic operation, resulting in more NaNs.

Both NaNs are encoded as all their exponent set to ones and the mantissa part being everything other than all zeroes (to distinguish them from infinities).

## Further Reading

If you are so inclined, you can read the classic "[What Every Computer Scientist Should Know About Floating-Point Arithmetic](https://www.itu.dk/~sestoft/bachelor/IEEE754_article.pdf)" (1991) and [the paper introducing Grisu3](https://www.cs.tufts.edu/~nr/cs257/archive/florian-loitsch/printf.pdf), the current state-of-the art for printing floating-point numbers.