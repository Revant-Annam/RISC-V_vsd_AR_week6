# Sky130 Day 4 - Pre-layout timing analysis and importance of good clock tree

## Introduction to Delay Tables

**Power Aware CTS**: Setting the AND gate enable pin to logic '1' propagates the clock; logic '0' blocks it. Similarly, OR gate propagates with enable at logic '0' and blocks at logic '1'. This blocking 
capability significantly reduces clock tree power consumption.

<img width="701" height="425" alt="image" src="https://github.com/user-attachments/assets/23546ce2-b0f0-4924-9658-001a577ec55f" />

Consider a clock tree where the first buffer drives two second-level buffers. We've split the buffer and in clock gating, replaced the buffer with an AND gate. Will other characteristics remain unchanged?

<img width="1081" height="395" alt="image" src="https://github.com/user-attachments/assets/867761fc-66b1-4096-9156-fc95f23cafcc" />

Before buffer-to-gate replacement, we made these assumptions:

Assume c1=c2=c3=c4=25fF
Assume Cbuf1=Cbuf2=30fF
Total Cap at node 'A'=> 60fF
Total Cap at node 'B'=> 50fF
Total Cap at node 'C'=> 50fF

<img width="397" height="372" alt="image" src="https://github.com/user-attachments/assets/431f6c74-cc0b-4ffe-b8c3-517dfe19a3da" />

Observations:
- 2 buffering levels
- Each node at every level drives identical load
- Identical buffers at same level

Buffer output capacitance across the circuit isn't constant—load varies, causing input transition variation.

This input/output variation creates delay variety, captured through delay tables.

**Delay Table Preparation**:

Extract one buffer from the circuit and vary its input transition within a range (10ps-100ps), while output load also varies. Characterize the delay and tabulate the data.

### Delay Table Usage Part 1

Consider other buffer examples.

<img width="1237" height="317" alt="image" src="https://github.com/user-attachments/assets/962f3a4b-aba7-4555-a2f0-ad65f95a1dc4" />

For a practical case: buffer1 with 40ps input slew and 60fF output capacitance produces delay between x9-x10. Values absent from delay tables are extrapolated from available data.

### Delay Table Usage Part 2

Calculate buffer 2 delay, then determine latency at 4 clock endpoints. Both buffers share input transition. Assuming approximately 60ps input slew and 50fF load at both buffers yields y15 delay.

Total input-to-output delay = x9' + y15 (ignoring wire delay), resulting in zero skew at any output point. Unequal node loads produce non-zero skew.

## Timing Analysis with Ideal Clocks Using OpenSTA

### Setup Timing Analysis and Introduction to Flip-flop Setup Time

**Timing Analysis (with ideal clock)**: Begin setup analysis with ideal clock (single clock). Clock specifications:

clock frequency = 1 GHz
clock period = 1 ns

Analyze between '0' and 'T' clock period. Edge sent to launch flop at '0' clock period; at T=1ns, second edge reaches capture flop.

With combinational delay theta, setup timing analysis requires this combinational delay be less than T for proper system operation.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/834858e3-195c-4246-99bd-a19e69b6b144" />

Opening the capture flop reveals combinational circuitry with multiple MOSFETs, logic elements, resistances, and capacitances, with corresponding time graphs.

<img width="1147" height="407" alt="image" src="https://github.com/user-attachments/assets/ce1fe1d4-45db-4c65-a2d9-8342d24a569d" />

Clock logic '0' or '1' causes MUX1 and MUX2 delays to restrict or affect combinational delay requirements.

Finite time required for D input settling is called SETUP TIME.

Therefore, finite time 's' required before clock edge for 'D' to reach Qm.

MUX1 internal delay = setup time (S).

Thus, θ<T becomes θ<(T-S).

<img width="1122" height="317" alt="image" src="https://github.com/user-attachments/assets/caff9a53-890f-4d25-828f-3a10042c5073" />

### Introduction to Clock Jitter and Uncertainty

Jitter occurs because PLLs (phase-locked loops) create clocks, and the clock source should send signals exactly at 0, T, 2T... However, inherent variations prevent exact timing—this is jitter. Jitter represents
temporary clock pulse variation.

Considering uncertainty time (US), the equation becomes θ<(T-S-US). Assuming 'S'=0.01ns and 'US'=0.09ns, identify the timing path in our circuit where stage 1 and stage 3 logic paths have single clocks.

Identify combinational path delay for both logic paths.

<img width="1198" height="636" alt="image" src="https://github.com/user-attachments/assets/637295d2-fc83-4a1d-8029-e94b7254a8fa" />

<img width="1102" height="668" alt="image" src="https://github.com/user-attachments/assets/9e75e9ed-5e03-45dc-b54c-726c9bba5f5b" />

---

## Clock tree synthesis and signal integrity

### Clock tree routing and buffering using H-Tree algorithm

If we simply connect a clock signal with a single wire to multiple flip-flops, the physical wire length from the clock source to each flip-flop will be different. This means the clock arrival time will also be different. For example, the time to reach $FF1$ ($t1$) will be less than the time to reach $FF2$ ($t2$).

This difference in arrival times is called **clock skew** ($\text{Skew} = t2 - t1$), which we ideally want to be zero.

A smarter routing approach, like an **H-Tree**, solves this. Instead of a direct connection, the clock is routed to a central midpoint. From there, it splits and routes to other midpoints, fanning out in a balanced, tree-like structure. This ensures the clock signal travels a similar physical distance to every flip-flop, making the arrival times nearly identical and minimizing skew.

### Clock tree synthesis (Buffering)

Beyond just balancing path lengths, we must also manage the **signal quality**. A long wire acts as an RC network (resistor-capacitor network). As the clock signal travels down this wire, its waveform is degraded—the sharp edges become slow and sloped.

To resolve this, we insert **repeaters (buffers or inverters)** at intervals along the clock path. These repeaters effectively regenerate the signal, ensuring a sharp waveform is delivered to the endpoints.

The key difference for clock repeaters, compared to standard data path buffers, is that they must be designed to have **equal rise and fall times**. This is critical to prevent distorting the clock's duty cycle (the 50/50 split between high and low). We insert as many repeaters as needed to maintain signal integrity across the entire tree.

### Crosstalk and clock net shielding

Even after we build a perfectly balanced tree with zero skew, the clock net is still vulnerable. The clock is a **critical net** and must be protected from noise generated by neighboring wires.

When a nearby aggressor wire switches, the **coupling capacitance** between it and our victim clock net can cause significant problems. This interference, known as **crosstalk**, can lead to two issues:

1.  **Glitch:** An unwanted, false pulse on the clock line.
2.  **Delta Delay:** The clock signal is either sped up or slowed down by the aggressor's switching.

The biggest impact of delta delay is that it **makes our carefully balanced skew non-zero** again, which can cause timing failures.

**Clock Net Shielding** is the technique used to prevent this. We protect the clock net by placing dedicated "shield" wires parallel to it, often on both sides. These shield wires are connected to a stable voltage (VDD or GND) and **do not switch**. This shielding wire effectively breaks the coupling capacitance between the aggressor and the victim, isolating the clock net and preserving its signal integrity.
