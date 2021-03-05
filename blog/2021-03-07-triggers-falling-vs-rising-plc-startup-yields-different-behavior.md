---
title: Triggers, falling vs rising and PLC startup yields different behavior
path: /triggers-falling-vs-rising-plc-startup-yields-different-behavior
date: 2021-03-04
summary: The trigger output behavior of the R_TRIG vs F_TRIG is different on the first PLC cycle. During normal operation they are inversely identical.
tags: ['beckhoff', 'twincat', 'fb', 'software']
---

## Introduction
Triggers are one of the basic building blocks for any PLC and even general programing. It allows you to run a function or routine once on a condition change. For example sending an alarm to the HMI or sending an event to a database. For a trigger to work it requires a change in condition or an edge. This is the reason for calling it an edge detector.

They can be either falling edge triggers or rising edge triggers. Beckhoff has both built in as [`F_TRIG`](https://infosys.beckhoff.com/content/1033/tcplclib_tc2_standard/9007199329131019.html?id=7995687441303784270) and [`R_TRIG`](https://infosys.beckhoff.com/content/1033/tcplclib_tc2_standard/9007199329132555.html?id=4560815639834665883) function blocks.

## Comparison
One may expect that a negated input to an `F_TRIG` would yield the same result as an `R_TRIG` and vice versa. As expected the following will yield the same results with `rcnt = fcnt`.

```typescript
rtrig : R_TRIG;
ftrig : F_TRIG;
in1   : BOOL := FALSE;
rcnt  : INT;
fcnt  : INT;


rtrig(CLK := in1);
IF rtrig.Q THEN
    rcnt := rcnt + 1;
END_IF

ftrig(CLK := NOT in1);
IF ftrig.Q THEN
    fcnt := fcnt + 1;
END_IF
```

We can also look at this in the opposite way, this being a falling edge detector. It will also yield the same results.

```typescript
rtrig2 : R_TRIG;
ftrig2 : F_TRIG;
in2   : BOOL := TRUE;
rcnt2  : INT;
fcnt2  : INT;


rtrig2(CLK := NOT in2);
IF rtrig2.Q THEN
    rcnt2 := rcnt2 + 1;
END_IF

ftrig2(CLK := in2);
IF ftrig2.Q THEN
    fcnt2 := fcnt2 + 1;
END_IF
```

## PLC Startup
Now, as I've mentioned above, a trigger requires an edge to work correctly. What happens on the first PLC scan after starting up the PLC and the input is already at the resulting edge logic to normally create a trigger? This is not a common occurrence, but it can happen. For example after a power outage.

Looking at the result from the above code gives the expected result. However if a small change is made the behavior is no longer the same. The change below has the initial values of `in1` and `in2` swapped.

```typescript
rtrig : R_TRIG;
ftrig : F_TRIG;
in1   : BOOL := TRUE;
rcnt  : INT;
fcnt  : INT;

rtrig2 : R_TRIG;
ftrig2 : F_TRIG;
in2   : BOOL := FALSE;
rcnt2  : INT;
fcnt2  : INT;


rtrig(CLK := in1);
IF rtrig.Q THEN
    rcnt := rcnt + 1;
END_IF

ftrig(CLK := NOT in1);
IF ftrig.Q THEN
    fcnt := fcnt + 1;
END_IF

rtrig2(CLK := NOT in2);
IF rtrig2.Q THEN
    rcnt2 := rcnt2 + 1;
END_IF

ftrig2(CLK := in2);
IF ftrig2.Q THEN
    fcnt2 := fcnt2 + 1;
END_IF
```

After the first PLC cycle `rcnt1 = rcnt2 = 1` whereas `fcnt1 = fcnt2 = 0`.

## Conclusion
During normal operation the behavior of `R_TRIG` and `F_TRIG` are interchangeable by inverting the logic of the input. However this is no longer true during the first PLC cycle.

On a first PLC cycle if the input to a `R_TRIG` is true, the `R_TRIG` will consider that being an edge and set its output `Q` to true for that first cycle.

On a first PLC cycle if the input to a `F_TRIG` is false, the `F_TRIG` will NOT consider that being an edge and it wil keep its output `Q` set to false.

One common use of triggers is for errors or warnings. When integrating an external device there is normally an error signal. This signal can be NC or NO. In the NO variant a `R_TRIG` would be used to catch the error. In an NC variant a `F_TRIG` would be the default.

For example given a device with a NC error signal connected to a PLC. After a power outage the device starts up faster than the PLC (this is common) and sets its error output to false (meaning there is an error). Using and `F_TRIG` once the PLC starts, it would not trigger and would not take the necessary steps to warn the operator resulting in troubleshooting downtime. Whereas if a `R_TRIG` had been used with an inverted input, the operator would have been correctly warned.

This examples makes assumptions on the structure of error reporting. However the fact remains that the way `R_TRIG` and `F_TRIG` work are not inversely identical on the first PLC cycle.

This behavior was tested on builds 4022.32 and 4024.12 with the exact same results. I don't know how similar triggers work on different PLCs, but be weary of how your triggers work on the first PLC scan!