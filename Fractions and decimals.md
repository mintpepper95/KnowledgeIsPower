
When we talk about numbers less than 1 (like `0.5` or `0.1`), we are forced to use fractions because **a fraction is simply the mathematical definition of a piece of a whole.** Without fractions, you only have integers ($... -2, -1, 0, 1, 2 ...$). The moment you need to measure half a cup of water, 15 minutes of an hour, or 10 cents of a dollar, you have to break that single whole unit ("1") into smaller pieces.

The real magic—and the reason it gets confusing—is how we write those fractions down using **positional notation**.

## The Trick of Positional Notation

We don't usually write "I have $\frac{75}{100}$ of a dollar." We write `$0.75`.

The decimal point is a boundary marker. Every step you take to the **left** of the decimal point multiplies the value by 10. Every step you take to the **right** divides the value by 10.

When you write the number `0.35`, you are actually writing a shorthand addition of fractions:

$$\frac{3}{10} + \frac{5}{100} = \frac{35}{100}$$

Every digit to the right of the dot is fundamentally a fraction where the denominator is a power of 10 ($10, 100, 1000$, etc.).

In a computer, it works the exact same way, but the steps divide by 2 instead of 10. The binary number `0.11` means:

$$\frac{1}{2} + \frac{1}{4} = \frac{3}{4} \text{ (or } 0.75 \text{ in base-10)}$$

## Why use decimals instead of just writing raw fractions like $\frac{3}{4}$?

If fractions are what these numbers actually are, why do both humans and computers bother converting them into decimals (`0.75`) or binary?

### 1. Universal Arithmetic (Math is way easier)

Imagine trying to calculate $\frac{1}{3} + \frac{2}{7} - \frac{5}{12}$ in your head. You have to find a lowest common denominator (which is 84), convert every single fraction, perform the math, and then reduce it. It's incredibly slow.

With positional notation (decimals), every single number shares a predictable, standardized denominator base. Because of this, adding decimals works _exactly_ like adding whole numbers. You just line up the dots and carry the one:

Plaintext

```
  0.333
+ 0.285
-------
  0.618
```

Computers love this. Hardware can process positional numbers incredibly fast because the rules for adding `1` and `0` never change, no matter where they are relative to the decimal point.

### 2. Infinite Precision

Raw fractions are rigid. If you want to express a highly precise scientific measurement, a fraction like $\frac{141592}{100000}$ is ugly. Positional notation allows you to just keep writing digits to the right forever ($\pi = 3.141592...$) to achieve whatever level of precision you need.

## In short...

We use fractions because nature is continuous, not discrete; things exist between the whole numbers. We use _positional decimals/binary_ to write those fractions down because it makes doing math infinitely faster and simpler for both human brains and silicon chips.

Does this distinction between raw fractions and positional notation make sense, or would you like to look closer at how computers handle the trade-off between speed and precision?