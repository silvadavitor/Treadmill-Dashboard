# Treadmill Dashboard

A treadmill control panel built entirely from 7-segment displays, running on an **Arduino Mega 2560**. It shows five workout metrics in real time — time, distance, calories, heart rate, and speed — controlled by three push buttons plus a speed potentiometer. The logic was built visually in **Visuino**, and the whole circuit was assembled and tested in the **SimulIDE** simulator.

> Built as a college project — the goal was to reproduce the kind of dashboard you find on a real treadmill: several independent counters running at once, each on its own display.

## Tools

- Visuino `8.0.0.160`
- SimulIDE `1.1.0-SR2`

## Repository contents

| File | Description |
|------|-------------|
| `simulIDE-TRABALHO3V10.sim1` | SimulIDE circuit (displays, transistors, buttons, potentiometer) |
| `visuinoTRABALHO3V10.visuino` | Visuino project with all the logic |

### How to open

- **Circuit:** open `simulIDE-TRABALHO3V10.sim1` in SimulIDE 1.1.0-SR2.
- **Logic:** open `visuinoTRABALHO3V10.visuino` in Visuino 8.0.0.160. Visuino generates the Arduino sketch from the diagram.

## Components

- Arduino Mega 2560
- 7-segment displays: 4 digits for the timer, 3 digits each for distance, calories, heart rate, and speed
- DIP resistor arrays for the segment lines
- Transistors for digit (common) switching
- 3 push buttons (Start, Stop, Reset)
- 1 potentiometer (speed control)
- 5V source, resistors, and jumpers

## The panels

- **Time** – elapsed workout time as MM:SS (4 digits)
- **Distance** – total distance in km, shown as 00.0 (3 digits)
- **Calories** – estimated calories burned (3 digits)
- **Heart rate** – simulated BPM (3 digits)
- **Speed** – current speed in km/h, from 0 to 25 (3 digits)

## The controls

- **Start (Inicia)** – begins the session; all counters start
- **Stop (Para)** – pauses everything; pressing Start resumes
- **Reset (Zera)** – clears every display back to zero
- **Speed potentiometer** – sets the current speed

---

## How the hardware works (SimulIDE)

The displays are multiplexed. The segment lines (a–g) run from the Arduino, through a DIP resistor array, into the segment pins of every display. Each digit's common line is driven by its own transistor: when the Arduino sends a signal through a base resistor, the transistor turns on, connects that digit to ground, and lights it up. Tunnels (named nets) were used instead of long wires to keep the schematic readable.

The buttons are wired to 5V and ground. Speed uses a potentiometer wired to 5V, ground, and an analog input. Heart rate has no sensor — the value is generated randomly in Visuino. The finished layout was placed over a treadmill panel image to make it look faithful to the real thing.

## How the logic works (Visuino)

Each display is driven by a 7-segment display component, wired to the same Arduino ports used in the hardware. Six components were used: one per panel, plus an extra one for the timer (split into minutes and seconds).

**Timer.** A Pulse Generator fires once per second into a Counter (0–59, seconds). Its *Max Reached* output clocks a second Counter (0–59, minutes). Each counter drives its own display → MM:SS.

**Speed.** The potentiometer's analog input passes through a Multiply block (×0.25) and goes to the speed display, giving the 0–25 km/h reading. This panel uses the *analog* 7-segment component, since speed is a float.

**Heart rate.** A Pulse Generator clocks a Random Integer Generator (60–120), sent straight to the display.

**Distance & calories (Custom Code).** These must *accumulate* over time, so a Custom Code block handles them.

Inputs: `Velocidade`, `Zera`, `Ligado` — Outputs: `Distancia`, `Calorias`.

Global variables:

```cpp
unsigned long tempoAnterior = 0;
float distanciaAcumulada = 0.0;
float velocidadeAtual = 0.0;
bool estadoLigado = false;
```

On Data (one per input):

```cpp
// Velocidade
velocidadeAtual = AValue;

// Zera
distanciaAcumulada = 0.0;

// Ligado
estadoLigado = AValue;
```

On Execute:

```cpp
unsigned long tempoAtual = millis();
float deltaTempo = (tempoAtual - tempoAnterior) / 1000.0;
tempoAnterior = tempoAtual;

float vel_ms = 0.0;
if (estadoLigado) {
    vel_ms = velocidadeAtual / 3.6;   // km/h -> m/s
}

distanciaAcumulada += vel_ms * deltaTempo;

Distancia.Send((long)distanciaAcumulada);
Calorias.Send((long)(distanciaAcumulada * 7.0));
```

The math, briefly:

- **Delta time** — the block isn't clocked at a fixed rate, so `millis()` is used to measure exactly how much time passed since the last run.
- **÷ 3.6** — converts km/h to m/s (1 km/h = 1000 m / 3600 s), so `m/s × seconds = meters`.
- **`estadoLigado` check** — while paused, speed is forced to 0 so nothing accumulates.
- **× 7** — a rough calories-per-distance factor for a ~70 kg person. Adjust for a different weight.

**Controls.** Start/Stop feed an RS flip-flop that enables/disables the 1-second Pulse Generator and sets the `Ligado` flag — one latch starts and stops the whole panel. Reset zeroes the counters and the Custom Code's accumulated values.

## Behavior

On start, everything reads zero except the speed potentiometer. Press **Start** and the counters begin. **Stop** freezes everything (Start resumes), and **Reset** clears every display back to zero.

---

## Authors

- **Vitor da Silva** ([@silvadavitor](https://github.com/silvadavitor))
- **Samuel José Candido** ([@SamuelCandido](https://github.com/SamuelCandido))

A special thanks to **Professor Miguel Wisintainer** for the lessons in class and for proposing a challenge we really enjoyed working on.
