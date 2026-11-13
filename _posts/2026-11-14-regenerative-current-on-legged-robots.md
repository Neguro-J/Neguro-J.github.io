# When Motors Give Energy Back: Understanding Regenerative Energy in Legged Robots

## How a simple conversation turned into a much bigger calculation

This whole thing started with a conversation with one of my professors.

We were discussing electrical systems and some of the considerations involved in designing robots with multiple motors. At some point, he mentioned something particularly important: **motor regeneration and what happens to the energy that electric motors can send back into a system.**

It was something worth thinking about, especially when designing robots with many actuators connected to the same power supply.

After that conversation, I started looking into the topic. After all, if I ever end up building a complex robot, I'd rather not accidentally fry the whole thing—at least not that quickly. I wanted to understand how much regenerative current its motors could potentially produce and whether that energy could become a problem for the rest of the electronics.

The tricky part was that I couldn't find a directly usable regeneration-current estimate in the available actuator documentation.

And since I really didn't want to accidentally damage a robot because of something I hadn't considered in the electrical design, I decided to investigate how I could estimate it myself.

What started as a search for a few equations somehow turned into a much larger exercise involving mechanical power, gearbox efficiency, winding losses, inverter efficiency, DC-bus current, and braking resistors.

I eventually put together a simplified theoretical model using established principles of electrical machines and power electronics.

None of these equations are new. I was simply trying to connect them to a practical question: **how can we estimate the energy that a robot's motors might return to its electrical system?**

So I thought it would be interesting to share the reasoning, equations, and examples for anyone working with humanoids, bipeds, quadrupeds, or other multi-actuator robots.

> **Disclaimer:** Everything presented here is a theoretical engineering exercise. The numerical examples use hypothetical motors and electrical parameters. The calculations are not experimental results, manufacturer specifications, or a validated hardware protection design.

---

## 1. First, how can a motor regenerate energy?

Normally, we think of electric motors as devices that consume electrical energy to generate movement.

But electric machines can also operate in the opposite direction.

When an external mechanical force drives a motor while the motor resists that movement, part of the mechanical energy can be converted into electrical energy.

This is the basic principle behind regenerative operation.

Imagine a biped robot walking downhill. As its knee bends under the robot's weight, the motor may need to apply torque opposite to the joint's movement to control the descent.

In that situation, the motor is absorbing mechanical energy.

Depending on the actuator and its controller, some of this energy may return to the electrical system.

The signed mechanical power of a joint is:

$$
P_{\mathrm{joint}}=\tau_{\mathrm{act}}\cdot\omega
$$

Where:

- <i>τ</i><sub>act</sub>: actuator torque (Nm).
- <i>ω</i>: joint angular velocity (rad/s).
- <i>P</i><sub>joint</sub>: mechanical power (W).

When torque and angular velocity have the same sign, the actuator delivers mechanical power.

When they have opposite signs, the actuator absorbs mechanical power.

For a braking event, we can define the magnitude of absorbed power as:

$$
P_{\mathrm{brake}}=\max(0,-P_{\mathrm{joint}})
$$

However, absorbing mechanical energy does not necessarily mean that all of it reaches the battery or DC bus.

Some energy is lost through the gearbox, motor windings, and electronic components.

This is where the calculations become interesting.

---

## 2. Following the energy backward through the motor

To estimate regeneration, I decided to follow the energy conversion process in reverse.

Instead of starting with electrical power and calculating mechanical output, we start with the mechanical power absorbed by the joint and estimate how much electrical power remains after accounting for losses.

The simplified energy path looks like this:

**Joint motion → Gearbox → Motor rotor → Electrical windings → Motor driver → DC bus**

Each stage introduces losses.

### Step 1 — Angular velocity

Motor specifications often express rotational speed in revolutions per minute (RPM).

For mechanical power calculations, we convert it to radians per second:

$$
\boxed{\omega=\mathrm{RPM}\cdot\frac{2\pi}{60}}
$$

This gives the output-shaft angular velocity in rad/s.

### Step 2 — Mechanical power at the joint

Using positive magnitudes for braking torque and angular velocity:

$$
\boxed{P_{\mathrm{joint,abs}}=T\cdot\lvert\omega\rvert}
$$

Where <i>T</i> is the magnitude of the resisting torque.

This represents the mechanical power being absorbed at the output joint.

An important detail is that torque and velocity must correspond to the same physically possible operating point.

We should not automatically combine maximum torque and maximum speed from a motor specification, because those limits may not be achievable simultaneously.

### Step 3 — Mechanical power reaching the rotor

Most robotic actuators include a gearbox.

During reverse operation, mechanical energy travels from the joint toward the motor rotor, and some of it is lost through friction.

For a simplified model:

$$
\boxed{P_{\mathrm{rot}}=P_{\mathrm{joint,abs}}\cdot\eta_{\mathrm{mec}}}
$$

Where <i>η</i><sub>mec</sub> represents the assumed reverse mechanical efficiency.

For example, if we assume:

$$
\eta_{\mathrm{mec}}=0.88
$$

we are estimating that 88% of the mechanical power reaches the motor rotor.

This is only an illustrative efficiency. Real reverse efficiency depends on the transmission design, load, and operating conditions.

### Step 4 — Estimating motor phase current

To estimate electrical losses, we first need the motor current.

Motor torque is related to current through the torque constant:

$$
T_{\mathrm{motor}}=K_t I_{\mathrm{fase}}
$$

Therefore:

$$
\boxed{I_{\mathrm{fase}}=\frac{T_{\mathrm{motor}}}{K_t}}
$$

For a geared actuator, we must distinguish motor-shaft torque from joint output torque.

Using gear ratio <i>G</i> and the same reverse efficiency approximation:

$$
T_{\mathrm{motor}}\approx\frac{\eta_{\mathrm{mec}}T}{G}
$$

Therefore:

$$
\boxed{I_{\mathrm{fase}}\approx\frac{\eta_{\mathrm{mec}}T}{G K_t}}
$$

Here, <i>K</i><sub>t</sub> is assumed to be the motor-shaft torque constant expressed in Nm per ampere RMS.

This detail matters because manufacturers may express torque constants using peak phase current, RMS phase current, or output-shaft torque.

Those conventions cannot be mixed without conversion.

### Step 5 — Electrical losses in the windings

Electric motors produce heat because their windings have electrical resistance.

These are commonly called copper losses or Joule losses.

For a balanced three-phase motor:

$$
P_{\mathrm{Cu}}=3R_{\mathrm{fase}}I_{\mathrm{fase,RMS}}^2
$$

For a wye-connected motor, assuming the measured line-to-line DC resistance is twice the individual phase resistance:

$$
R_{\mathrm{fase}}=\frac{R_{L-L}}{2}
$$

Substituting:

$$
\boxed{P_{\mathrm{Cu}}=1.5R_{L-L}I_{\mathrm{fase,RMS}}^2}
$$

Where:

- <i>R</i><sub>L−L</sub>: resistance measured between two motor leads (Ω).
- <i>I</i><sub>fase,RMS</sub>: RMS phase current (A).
- <i>P</i><sub>Cu</sub>: winding losses (W).

The factor 1.5 depends on the stated winding and current conventions.

One interesting detail is that copper losses occur whenever current flows, regardless of whether the motor is producing movement or absorbing mechanical energy.

This means that a motor can absorb mechanical power without necessarily returning useful electrical power.

### Step 6 — Electrical power available for regeneration

Now we subtract the estimated winding losses from the power arriving at the rotor:

$$
\boxed{P_{\mathrm{AC,regen}}=\max(0,P_{\mathrm{rot}}-P_{\mathrm{Cu}})}
$$

The `max(0, ...)` operation keeps only the positive regenerative contribution predicted by this simplified model.

If the mechanical power reaching the motor is insufficient to overcome the modeled electrical losses, the estimate becomes zero.

This does not mean the actuator consumes no electricity. It simply means we are not predicting positive regenerative output in that condition.

Other losses, such as magnetic losses, are neglected here.

### Step 7 — Power reaching the DC bus

The motor driver also introduces conversion losses.

Using a simplified inverter efficiency:

$$
\boxed{P_{\mathrm{DC,regen}}=P_{\mathrm{AC,regen}}\cdot\eta_{\mathrm{inv}}}
$$

For this exercise, we can assume:

$$
\eta_{\mathrm{inv}}=0.96
$$

Finally, we calculate regenerative current:

$$
\boxed{I_{\mathrm{regen}}=\frac{P_{\mathrm{DC,regen}}}{V_{\mathrm{bus}}}}
$$

Where <i>V</i><sub>bus</sub> is the DC-link voltage.

Notice that the regenerative current reaching the DC bus is not the same as the motor's phase current.

They refer to different sides of the conversion process.

---

## 3. A numerical example with a hypothetical motor

Let's imagine a geared electric actuator with the following parameters.

These numbers are invented for the exercise and do not correspond to a specific commercial motor.

| Parameter | Assumed value |
|---|---:|
| Joint braking torque | 40 Nm |
| Joint angular velocity | 50 RPM |
| Gear ratio | 10:1 |
| Reverse mechanical efficiency | 0.88 |
| Motor-shaft torque constant (RMS) | 0.20 Nm/A |
| Line-to-line resistance | 0.12 Ω |
| Inverter efficiency | 0.96 |
| DC-bus voltage | 52.9 V |

Now we calculate everything step by step.

### 1. Angular velocity

$$
\omega=50\frac{2\pi}{60}
$$

$$
\omega\approx5.236\ \mathrm{rad/s}
$$

### 2. Mechanical power absorbed by the joint

$$
P_{\mathrm{joint,abs}}=40(5.236)
$$

$$
P_{\mathrm{joint,abs}}\approx209.4\ \mathrm{W}
$$

### 3. Mechanical power reaching the rotor

$$
P_{\mathrm{rot}}=209.4(0.88)
$$

$$
P_{\mathrm{rot}}\approx184.3\ \mathrm{W}
$$

### 4. Motor phase current

$$
I_{\mathrm{fase}}=\frac{0.88(40)}{10(0.20)}
$$

$$
I_{\mathrm{fase}}=17.6\ \mathrm{A}
$$

### 5. Copper losses

$$
P_{\mathrm{Cu}}=1.5(0.12)(17.6)^2
$$

$$
P_{\mathrm{Cu}}\approx55.8\ \mathrm{W}
$$

### 6. Net regenerated power at the DC bus

$$
P_{\mathrm{DC,regen}}=(184.3-55.8)(0.96)
$$

$$
\boxed{P_{\mathrm{DC,regen}}\approx123.4\ \mathrm{W}}
$$

### 7. Regenerative DC current

$$
I_{\mathrm{regen}}=\frac{123.4}{52.9}
$$

$$
\boxed{I_{\mathrm{regen}}\approx2.33\ \mathrm{A}}
$$

Under these theoretical conditions, our imaginary actuator could return approximately **123.4 W and 2.33 A** to the DC bus.

Of course, this represents just one operating point.

The current would change with torque, angular velocity, mechanical efficiency, winding temperature, and other operating conditions.

It is not a measured value or a guaranteed maximum.

---

## 4. Why things get more interesting with multiple motors

Now imagine we are designing a humanoid with 20 actuators.

Not every actuator will be regenerating energy simultaneously.

During walking, some joints may consume electrical power while others absorb mechanical energy.

For example:

- One knee is braking during a controlled descent.
- Another leg is extending to support the robot.
- A hip actuator is repositioning the body.
- Other joints are moving or maintaining posture.

The interesting part is that these events can happen at the same time.

If several identical motors regenerate simultaneously, a simple group estimate is:

$$
\boxed{I_{\mathrm{grupo}}=N I_{\mathrm{regen}}}
$$

Where <i>N</i> is the number of motors under the same assumed operating condition.

For different motor types or operating points, we should instead sum their individual contributions:

$$
\boxed{I_{\mathrm{regen,total}}=\sum_{i=1}^{N}I_{\mathrm{regen},i}}
$$

For example, suppose four hypothetical actuators regenerate the same 123.4 W from our previous calculation.

$$
P_{\mathrm{regen,total}}=4(123.4)
$$

$$
\boxed{P_{\mathrm{regen,total}}=493.6\ \mathrm{W}}
$$

At 52.9 V, this is approximately:

$$
I_{\mathrm{regen,total}}=\frac{493.6}{52.9}
$$

$$
I_{\mathrm{regen,total}}\approx9.33\ \mathrm{A}
$$

But does that mean all 493.6 W must be sent to a braking resistor?

Not necessarily.

And this brings us to one of the most interesting aspects of a multi-motor electrical architecture.

---

## 5. Why a shared DC bus matters

When designing a robot with several electric motors, a shared DC power bus can be useful.

Instead of treating every actuator as an isolated electrical system, compatible motor drivers may share the same DC supply.

Each motor still requires its own driver and control system.

**We are not connecting the three-phase motor windings directly in parallel. We are talking about sharing a DC power bus between compatible motor controllers.**

This distinction is important.

Suppose one motor is mechanically braking and regenerating 120 W.

At the same time, another motor requires 90 W to accelerate a joint.

If both motor drivers support bidirectional energy flow through their shared DC bus, the regenerative contribution of the first motor can help supply the second.

That means less net power may need to come from the battery.

The energy does not need to be deliberately transferred from one particular motor to another. Both interact with the electrical power balance of the same bus.

A simplified model is:

$$
P_{\mathrm{net}}=
P_{\mathrm{regen,total}}
-P_{\mathrm{motoring,total}}
-P_{\mathrm{aux}}
$$

Where:

- <i>P</i><sub>regen,total</sub>: total power returned by regenerating actuators.
- <i>P</i><sub>motoring,total</sub>: power consumed by actuators operating in motoring mode.
- <i>P</i><sub>aux</sub>: power consumed by auxiliary electronics.

Let's return to our hypothetical humanoid.

Suppose:

| Electrical contribution | Power |
|---|---:|
| Four motors regenerating | +493.6 W |
| Other motors consuming | −300 W |
| Auxiliary electronics | −40 W |

The net power is:

$$
P_{\mathrm{net}}=493.6-300-40
$$

$$
\boxed{P_{\mathrm{net}}=153.6\ \mathrm{W}}
$$

Although the four actuators are collectively generating 493.6 W, the electrical system has only 153.6 W of net surplus under these assumptions.

This is why calculating the entire system's energy balance can be more useful than simply adding the maximum theoretical regeneration of every motor.

If the battery can safely accept some of that excess energy, the amount remaining to be managed becomes even smaller.

For example, if the battery accepts 100 W:

$$
P_{\mathrm{excess}}=153.6-100
$$

$$
\boxed{P_{\mathrm{excess}}=53.6\ \mathrm{W}}
$$

A general first-order expression is:

$$
\boxed{
P_{\mathrm{excess}}=
\max\left(
0,
P_{\mathrm{regen,total}}
-
P_{\mathrm{motoring,total}}
-
P_{\mathrm{aux}}
-
P_{\mathrm{battery,accepted}}
\right)
}
$$

This assumes an instantaneous power balance. A real dynamic model must also consider energy temporarily stored in the bus capacitors and the response of the connected electronics.

The key idea is simple:

**In a properly designed shared-bus architecture, regenerated energy may be used by other motors before any remaining surplus needs to be dissipated.**

This is not a universal feature of every controller. It requires electrically compatible hardware designed to support this behavior.

---

## 6. What happens when there is too much regenerated energy?

This is where electrical protection becomes important.

Imagine several actuators braking simultaneously while the rest of the robot consumes very little power.

If the incoming electrical power exceeds what the motors, battery, and auxiliary electronics can absorb, the excess energy may increase the voltage of the DC bus.

The bus capacitors temporarily store electrical energy according to:

$$
\boxed{E_C=\frac{1}{2}CV_{\mathrm{bus}}^2}
$$

Therefore, when their voltage changes:

$$
\boxed{\Delta E_C=\frac{1}{2}C(V_2^2-V_1^2)}
$$

Where:

- <i>C</i>: equivalent DC-bus capacitance.
- <i>V</i><sub>1</sub>: initial bus voltage.
- <i>V</i><sub>2</sub>: final bus voltage.

With a simplified net charging current, the instantaneous voltage-rise rate is approximately:

$$
\boxed{\frac{dV_{\mathrm{bus}}}{dt}=\frac{I_{\mathrm{net}}}{C}}
$$

This explains why even a relatively short pulse of regenerative current can matter if the system cannot absorb it quickly enough.

If the DC-bus voltage exceeds the safe operating limits of the connected devices, the motor controllers or other electronic components may experience protection faults or potentially damaging electrical stress.

The goal is therefore not only to estimate regenerative power, but also to understand what happens to that energy under different operating conditions.

---

## 7. An example using a hypothetical regeneration clamp

One possible way to manage excess energy is through a braking resistor controlled by a regeneration clamp or braking chopper.

The basic idea is straightforward.

When the bus voltage exceeds a predefined threshold, the circuit connects an appropriate resistor to dissipate part of the excess electrical energy as heat.

The resistor current is:

$$
\boxed{I_{\mathrm{brake}}=\frac{V_{\mathrm{bus}}}{R_{\mathrm{brake}}}}
$$

And the instantaneous dissipated power is:

$$
\boxed{P_{\mathrm{brake}}=\frac{V_{\mathrm{bus}}^2}{R_{\mathrm{brake}}}}
$$

Now imagine a completely hypothetical braking circuit operating at 52 V with a 2 Ω external resistor.

The theoretical current when fully switched on is:

$$
I_{\mathrm{brake}}=\frac{52}{2}
$$

$$
\boxed{I_{\mathrm{brake}}=26\ \mathrm{A}}
$$

Its instantaneous dissipation is:

$$
P_{\mathrm{brake}}=\frac{52^2}{2}
$$

$$
\boxed{P_{\mathrm{brake}}=1352\ \mathrm{W}}
$$

That is approximately 1.35 kW while the resistor is fully connected at the assumed voltage.

However, this does not mean the resistor can continuously dissipate 1.35 kW.

For short braking events, we also need to consider the total pulse energy:

$$
\boxed{E_{\mathrm{brake}}=\int P_{\mathrm{brake}}(t)\,dt}
$$

For example, a 100 ms pulse at a constant 1352 W would correspond to:

$$
E_{\mathrm{brake}}=1352(0.1)
$$

$$
\boxed{E_{\mathrm{brake}}=135.2\ \mathrm{J}}
$$

The actual circuit must be evaluated against its switching-device limits, resistor pulse-energy capacity, cooling, duty cycle, voltage limits, and control response.

For a first screening, we might compare the estimated excess regenerative current against the braking circuit's available current capacity.

However, current capacity alone does not guarantee that the bus voltage will remain within safe limits.

The protection must respond at the right voltage and fast enough for the electrical system's dynamics.

**This example is only intended to show how the equations work. It is not a recommendation to use a particular resistor or clamp in a real robot.**

---

## 8. What about stall conditions and emergency stops?

Another detail that came up while exploring the equations was the difference between high torque and actual mechanical power.

A motor can produce substantial torque while rotating very slowly.

But mechanical power depends on both torque and angular velocity:

$$
P=T\omega
$$

At zero rotational speed:

$$
\omega=0
$$

Therefore:

$$
\boxed{P_{\mathrm{mechanical}}=0}
$$

This means that a stationary motor, even while producing holding torque, cannot generate mechanical regenerative power through shaft movement.

However, maintaining that torque can still require electrical current and produce significant winding heat.

Emergency stops introduce another complication.

Depending on how an emergency-stop system works, the motor controllers may disable active control, disconnect the power supply, or enter another defined state.

The electrical behavior after this event cannot automatically be described by the normal active-regeneration equations.

Motor back-EMF, passive current paths, bus capacitance, and remaining connections all depend on the actual hardware architecture.

In other words, **regeneration protection and emergency-stop safety are different design considerations**, even though they may interact.

---

## 9. Where could this be useful in robotics?

Although I started thinking about this problem in the context of a humanoid, the same principles apply to many other robotic systems.

### Humanoid and biped robots

During walking, squatting, controlled descents, or landing, some joints may absorb mechanical energy.

Knees, hips, and ankles can experience braking conditions depending on the movement and control strategy.

If these actuators share a compatible DC bus, their regenerated energy may partially offset the consumption of other joints.

### Quadruped robots

Quadrupeds can experience repeated braking and acceleration events across several legs.

During dynamic locomotion, some actuators may absorb energy while others produce mechanical work.

This makes the timing of regenerative events particularly interesting.

### Robotic arms

A robotic arm lowering a payload may have joints operating under negative mechanical power.

Regeneration can also occur during controlled deceleration of rotating links.

### Other multi-actuator systems

The same general reasoning can apply to exoskeletons, industrial manipulators, and electrically actuated mobile mechanisms.

The specific parameters change, but the underlying energy-conversion equations remain relevant.

---

## 10. How could we build a better model?

The examples above estimate regeneration at individual operating points.

But a real robot does not operate at a constant torque and angular velocity.

Both values change continuously during movement.

A more realistic approach would be to calculate the regeneration of every actuator as a function of time.

For each motor:

$$
P_{\mathrm{joint},i}(t)=\tau_i(t)\omega_i(t)
$$

Then estimate the power that reaches the DC bus:

$$
P_{\mathrm{DC,regen},i}(t)
$$

Finally, combine the contributions from all actuators:

$$
P_{\mathrm{regen,total}}(t)=\sum_{i=1}^{N}P_{\mathrm{DC,regen},i}(t)
$$

We can then compare this against the power consumed by other motors and the electrical system.

This would let us investigate questions such as:

- Which movements produce the largest regenerative peaks?
- How much regeneration occurs during downhill locomotion?
- How much energy can be reused by other actuators?
- What happens when several joints brake at the same time?
- How much surplus energy may need to be dissipated?

For the braking circuit, we could estimate accumulated dissipation over an interval:

$$
\boxed{E_{\mathrm{resistor}}=\int_{t_0}^{t_1}P_{\mathrm{resistor}}(t)\,dt}
$$

That would be more informative than looking at peak current alone.

Of course, making this model useful for actual hardware would require realistic torque and velocity profiles, correct motor constants, verified efficiencies, controller behavior, battery limitations, and electrical measurements.

Simulation could provide an initial operating profile, but real-world validation would still be necessary.

---

## Final thoughts

It's funny how a relatively simple conversation can turn into a much bigger engineering exercise than originally planned.

I initially just wanted to estimate how much current a motor could potentially return to a robot's electrical system.

But to understand that, I ended up looking into mechanical power, transmission losses, winding resistance, motor constants, inverter efficiency, and the behavior of shared DC power buses.

One of the most interesting things I learned was that regenerated energy doesn't necessarily have to be wasted.

In a properly designed system, energy returned by one actuator may help supply another motor that is consuming power at the same time.

The remaining challenge is understanding what happens when the robot produces more electrical energy than it can consume or safely store.

The equations presented here come from established principles of electrical machines and power electronics. The simplified model is just an attempt to put those principles together to explore a practical robotics question.

There is still a big difference between estimating regenerative power theoretically and verifying the behavior of a real motor, driver, battery, and protection circuit.

But that was exactly the motivation behind this exercise.

**Before deciding how to protect a robot from regenerative energy, I first wanted to understand how much energy could potentially be there.**

And all of this started because a professor mentioned something worth paying attention to during a conversation.

---

## References and further reading

The following sources provide the theoretical foundations and relevant background for the equations and discussion. The numerical parameters used throughout this post are hypothetical.

**1. Electrical machines and energy conversion**

Wildi, T. (2006). *Electrical Machines, Drives, and Power Systems* (6th ed.). Pearson Prentice Hall.

A useful reference for electrical-machine fundamentals, torque, power conversion, efficiency, winding losses, and three-phase electrical systems.

**2. Power electronics and electrical energy conversion**

Erickson, R. W., & Maksimović, D. (2020). *Fundamentals of Power Electronics* (3rd ed.). Springer.

[DOI](https://doi.org/10.1007/978-3-030-43881-4)

Relevant for converter efficiency, electrical energy storage, power balance, and converter operation.

**3. Energy efficiency in legged robotics**

Seok, S., et al. (2015). *Design Principles for Energy-Efficient Legged Locomotion and Implementation on the MIT Cheetah Robot*. IEEE/ASME Transactions on Mechatronics, 20(3), 1117–1129.

[DOI](https://doi.org/10.1109/TMECH.2014.2339013)

Discusses electrical and mechanical losses, regenerative electronics, and design considerations for energy-efficient legged locomotion.

**4. Energy recovery in robotic systems**

Carabin, G., Wehrle, E., & Vidoni, R. (2017). *A Review on Energy-Saving Optimization Methods for Robotic and Automatic Systems*. Robotics, 6(4), 39.

[DOI](https://doi.org/10.3390/robotics6040039)

Reviews energy-saving techniques in robotic systems, including regenerative drives, DC-bus energy sharing, and energy recovery.
