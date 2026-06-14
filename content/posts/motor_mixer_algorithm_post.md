---
title: "Quadcopter: Motor Mixer Algorithm"
date: 2025-01-15T00:25:52-05:00
type: "post"
showTableOfContents: true
---

{{< figure src="/images/motor-mixer-blog/my-quadcopter-small.png"
alt="My Quadcopter"
class="center-caption" >}}

## Introduction

With a personal goal of developing a deep understanding of quadcopter dynamics, I built my own quadcopter based on the open-source Carbon Aeronautics drone. This required purchasing parts online, making modifications, and assembling the whole quadcopter from the ground up until flight testing. Even though it may sound like a straightforward path from assembly to testing, I spent months stuck trying to make it fly due to both hardware and software issues. Little by little, the fun turned into frustration.

One of the main issues I faced was that my quadcopter flipped on every takeoff. After extensive troubleshooting, I became convinced that the problem was the motor mixer algorithm, without realizing there were other possible causes for this behavior. Weeks later, I discovered that the real issue was the electronic speed controller (ESC) miscalibration. However, by then I had already spent a lot of time learning about motor mixer algorithms, to the point that I derived one for my quadcopter to confirm whether it was actually correct.

## What you will learn

The purpose of this blog is to share the derivation of the motor mixer from the quadcopter’s geometric layout, assuming you already know some basic physics and linear algebra. This gives us a starting idea of how popular flight controllers handle motor mixers and it helps us understand through equations how most of the drone’s physical parameters are absorbed into the PID gains, so that in practice we mostly care about retuning the drone experimentally.

## Quadcopter

The open-source quadcopter is a low-cost micro H-configuration drone powered by a 2S LiPo battery and designed for teaching mechanical, electronic, and flight controller software concepts. Its greatest advantage is its well-documented manual and YouTube videos, which cover everything from assembly and flight testing to the mathematical modeling of quadcopter dynamics. I highly recommend it if you are interested in building your own drone from scratch.

This platform was especially helpful for my learning goals because both the hardware and software are fully hackable and adaptable. This allowed me to modify the design and develop my own flight controller version. Some of the changes I made include a custom carbon-fiber lower frame, an sBUS receiver, different motors and ESCs, flight logging through an SD card, and several software features such as a task scheduler, arming/disarming functionality, and buzzer notifications. The source code is available on GitHub.

The video below shows the quadcopter flying in acro mode.

## Motor Mixer Algorithm

{{< figure src="/images/motor-mixer-blog/motor-mixer-general.svg"
alt="Motor Mixer Typical"
caption="Fig. 1. General motor mixer algorithm diagram."
class="center-caption" >}}

As shown in Fig. 1, the motor mixer algorithm combines the outputs of the controller to generate individual motor commands. In general, the PID controller produces the desired thrust and the desired roll, pitch, and yaw torques. The mixer then converts these values into individual motor commands according to the quadcopter geometry.

Depending on the implementation, the mixer outputs may represent either desired motor thrust commands or normalized motor commands. For this quadcopter, the mixer outputs are normalized motor commands that are fed directly to the ESCs as PWM signals, as demonstrated in Fig. 2.

{{< figure src="/images/motor-mixer-blog/motor-mixer-normalize.svg"
alt="Motor Mixer Normalize"
caption="Fig. 2. Motor mixer algorithm diagram for this quadcopter." >}}

Therefore, the purpose of the mixer algorithm is to determine the contribution of each motor required to achieve the desired motion of the quadcopter.

## Find Motor Mixer Algorithm iteratively

The manual of the Carbon Aeronautics drone explains in detail that the motor mixer for this drone is a linear combination of the throttle, roll, pitch, and yaw inputs, as shown below. To find the correct combination, we have to use the quadcopter configuration and go through every movement case until the mixer algorithm is correct. This is a very tedious iterative approach, but it allows us to avoid going through the quadcopter dynamics mathematically. Note that these inputs are provided directly by the controller.

{{< small-matrix-block >}}

$$
\begin{aligned}
\text{output motor 1} &= \text{throttle input} - \text{roll input} - \text{pitch input} - \text{yaw input} \\
\text{output motor 2} &= \text{throttle input} - \text{roll input} + \text{pitch input} + \text{yaw input} \\
\text{output motor 3} &= \text{throttle input} + \text{roll input} + \text{pitch input} - \text{yaw input} \\
\text{output motor 4} &= \text{throttle input} + \text{roll input} - \text{pitch input} + \text{yaw input}
\end{aligned}
$$

{{< /small-matrix-block >}}

Rewriting the mixer in a clean matrix form, we have
{{< small-matrix-block >}}

$$
\begin{bmatrix} \text{output motor 1} \\ \text{output motor 2} \\ \text{output motor 3} \\ \text{output motor 4} \end{bmatrix} = \begin{bmatrix} 1 & -1 & -1 & -1 \\ 1 & -1 & 1 & 1 \\ 1 & 1 & 1 & -1 \\ 1 & 1 & -1 & 1 \end{bmatrix} \begin{bmatrix} \text{throttle input} \\ \text{roll input} \\ \text{pitch input} \\ \text{yaw input} \end{bmatrix} \tag{1}
$$

{{< /small-matrix-block >}}

For now, let's assume the motor mixer is just the summation of each input without considering the sign of each input. We will add the sign of each input as we consider each case scenario.

{{< small-matrix-block >}}

$$
\begin{aligned}
\text{output motor 1} &= \text{throttle input} + \text{roll input} + \text{pitch input} + \text{yaw input} \\
\text{output motor 2} &= \text{throttle input} + \text{roll input} + \text{pitch input} + \text{yaw input} \\
\text{output motor 3} &= \text{throttle input} + \text{roll input} + \text{pitch input} + \text{yaw input} \\
\text{output motor 4} &= \text{throttle input} + \text{roll input} + \text{pitch input} + \text{yaw input}
\end{aligned}
$$

{{< /small-matrix-block >}}

Let's take the rolling case scenario, where we want the quadcopter to roll to the right. First, we assume that an input percentage maps directly to a motor power percentage. Suppose the quadcopter is hovering with a throttle input of 50%, which causes each motor to operate at 50% of its power. We then apply a roll input of 20%, while assuming there is no pitch input (0%) and no yaw input (0%) interfering with the roll movement. Therefore, the above placeholder mixer consists only of the throttle and roll inputs.

{{< small-matrix-block >}}

$$
\begin{aligned}
\text{output motor 1} &= \text{throttle input} + \text{roll input} \\
\text{output motor 2} &= \text{throttle input} + \text{roll input} \\
\text{output motor 3} &= \text{throttle input} + \text{roll input} \\
\text{output motor 4} &= \text{throttle input} + \text{roll input}
\end{aligned}
$$

{{< /small-matrix-block >}}

{{< figure src="/images/motor-mixer-blog/roll-right-motion-iterative.svg"
alt="Motor coordinates"
caption="Fig. 4. (a) The quacopter rolling to the left. (b) 2D presentation of rolling to the left."
class="center-caption" >}}

From Fig. 3, we see that to roll to the right, we have to decrease the power of motors 1 and 2 by 20%. At the same time, we have to increase the power of motors 3 and 4 by 20%. This means the roll input for motors 1 and 2 must have a negative sign, while the roll input for motors 3 and 4 must have a positive sign. We end up with the following mixer for rolling to the right. If we plug in the roll input, we can clearly see that the two left motors provide less power, while the two right motors provide more power to achieve the rightward movement.

{{< small-matrix-block >}}

$$
\begin{aligned}
\text{output motor 1} &= \text{throttle input} - \text{roll input} = 50\% - (20\%) = 30\% \\
\text{output motor 2} &= \text{throttle input} - \text{roll input} = 50\% - (20\%) = 30\% \\
\text{output motor 3} &= \text{throttle input} + \text{roll input} = 50\% + (20\%) = 70\% \\
\text{output motor 4} &= \text{throttle input} + \text{roll input} = 50\% + (20\%) = 70\%
\end{aligned}
$$

{{< /small-matrix-block >}}

{{< figure src="/images/motor-mixer-blog/roll-left-motion-iterative.svg"
alt="Motor coordinates"
caption="Fig. 4. (a) The quacopter rolling to the right. (b) 2D presentation of rolling to the right."
class="center-caption" >}}

The next question is whether this mixer also works for rolling to the left, where motors 1 and 2 need higher power and motors 3 and 4 need lower power, as shown in Fig. 4. Clearly, this mixer still produces a roll to the right if we provide a positive roll input. Therefore, we must provide a negative roll input so that the signs in the mixer are effectively reversed, resulting in a roll to the left. If we plug in the negative roll input, we can clearly see that the drone rolls to the left. With this in mind, the sign of the roll input determines the direction of the roll motion.

{{< small-matrix-block >}}

$$
\begin{aligned}
\text{output motor 1} &= \text{throttle input} - \text{roll input} = 50\% - (-20\%) = 70\% \\
\text{output motor 2} &= \text{throttle input} - \text{roll input} = 50\% - (-20\%) = 70\% \\
\text{output motor 3} &= \text{throttle input} + \text{roll input} = 50\% + (-20\%) = 30\% \\
\text{output motor 4} &= \text{throttle input} + \text{roll input} = 50\% + (-20\%) = 30\%
\end{aligned}
$$

{{< /small-matrix-block >}}

To obtain the mixers for pitch and yaw movements, we follow the same iterative approach. This is how we arrive at the final motor mixer algorithm in (1) for this quadcopter.

## Find Motor Mixer Algorithm mathematically

From studying the quadcopter rigid-body dynamics, we learned that the translational dynamics of the vehicle depend on the total thrust, $T_{total}$ and the rotational dynamics, on the other hand, depend on the three body torques, $\tau_{roll}$, $\tau_{pitch}$, and $\tau_{yaw}$.

By collecting the equations that describe these dynamics, we can form a matrix relationship between the four motor thrusts and the total thrust and three body torques. This relationship is represented by the mixer matrix $M$:

$$
\begin{bmatrix} T_{total} \\ \tau_{roll} \\ \tau_{pitch} \\ \tau_{yaw} \end{bmatrix} = M \begin{bmatrix} T_1 \\ T_2 \\ T_3 \\ T_4 \end{bmatrix}
$$

By inverting the mixer matrix, we can compute the individual motor thrusts required to produce the desired total thrust and body torques.

$$
\begin{bmatrix} T_1 \\ T_2 \\ T_3 \\ T_4 \end{bmatrix} =  M^{-1} \begin{bmatrix} T_{total} \\ \tau_{roll} \\ \tau_{pitch} \\ \tau_{yaw} \end{bmatrix}
$$

I personally prefer this motor mixer derivation approach because it allows us to verify that the resulting mixer is physically consistent and can be extended to other multirotor configurations.

### Total Thrust

{{< figure src="/images/motor-mixer-blog/total-thrust.svg"
alt="Motor coordinates"
caption="Fig. 4. Quadcopter rotor thrust."
class="center-caption" >}}

It is defined that the thrust force generated by a propeller is directly proportional to the square of the rotor angular velocity, $\omega$, with thrust coefficient $k_T$:

$$
\begin{aligned}
T_i &= k_T \omega_i^2 \qquad i=1,2,3,4\tag{1}
\end{aligned}
$$

Therefore, the total vertical thrust is the sum of the thrust generated by each rotor:

$$
    T_{total} = T_1+T_2+T_3+T_4\tag{2}
$$

### Aerodynamic Drag Torque

It is also known that a propeller generates an aerodynamic drag torque that is proportional to the square of the rotor angular velocity, $\omega$, with moment coefficient $k_Q$. This aerodynamic drag torque acting on the propeller is opposite to the rotation of the propeller. The sign of the drag torque can be determined from the propeller spin direction and the body-frame coordinate convention.

$$
    Q_{i} = k_{Q}\omega_{i}^{2}\qquad i=1,2,3,4\tag{3}
$$

{{< figure src="/images/motor-mixer-blog/drag-torque.svg"
alt="Motor coordinates"
caption="Fig. 4. Aerodynamic Drag Torque for this quadcopter."
class="center-caption" >}}

To obtain the drag torque with its proper sign for this quadcopter, we use the marked axes of the IMU board as the body-frame coordinate convention, where $+x$ is forward and $+y$ is left, as shown in Fig. 4. By taking the cross product of these axes, the $+z$ axis points upward:

$$
+\vec{z} = +\vec{x} \times +\vec{y}
$$

Applying the right-hand rule to the $+z$ axis, a counterclockwise (CCW) propeller has a positive angular velocity, $+ω$, about $+z$. Since aerodynamic drag opposes the rotation, a counterclockwise (CCW) propeller generates a negative aerodynamic drag torque, whereas a clockwise (CW) propeller produces a positive aerodynamic drag torque.

Table 1 summarizes the aerodynamic drag torque with its correct sign for each rotor of this quadcopter.

{{<small-table>}}
_Table 1: Aerodynamic Drag torque from each rotor_
| Rotor | Prop Direction | Drag Torque Sign |
| :------ | :-------: | :---------: |
| motor 1 | CCW | $-1$ |
| motor 2 | CW | $+1$ |
| motor 3 | CCW | $-1$ |
| motor 4 | CW | $+1$ |
{{</small-table>}}

### Roll and Pitch Torque

The roll and pitch torques result from differences in the thrust produced by opposing rotors acting at a distance from the center of the quadcopter. We derive these torques from the standard definition of torque, which is the cross product of the lever-arm vector and the force vector.

$$
\vec{\tau} = \vec{r} \times \vec{F}
$$

Hence, the net torque acting on the drone body is the sum of all individual torques.

$$
\vec{\tau}_{net} = \sum \vec{r_i} \times \vec{F_i}
$$

Once we perform the cross product and separate the x-axis and y-axis components, the roll torque and pitch torque are defined as

$$\tau_\text{roll} = \sum y_iT_i$$

$$\tau_\text{pitch}  = \sum(-x_iT_i)$$

Fig. 5 illustrates the correct coordinate position of each motor of this quadcopter, where the axes of the IMU board are used as the body-frame coordinate convention. In this convention, $+x$ is forward, $+y$ is left, and $+z$ is upward.

{{< figure src="/images/motor-mixer-blog/motor-coordinates.svg"
alt="Motor coordinates"
caption="Fig. 5. Motor coordinate position using the body-frame convention."
class="center-caption" >}}

By plugging in the corresponding y-coordinates of each motor position, we obtain the final roll torque.

$$
\begin{aligned}
\tau_\text{roll} = \sum y_iT_i = y_1T_1+y_2T_2+y_3T_3+y_4T_4 \\
\tau_\text{roll} = (-L_y)T_1+(-L_y)T_2+(L_y)T_3+(L_y)T_4 \\
\tau_\text{roll} = L_y(-T_1-T_2+T_3+T_4)
\end{aligned}
$$

Similarly, by plugging in the corresponding x-coordinates of each motor position, we obtain the final pitch torque.

$$
\begin{aligned}
\tau_\text{pitch}  = \sum(-x_iT_i) = -(x_1T_1+x_2T_2+x_3T_3+x_4T_4) \\
\tau_\text{pitch}  = -[(L_x)T_1+(-L_x)T_2+(-L_x)T_3+(L_x)T_4] \\
\tau_\text{pitch}  =  L_x(-T_1+T_2+T_3-T_4)
\end{aligned}
$$

### Yaw Torque

The aerodynamic drag torque, $Q_i$, acting on each propeller generates an equal-magnitude reaction torque on the drone frame through the motor–propeller interaction. This reaction torque is responsible for the yaw motion of the quadcopter. Hence, the net yaw torque acting on the quadcopter frame is the sum of the reaction torques from all rotors about the body-frame z-axis.

$$
\tau_{yaw} = \sum_{i=1}^{4} s_i Q_i \qquad s_i \in \{+1, -1\}
$$

For this quadcopter, a positive yaw torque corresponds to a counterclockwise rotation about the body-frame z-axis according to the right-hand rule. A counterclockwise (CCW) rotor generates a reaction torque on the frame in the clockwise direction, which corresponds to a negative yaw contribution. Conversely, a clockwise (CW) rotor generates a positive yaw contribution. Therefore, the sign factor $s_i$ is defined as

$$
s_i =
\begin{cases}
+1 & \text{CW rotor} \\
-1 & \text{CCW rotor}
\end{cases}
$$

Fig.6 shows the sign factor, $s_i$, associated with each rotor's spin direction. Substituting these values yields

$$
\tau_{yaw} = - Q_1 + Q_2-Q_3 + Q_4
$$

By introducing the rotor drag-to-thrust coefficient ratio $\gamma = \frac{k_Q}{k_T}$, it is convenient to express $Q_i$ in terms of the thrust force $T_i$.

$$
Q_i = \frac{k_Q}{k_T} T_i = \gamma T_i
$$

Substituting into (2), we obtain the final yaw torque in terms of $T_i$ and $\gamma$.

$$
\tau_{yaw} = -\gamma T_1+\gamma T_2- \gamma T_3+\gamma  T_4
$$

$$
\tau_{yaw} = \gamma (-T_1+T_2-T_3+T_4)\tag{3}
$$

### Defining Motor Mixer Matrix

Grouping the total thrust, roll torque, pitch torque, and yaw torque in the following order, we have:

$$
\begin{aligned}
T_\text{total} = T_1+T_2+T_3+T_4 \\
\tau_\text{roll} = L_y(-T_1-T_2+T_3+T_4) \\
\tau_\text{pitch}  =  L_x(-T_1+T_2+T_3-T_4) \\
\tau_\text{yaw} = \gamma (T_1-T_2+T_3-T_4)
\end{aligned}
$$

Representing these equations in matrix form, we get

$$
\begin{bmatrix} T_{total} \\ \tau_{roll} \\ \tau_{pitch} \\ \tau_{yaw} \end{bmatrix} = \begin{bmatrix} 1 & 1 & 1 & 1 \\ -L_y & -L_y & L_y & L_y \\ -L_x & L_x & L_x & -L_x \\ -\gamma & \gamma & -\gamma & \gamma \end{bmatrix} \begin{bmatrix} T_1 \\ T_2 \\ T_3 \\ T_4 \end{bmatrix}
$$

To obtain the individual thrust generated by each rotor, we invert the matrix:

$$
\begin{bmatrix} T_{1} \\ T_{2} \\ T_{3} \\ T_{4} \end{bmatrix} = \begin{bmatrix} \frac14 & -\frac{1}{4L_y} & -\frac{1}{4L_x} & -\frac{1}{4\gamma} \\ \frac14 & -\frac{1}{4L_y} & \frac{1}{4L_x} & \frac{1}{4\gamma} \\ \frac14 & \frac{1}{4L_y} & \frac{1}{4L_x} & -\frac{1}{4\gamma} \\ \frac14 & \frac{1}{4L_y} & -\frac{1}{4L_x} & \frac{1}{4\gamma} \end{bmatrix} \begin{bmatrix} T_{total} \\ \tau_{roll} \\ \tau_{pitch} \\ \tau_{yaw} \end{bmatrix}
$$

We can go one step further by normalizing the matrix. This is done by factoring the physical constants into the input vector and leaving the matrix entries bounded between -1 and +1.

$$
\begin{bmatrix} T_{1} \\ T_{2} \\ T_{3} \\ T_{4} \end{bmatrix} = \begin{bmatrix} 1 & -1 & -1 & -1 \\ 1 & -1 & 1 & 1 \\ 1 & 1 & 1 & -1 \\ 1 & 1 & -1 & 1 \end{bmatrix} \begin{bmatrix} \frac14T_{total} \\ \frac14\frac{\tau_{roll}}{L_y} \\ \frac14\frac{\tau_{pitch}}{L_x} \\ \frac14\frac{\tau_{yaw}}{\gamma} \end{bmatrix}
$$

At this point, we can see that the motor mixer matrix from the dynamic model is identical to the motor mixer in (1) from the iterative approach. Therefore, the dynamic derivation verifies that the iterative motor mixer is correct.
s

$$
\begin{bmatrix} \text{output motor 1} \\ \text{output motor 2} \\ \text{output motor 3} \\ \text{output motor 4} \end{bmatrix} = \begin{bmatrix} 1 & -1 & -1 & -1 \\ 1 & -1 & 1 & 1 \\ 1 & 1 & 1 & -1 \\ 1 & 1 & -1 & 1 \end{bmatrix} \begin{bmatrix} \text{throttle input} \\ \text{roll input} \\ \text{pitch input} \\ \text{yaw input} \end{bmatrix}
$$

In the PID controller design, the controller outputs the total thrust, roll torque, pitch torque, and yaw torque. We represent these quantities by the inputs $u_1$, $u_2$, $u_3$, and $u_4$, respectively. Hence, the motor mixer matrix becomes

$$
\begin{bmatrix} T_{1} \\ T_{2} \\ T_{3} \\ T_{4} \end{bmatrix} = \begin{bmatrix} 1 & -1 & -1 & -1 \\ 1 & -1 & 1 & 1 \\ 1 & 1 & 1 & -1 \\ 1 & 1 & -1 & 1 \end{bmatrix} \begin{bmatrix} \frac14u_1 \\ \frac14\frac{u_2}{L_y} \\ \frac14\frac{u_3}{L_x} \\ \frac14\frac{u_4}{\gamma} \end{bmatrix} \tag{8}
$$

Finally, by defining $\tilde{u}_1$, $\tilde{u}_2$, $\tilde{u}_3$, $\tilde{u}_4$, we can rewrite the mixer matrix (8) as

$$
\begin{bmatrix} T_{1} \\ T_{2} \\ T_{3} \\ T_{4} \end{bmatrix} = \begin{bmatrix} 1 & -1 & -1 & -1 \\ 1 & -1 & 1 & 1 \\ 1 & 1 & 1 & -1 \\ 1 & 1 & -1 & 1 \end{bmatrix} \begin{bmatrix} \tilde{u}_1 \\ \tilde{u}_2 \\ \tilde{u}_3 \\ \tilde{u}_4 \end{bmatrix}
$$

where

$$
    \tilde{u}_1 = \frac{1}{4}u_1,
    \qquad
    \tilde{u}_2 = \frac{1}{4L_{y}}u_2,
    \qquad
    \tilde{u}_3 = \frac{1}{4L_{x}}u_3,
    \qquad
    \tilde{u}_4 = \frac{1}{4\gamma}u_4
$$

## Rescaling PID gains

We observe that the final motor mixer implemented for this quadcopter does not depend on physical constants such as $\frac{1}{4}$, $L_x$, $L_y$ and $\gamma$. This is possible because these constants can be absorbed into the PID gains, allowing the tuning process to account for them automatically. Once this is done, the PID outputs become command inputs rather than physical quantities. This is why we use command inputs instead of thrust and torque in the final motor mixer.

To see how this PID gain rescaling works, let's consider the roll torque input $u_2$, defined as

$$
u_2 =\tau_{roll} = K_P e_{roll} + K_I \int e_{roll}\,dt + K_D \frac{de_{roll}}{dt}
$$

We know $\tilde{u}_2$ and by substituting $u_2$, we get

$$
\tilde{u}_2 = \frac{1}{4L_{y}}u_2 = \frac{1}{4L_{y}}
\left(
K_{P}e_{roll}
+
K_{I}\int e_{roll}\,dt
+
K_{D}\frac{d e_{roll}}{dt}
\right)
$$

Next, we define the following PID gains that absorb the physical constant $\frac{1}{4L_y}$:

$$
    \tilde{K}_{P} = \frac{1}{4L_{y}}K_{P},
    \qquad
    \tilde{K}_{I} = \frac{1}{4L_{y}}K_{I},
    \qquad
    \tilde{K}_{D} = \frac{1}{4L_{y}}K_{D}
$$

Then $\tilde{u}_2$ remains a PID controller, but with rescaled gains.

$$
    \tilde{u}_2
    =
    \tilde{K}_{P}e_{roll}
    +
    \tilde{K}_{I}\int e_{roll}\,dt
    +
    \tilde{K}_{D}\frac{d e_{roll}}{dt}
$$

Note that the same rescaling can be applied to the other inputs.

From this example, we see that we do not need to explicitly include the physical constants in the motor mixer because they are absorbed into the PID gains during the tuning process. As a result, the PID outputs become command inputs rather than physical quantities. The main drawback of this approach is that the PID gains must be retuned whenever the propellers, motors, or drone size change.

## Code Implementation

For my version of the flight controller, I implemented the motor mixer directly in its matrix form. I find this approach much easier to understand because it closely matches the mathematical derivation and helps me become more familiar with quadcopter terminology. The following C++ snippet completely summarizes the mixer, and it still amazes me that this implementation actually works and is able to fly a real quadcopter.

```cpp
    // Motors.h
    #define NUM_MOTORS 4
    float _command_inputs[4] = {0.0f};
    float _mixed_motor_outputs[NUM_MOTORS] = {0.0f};
    float _mixer[NUM_MOTORS][NUM_MOTORS] = {
        {1, -1, -1, -1}, // Motor 1
        {1, -1, 1, 1},   // Motor 2
        {1, 1, 1, -1},   // Motor 3
        {1, 1, -1, 1},   // Motor 4
    };
```

```cpp
    // Motors.cpp
    void Motors::set_command_inputs(float throttle_command, float roll_command, float pitch_command, float yaw_command)
    {
        _command_inputs[Input::THROTTLE] = throttle_command;
        _command_inputs[Input::ROLL] = roll_command;
        _command_inputs[Input::PITCH] = pitch_command;
        _command_inputs[Input::YAW] = yaw_command;

        if (_command_inputs[Input::THROTTLE] > SAFE_MAX_THROTTLE)
        {
            _command_inputs[Input::THROTTLE] = SAFE_MAX_THROTTLE;
        }
    }

    void Motors::compute_mixer_outputs()
    {
        for (int i = 0; i < NUM_MOTORS; i++)
        {
            float sum = 0.0f;
            for (int j = 0; j < NUM_MOTORS; j++)
            {
                sum += _mixer[i][j] * _command_inputs[j];
            }
            _mixed_motor_outputs[i] = sum;
        }
    }
```
