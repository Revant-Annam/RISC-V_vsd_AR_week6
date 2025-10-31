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

<img width="1297" height="650" alt="image" src="https://github.com/user-attachments/assets/5e8e5690-1e5c-43c4-ac45-516f3548134c" />

This difference in arrival times is called **clock skew** ($\text{Skew} = t2 - t1$), which we ideally want to be zero.

<img width="716" height="247" alt="image" src="https://github.com/user-attachments/assets/0250d398-e578-4d86-b05a-5c0abdf89f0b" />

A smarter routing approach, like an **H-Tree**, solves this. Instead of a direct connection, the clock is routed to a central midpoint. From there, it splits and routes to other midpoints, fanning out in a balanced, tree-like structure. This ensures the clock signal travels a similar physical distance to every flip-flop, making the arrival times nearly identical and minimizing skew.

<img width="881" height="632" alt="image" src="https://github.com/user-attachments/assets/836817d9-5007-4869-be20-ca71642ce2e1" />

### Clock tree synthesis (Buffering)

Beyond just balancing path lengths, we must also manage the **signal quality**. A long wire acts as an RC network (resistor-capacitor network). As the clock signal travels down this wire, its waveform is degraded—the sharp edges become slow and sloped.

<img width="878" height="498" alt="image" src="https://github.com/user-attachments/assets/a26df2d7-910b-47ff-b526-342f7ff99cd9" />

To resolve this, we insert **repeaters (buffers or inverters)** at intervals along the clock path. These repeaters effectively regenerate the signal, ensuring a sharp waveform is delivered to the endpoints.

The key difference for clock repeaters, compared to standard data path buffers, is that they must be designed to have **equal rise and fall times**. This is critical to prevent distorting the clock's duty cycle (the 50/50 split between high and low). We insert as many repeaters as needed to maintain signal integrity across the entire tree.

<img width="886" height="635" alt="image" src="https://github.com/user-attachments/assets/2472acdc-ca86-4eb0-9176-afca5196325d" />

### Crosstalk and clock net shielding

Even after we build a perfectly balanced tree with zero skew, the clock net is still vulnerable. The clock is a **critical net** and must be protected from noise generated by neighboring wires.

<img width="878" height="623" alt="image" src="https://github.com/user-attachments/assets/745f6424-116d-403d-b8fc-ba850250629e" />

When a nearby aggressor wire switches, the **coupling capacitance** between it and our victim clock net can cause significant problems. This interference, known as **crosstalk**, can lead to two issues:

1.  **Glitch:** An unwanted, false pulse on the clock line.
2.  **Delta Delay:** The clock signal is either sped up or slowed down by the aggressor's switching.

<img width="703" height="343" alt="image" src="https://github.com/user-attachments/assets/4b805884-347c-4c63-bb55-67b24277ff47" />

The biggest impact of delta delay is that it **makes our carefully balanced skew non-zero** again, which can cause timing failures.

**Clock Net Shielding** is the technique used to prevent this. We protect the clock net by placing dedicated "shield" wires parallel to it, often on both sides. These shield wires are connected to a stable voltage (VDD or GND) and **do not switch**. This shielding wire effectively breaks the coupling capacitance between the aggressor and the victim, isolating the clock net and preserving its signal integrity.

<img width="1237" height="667" alt="image" src="https://github.com/user-attachments/assets/332285de-8237-46de-92ef-d5f74e0f7cb1" />

---

## Timing analysis with real clock using openSTA

### Setup timing analysis using real clocks

When we move from an ideal clock to a **real clock**, the clock tree is no longer a perfect, zero-delay connection. It's a physical network of buffers and wires. This means the clock signal takes time to travel from its source to the launch and capture flip-flops, and these travel times are often different.

<img width="1201" height="692" alt="image" src="https://github.com/user-attachments/assets/dc2531de-9090-4d0d-adf8-8938aa0fa6e1" />

Let's assume:
* **$\theta$ (theta)** = Combinational logic delay
* **$T$** = Clock period
* **$\Delta 1$ (delta 1)** = Total delay of the clock path to the **launch flop** (1+2)
* **$\Delta 2$ (delta 2)** = Total delay of the clock path to the **capture flop** (1+3+4)

<img width="1227" height="633" alt="image" src="https://github.com/user-attachments/assets/86913a1c-c070-4102-abe6-3d83c23e33fe" />

The old ideal equation was $\theta < T$. The new "real" setup condition is that the data must arrive at the capture flop *before* the next clock edge arrives *at capture flop*.

* **Data Arrival Time:** The time it takes for data to launch and travel to the capture flop.
    * $\text{Data Arrival} = (\text{Clock arrival at launch flop}) + (\text{Flop Clock-to-Q delay}) + (\text{Combinational logic delay})$
    * For simplicity, let's represent this as $\Delta 1 + \theta$.

* **Data Required Time:** The time by which the data *must* arrive. This is the time of the next clock edge at the capture flop, minus the flop's setup time ($S$) and any clock uncertainty ($US$).
    * $\text{Data Required} = (\text{Clock Period}) + (\text{Clock arrival at capture flop}) - S - US$
    * $\text{Data Required} = T + \Delta 2 - S - US$

The final setup equation becomes:
**$(\theta + \Delta 1) < (T + \Delta 2 - S - US)$**

**Slack** is the margin we have. If `Slack = (Data Required Time) - (Data Arrival Time)` is positive, the check passes. If it is negative, we have a setup violation.

### Hold timing analysis using real clocks

The hold check is different. It ensures that the *new* data, launched by a clock edge, does not arrive at the capture flop *too early* and overwrite the *old* data that the capture flop is trying to hold from the *same* clock edge.

<img width="1228" height="686" alt="image" src="https://github.com/user-attachments/assets/abd425e8-b3d7-451d-97ac-b291526c3beb" />

The ideal condition is that the **combinational delay ($\theta$) must be greater than the hold time ($H$)** of the capture flop.

Now, let's add the real clock path delays:

* **Data Arrival Time (Earliest):** The fastest time the new data can launch and travel to the capture flop.
    * $\text{Data Arrival} = (\text{Clock arrival at launch flop}) + (\text{Flop C-Q delay}) + (\text{Earliest logic delay})$
    * We can represent this as **$\theta + \Delta 1$**.

* **Data Required Time (Latest):** The time at which the capture flop no longer needs the old data to be stable.
    * $\text{Data Required} = (\text{Clock arrival at capture flop}) + (\text{Hold Time } H)$
    * We can represent this as **$H + \Delta 2$**.

<img width="1107" height="600" alt="image" src="https://github.com/user-attachments/assets/80420c7b-78ad-480c-a03e-eade14ec7a10" />

The final hold equation is:
**$(\theta + \Delta 1) > (H + \Delta 2)$**

**Hold Slack** = `(Data Arrival Time) - (Data Required Time)`. This value must be positive or zero. If it's negative, the new data is arriving too fast, causing a hold violation. Clock uncertainty is also factored into this calculation by the STA tool.

<img width="1187" height="685" alt="image" src="https://github.com/user-attachments/assets/6b089ee3-071e-4744-ac5d-975d29d9d5c6" />

### Let's identify the timing paths from design, with single clock

<img width="1251" height="642" alt="image" src="https://github.com/user-attachments/assets/8fac953f-7263-444d-8434-448b69f1061d" />
