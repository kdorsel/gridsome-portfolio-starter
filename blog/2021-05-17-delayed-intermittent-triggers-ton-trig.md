---
title: Delayed, Debounced and Intermittent Timers and Triggers
path: /delayed-debounced-intermittent-timers-triggers
date: 2021-05-17
summary: 
tags: ['beckhoff', 'twincat', 'fb']
---

## Introduction
Sometimes a regular timer or trigger is not sufficient for the task. A delayed trigger may be needed to debounce a signal or flash a delayed output. How about intermittently flashing a light or siren or warning a user on the HMI? The human brain tends to eventually ignore continuous states. A blinking light or an on/off siren is more likely to alert an operator than a steady signal.

Then there is the combination of both with a delayed, but intermittent, signal. Think of a low priority warning or a timer based air blow during machine run. All of these can be done using a combination of Beckhoff's built in `TON`, `R_TRIG` and `F_TRIG` function blocks. However, it would be easier to have a function block that has all of this functionality in one neat package. Meet `TON_TRIG`.

## Description
It works with both a rising edge or a falling edge by providing an input for both. There are also two `time` inputs to specify the debounced and intermittent delay times.

> Using both rising and falling edge detection in the same FB can lead to undefined behavior!!

| Input | Description |
| ----- | ----------- |
| INR | Rising edge input |
| INF | Falling edge input |
| WT  | Debounce/work time |
| PT  | Intermittent/timer time |

Here are the outputs. It can be used as a basic trigger or timer with `QT` or `QET` respectively, but also allows all the extra functionality mentioned above.

| Output | Description |
| ------ | ----------- |
| QT  | Normal trigger. Will pulse after WT elapsed |
| QET | Normal timer. Will turn on after WT + PT elapsed |
| QDT | Delayed trigger. Will pulse after WT + PT elapsed |
| QIT | Intermittent trigger. Will pulse after WT elapsed then again after every PT elapsed |

## Implementation
```typescript
FUNCTION_BLOCK TON_TRIG
VAR_INPUT
	INR : BOOL;
	INF : BOOL := TRUE;
	WT : TIME;
	PT : TIME;
END_VAR
VAR_OUTPUT
	QT : BOOL;
	QET : BOOL;
	QDT : BOOL;
	QIT : BOOL;
END_VAR
VAR
	tm : TON;
	td : TON;
	tr : R_TRIG;
	reset : BOOL := TRUE;
END_VAR

td(IN := INR OR NOT INF, PT := WT);
tm(IN := NOT tm.Q AND td.Q, PT := PT);
tr(CLK := td.Q);

IF QDT THEN
	QDT := FALSE;
	reset := FALSE;
END_IF

IF tm.Q THEN
	IF reset THEN
		QDT := TRUE;
	END_IF
	QET := TRUE;
ELSIF NOT INR AND INF THEN
	QET := FALSE;
	reset := TRUE;
END_IF

QT := tr.Q;
QIT := QT OR tm.Q;
```

## Conclusion
It's a simple FB, but it gets the job done and is always nice to have handy when dealing with timers and triggers.

Happy coding!
