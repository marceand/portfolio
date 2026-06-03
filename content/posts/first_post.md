---
title: "Quadcopter: Motor Mixer Algorithm"
date: 2025-01-15T00:25:52-05:00
type: "post"
showTableOfContents: true
---

## Introduction

With a main goal of building a deep learning curve of the dynamic quadcopter, I built my own quadcopter based an the open source drone Carbon Quadcopter. For this I had to purchuse parts online, made a few necessary modification to assembling the whole quadcopter from ground up to the testing flight. Even thought it sounds there is an easy path from assembling to testing flight, I got stuck months in trying to fly it due to hardware or software issue, so the fun I was having little by little turn into frustration. From this frustration, I developed my skills to verify if certain implemented functionality is correct by applying physics and math, or just simulating the functionlaity in python using ideas that I accquired from quadcopter research papers.

With a main goal of building a deep learning curve of the dynamic quadcopter, I built my own quadcopter based an the open source drone Carbon Quadcopter. For this I had to purchuse parts online, made a few necessary modification to assembling the whole quadcopter from ground up to the testing flight. Even thought it sounds there is an easy path from assembling to testing flight, I got stuck months in trying to fly it due to hardware or software issue, so the fun I was having little by little turn into frustration. One of the issue I faced was that my quadcopter flipped at every take off. After going through an extensive troubleshooting, I was wrongly convinced that the issue was the motor mixer algorithm without realizing that there are other functionilies that cause this flip issue, until weeks later I found out the issue was caused by ESCs misscalibration but I already have learned so much about motor mixer algorithm to the point where i derived it for this quadcopter to confirm that the motor mixer is actually correct. This is my main purpose here to share with you the derivation of the motor mixer from the drone geometric layout with the assumtion you already know the basic of physic and linear algebra.

## What you learned

This gives us a clear idea of how popular flights controller handle motor mixers, and also this helps us to visually see with equations how must of the constant coffiecients are absorbed into the PID gains so that we only care about retunning the drone experimentially when its physical parameters (mass, arm length, propellers) are modified.

## Motor Mixer Algorithm

As shown in Fig.1, the motor mixer algoritm is an algorithm that mix the outputs of the controller, the thrust, roll torque, pitch torque and yaw torque, to produce a command for indivisual motor. Then we decide if we need to keep processing this commands using a thrust plot/graph before sending it to the ESC of each motor or sent the command as pwm directly to the ESCS. Thus this mixer algorithm determine the necessary thrust each motor need to produce to obtain the desired motion of the quadcopter.

## Find Motor Mixer Algorithm iteratively

The manual of Carbon Quadcopter do a great job at explaining the mixer in details and is given as a linear combination of throttle, roll, pitch and yaw input, as shown below. These inputs are provide directly by the controller. To find the correct combination we have to use the quadcopter configuration and go through every single movement case until the mixer algorithm is correct. This is very tedious iterative approach but avoid us from going thorough the dynamic mathermatically.

$$
\begin{aligned}
\text{output motor 1} &= \text{throttle input} - \text{roll input} - \text{pitch input} - \text{yaw input} \\
\text{output motor 2} &= \text{throttle input} - \text{roll input} + \text{pitch input} + \text{yaw input} \\
\text{output motor 3} &= \text{throttle input} + \text{roll input} + \text{pitch input} - \text{yaw input} \\
\text{output motor 4} &= \text{throttle input} + \text{roll input} - \text{pitch input} + \text{yaw input}
\end{aligned}
$$

For now, lets assume the motor mixer is just the summation of each input without considering the sign effect of each input, we find the sign of each input as we consider each case scenario.

$$
\begin{aligned}
\text{output motor 1} &= \text{throttle input} + \text{roll input} + \text{pitch input} + \text{yaw input} \\
\text{output motor 2} &= \text{throttle input} + \text{roll input} + \text{pitch input} + \text{yaw input} \\
\text{output motor 3} &= \text{throttle input} + \text{roll input} + \text{pitch input} + \text{yaw input} \\
\text{output motor 4} &= \text{throttle input} + \text{roll input} + \text{pitch input} + \text{yaw input}
\end{aligned}
$$

Let's take case scenario where the quadcopter is rolling to the right (Looking from top-rear view) with a rolling input of 25%. To achieve this sidewards movement, we assume the quadcopter is hovering at 50% input and ideally there is no pitch (0%) and yaw (0%) interfering with the roll movement. Therefore the above place holder mixers is only consist of throttle and roll input

$$
\begin{aligned}
\text{output motor 1} &= \text{throttle input} + \text{roll input} \\
\text{output motor 2} &= \text{throttle input} + \text{roll input} \\
\text{output motor 3} &= \text{throttle input} + \text{roll input} \\
\text{output motor 4} &= \text{throttle input} + \text{roll input}
\end{aligned}
$$

Looking at the drone configuration, we see that rolling to the right, we have to decrease motor 1 and 2 by 25% at the same time we have to increase motor 3 ad 4 by 25%. This means the roll input for mootor 1 and 2 have negative sign and roll input for motor 3 and 4 have positive sign. We end up with a mixer for rolling right for the four motor as below and if we plug in the values, we clearly see that two motors provide lower power and other two motors provide higher power to achieve the right side movements.

$$
\text{output motor 1} = \text{throttle input} - \text{roll input}
= 50\% - 25\% = 25\%
$$

$$
\text{output motor 2} = \text{throttle input} - \text{roll input}
= 50\% - 25\% = 25\%
$$

$$
\text{output motor 3} = \text{throttle input} + \text{roll input}
= 50\% + 25\% = 75\%
$$

$$
\text{output motor 4} = \text{throttle input} + \text{roll input}
= 50\% + 25\% = 75\%
$$

The next question is this mixer actually works for rolling to the left, where motor 1 and 2 need higher power and motor 3 and 4 need lower power. Clearly this still roll to the right if we provide a positive 10% power, so we must provide an input of negative 10% such that the sign of the input flip the signs to result in a roll to the left. Wit this in mind, the roll motor mixer actually make the quadcopter to roll to both right and left but the input must carry/have the correct sign to determine to which side the quadcopter moves.

$$
\begin{aligned}
\text{output motor 1} &= \text{throttle input} - \text{roll input} = 50\% - (-10\%) = 60\% \\
\text{output motor 2} &= \text{throttle input} - \text{roll input} = 50\% - (-10\%) = 60\% \\
\text{output motor 3} &= \text{throttle input} + \text{roll input} = 50\% + (-10\%) = 40\% \\
\text{output motor 4} &= \text{throttle input} + \text{roll input} = 50\% + (-10\%) = 40\%
\end{aligned}
$$

The same iterative approch hold for finding the motor mixer for pitch and yaw that's how the final motor mixer algorithm is obtained for this quadcopter.

## Find Motor Mixer Algorithm mathematically

A better method to find the motor mixer algorithm is derived it from the quadcopter motor coordinate geometric layout, then we apply the definition of total thrust and torque to form a matrix motor mixer, then we normalize this matrix to end up with a mixer algorithm as abtained with iterative approach.

it is known that the the thrust force is directly proportional to the square of the angular velocity of the rotor with a thrust coefficient $k\_{T}$:

$$
\begin{aligned}
T_i &= k_T \omega_i^2 \qquad i=1,2,3,4\tag{9}
\end{aligned}
$$

$$T\_{i}  = k\_{T}\omega\_{i} ^{2}\qquad i=1,2,3,4\tag{1}$$

Therefore, the total vertical thrust is the summation of the thrust contribute by each rotor:

$$
T\_{total} = T\_{1}+T\_{2}+T\_{3}+T\_{4}\tag{3}
$$

# Wrong sign, correct it

It is also known that there is a drag torque responsible for yaw rotation of the quadcopter which is also proportionnal to the square of the angular velocity of the the rotor with a moment coefficient $k\_{M}$, considering the sign of the torque is based on the propeller spinning direction, for clockwise propeller cause a negative yaw torque and counterclockwise propeller, produce a positive yaw torque.:
$$M\_{i} = k\_{M}\omega\_{i}^{2}\qquad i=1,2,3,4\tag{2}$$

For this quadcopter, using the table 1 that summarizes the sign of the yaw torque from each rotor,
therefore the total yaw torque is the summation of the drag torque produced by each rotor with its respective sign.

$$
\tau\_{yaw} = M\_{1}-M\_{2}+M\_{3}-M\_{4}\tag{3}
$$

{{<customtable>}}
_Table 1: Sign of yaw torque from each rotor_
| Rotor | Direction | Torque sign |
| :------ | :-------: | :---------: |
| motor 1 | CCW | +$M\_{1}$ |
| motor 2 | CW | -$M\_{2}$ |
| motor 3 | CCW | +$M\_{3}$ |
| motor 4 | CW | -$M\_{4}$ |
{{</customtable>}}

Using (1) and (2), we can represent $M$ in term of $T$ with $\gamma$ as the drag to thrust cofficient ratio, this will help us to easily manipulate all equation in term of thrust force $T$.

$$
M = \frac{k\_{M}}{k\_{T}} T
$$

Let $\gamma = \frac{k\_{M}}{k\_{T}}$, then

$$
M = \gamma*T
$$

With this substition, we can have the yawing torque in term of $T$ with $\gamma$

$$
\tau\_{yaw} = \gamma T\_{1}-\gamma T\_{2}+ \gamma T\_{3}-\gamma  T\_{4}
$$

$$
\tau\_{yaw} = \gamma (T\_{1}-T\_{2}+T\_{3}-T\_{4})\tag{3}
$$

We already have total thrust, yawing torque, but missing rolling and pitching torque. The rolling and pitching toque are obtained from the torque formula which is the cross product of vector lever arm and the vector force:

$$
\vec{\tau} = \vec{r} \times \vec{F}
$$

Once we do the cross production and separate the component for the x-axis and y-axis, the rolling torque and pitching torque are defined as

$$\tau_\text{roll} = \sum y_iT_i$$

$$\tau_\text{pitch}  = \sum(-x_iT_i)$$

The the IMU is rigidly attached to drone with its axis marked on its board and we take it as a reference for the body frame, in this way we know where each axis of body frame are oriented. For this quadcopter, the +x is pointing to the front whereas the +y is pointed to the left. With this axis in place, we can find the coordinates of each motors in the x-axis and y-axis as shown in figure 3. By plugginng in the corresping x-component and y component of each motor position into (5) and (6), the rolling toque and pitching torque are (6) and (7). Important to note that the coordinate of the motors depend on the quadcopter configuration, for example for the X-configuration each arm is phsically at a digonal of 45 degree.

$$
\begin{aligned}
\tau_\text{roll} = \sum y_iT_i = y_1T_1+y_2T_2+y_3T_3+y_4T_4 \\
\tau_\text{roll} = (-L_y)T_1+(-L_y)T_2+(L_y)T_3+(L_y)T_4 \\
\tau_\text{roll} = L_y(-T_1-T_2+T_3+T_4)
\end{aligned}
$$

$$
\begin{aligned}
\tau_\text{pitch}  = \sum(-x_iT_i) = -(x_1T_1+x_2T_2+x_3T_3+x_4T_4) \\
\tau_\text{pitch}  = -[(L_x)T_1+(-L_x)T_2+(-L_x)T_3+(L_x)T_4] \\
\tau_\text{pitch}  =  L_x(-T_1+T_2+T_3-T_4)
\end{aligned}
$$

Grouping in the correct order from total thrust, rolling, pitching and yawing torque, we get:

$$
\begin{aligned}
T_\text{total} = T_1+T_2+T_3+T_4 \\
\tau_\text{roll} = L_y(-T_1-T_2+T_3+T_4) \\
\tau_\text{pitch}  =  L_x(-T_1+T_2+T_3-T_4) \\
\tau_\text{yaw} = \gamma (T_1-T_2+T_3-T_4)
\end{aligned}
$$

Representing the above in a clean matrix form, we get:

$$
\begin{bmatrix} T_{total} \\ \tau_{roll} \\ \tau_{pitch} \\ \tau_{yaw} \end{bmatrix} = \begin{bmatrix} 1 & 1 & 1 & 1 \\ -L_y & -L_y & L_y & L_y \\ -L_x & L_x & L_x & -L_x \\ -\gamma & \gamma & -\gamma & \gamma \end{bmatrix} \begin{bmatrix} T_1 \\ T_2 \\ T_3 \\ T_4 \end{bmatrix}
$$

In order to get indivisual thrust for each rotor, we have to invert the matrix:

$$
\begin{bmatrix} T_{1} \\ T_{2} \\ T_{3} \\ T_{4} \end{bmatrix} = \begin{bmatrix} \frac14 & -\frac{1}{4L_y} & -\frac{1}{4L_x} & -\frac{1}{4\gamma} \\ \frac14 & -\frac{1}{4L_y} & \frac{1}{4L_x} & \frac{1}{4\gamma} \\ \frac14 & \frac{1}{4L_y} & \frac{1}{4L_x} & -\frac{1}{4\gamma} \\ \frac14 & \frac{1}{4L_y} & -\frac{1}{4L_x} & \frac{1}{4\gamma} \end{bmatrix} \begin{bmatrix} T_{total} \\ \tau_{roll} \\ \tau_{pitch} \\ \tau_{yaw} \end{bmatrix}
$$

We can go further by normalizing the matrix by factoring out the physical constants to the input vector and leave the entries bounded between -1 and +1. Finally this is the motor mixer algoritm obtained from H-configuration of the quadcopter where the row by row corresond to the motor out and the column represents throttle, roll, pitch and yaw.

$$
\begin{bmatrix} T_{1} \\ T_{2} \\ T_{3} \\ T_{4} \end{bmatrix} = \begin{bmatrix} 1 & -1 & -1 & -1 \\ 1 & -1 & 1 & 1 \\ 1 & 1 & 1 & -1 \\ 1 & 1 & -1 & 1 \end{bmatrix} \begin{bmatrix} \frac14T_{total} \\ \frac14\frac{\tau_{roll}}{L_y} \\ \frac14\frac{\tau_{pitch}}{L_x} \\ \frac14\frac{\tau_{yaw}}{\gamma} \end{bmatrix}
$$

Sames as the motor mixer from the iterative approach

$$
\begin{bmatrix} \text{output motor 1} \\ \text{output motor 2} \\ \text{output motor 3} \\ \text{output motor 4} \end{bmatrix} = \begin{bmatrix} 1 & -1 & -1 & -1 \\ 1 & -1 & 1 & 1 \\ 1 & 1 & 1 & -1 \\ 1 & 1 & -1 & 1 \end{bmatrix} \begin{bmatrix} \text{throttle input} \\ \text{roll input} \\ \text{pitch input} \\ \text{yaw input} \end{bmatrix}
$$

## PID Controller

When we design a real PID controller for the quadcopter, we relied on the controller to compute each input instead of calculating the throttle and torque using the physical values like mass, length, and coefficient. Therefore, we replace the input vector of the motor mixer with the controller innput u1, u2, u3 and u4. It is important to note that we deriver all the equations above using physical constants because we need them in simulation for further analysis.

$$
\begin{bmatrix} T_{1} \\ T_{2} \\ T_{3} \\ T_{4} \end{bmatrix} = \begin{bmatrix} 1 & -1 & -1 & -1 \\ 1 & -1 & 1 & 1 \\ 1 & 1 & 1 & -1 \\ 1 & 1 & -1 & 1 \end{bmatrix} \begin{bmatrix} u_{1} \\ u_{2} \\ u_{3} \\ u_{4} \end{bmatrix}
$$

To stabilize the quadcopter, we create a PID for each control input. With this approach, the PID only try to minimize the error to achieve a desired value. That's why we don't have a flight controller that depend on the quadcopter physical constant parameters such as mass and arm length. More specifically, these constants are absorbed into the PID gains. This means we only have to tune each PID since all connstant parameters are already handled in the gains. The only downside is that we have to retune experimentally the PID if one of these constant parameters change, for example increasing the mass or arm length.  
To actually discover that these constants are integrated into the gains of the PID, let build the PID for u2.
Let u2 = (1/4)(1/Ly)(torque_roll) as derived above, then we can have a PID to provide the torque for the roll:

$$
\tau_{roll} = K_P e_{roll} + K_I \int e_{roll}\,dt + K_D \frac{de_{roll}}{dt}
$$

Hence,

$$
u_{2} = \frac{1}{4L_{y}}
\left(
K_{P}e_{roll}
+
K_{I}\int e_{roll}\,dt
+
K_{D}\frac{d e_{roll}}{dt}
\right)
$$

We define the following PID gains that absorb the physical constant $\frac{1}{4L_y}$

$$
    \tilde{K}_{P} = \frac{1}{4L_{y}}K_{P},
    \qquad
    \tilde{K}_{I} = \frac{1}{4L_{y}}K_{I},
    \qquad
    \tilde{K}_{D} = \frac{1}{4L_{y}}K_{D}
$$

Then $u_{2}$ remains a PID controller, but with rescaled gains

$$
    u_{2}
    =
    \tilde{K}_{P}e_{roll}
    +
    \tilde{K}_{I}\int e_{roll}\,dt
    +
    \tilde{K}_{D}\frac{d e_{roll}}{dt}
$$

Therefore all we have to do just tune the gains of the PID of u2 without explicitly considering the physical constants since the gains already handle them.

Going back to the motor mixer algorithm, but what each input is actually representing? Lets take for example, u2=scale\*torque, we clearly see that u2 does not represent torque but a scaled thrust difference needed to produce roll, thus, we can say this controller inputs represents input commands that tell the actuators to achieve a desired output. With this in mind, it is also convinient to call them throtlle, roll, pitch and yaw as input command.

For my version of flight controller, I implemented this motor mixer algorithm as represent by its matrix form, this is much better for me to understand and be familiar with the quadcopter termenology, the following c++ snipet below completly summarize this mixer and it is amazing that it actually works and manage to fly this real quadcopter.

```cpp
    // Motors.h
    #define NUM_MOTORS 4
    float _command_inputs[NUM_MOTORS] = {0.0f};
    float _mixed_motor_outputs[NUM_MOTORS] = {0.0f};
    float _motor_outputs[NUM_MOTORS] = {1000.0f};
    float _mixer[4][4] = {
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

$$
\tau_{roll} = K_P e_{roll} + K_I \int e_{roll}\,dt + K_D \frac{de_{roll}}{dt}
$$

For this drone the commands from the mixer are the pwm signals sent directly to the ESCs limited between 1000us ad 1800us scaled according to the hardware pwm generator.

The meticulus manual of Carbon Quadcopter is designed to help beginners to quickly build a quadcopter but also there is open section where we can commit to find out why certain functionality works as is.
