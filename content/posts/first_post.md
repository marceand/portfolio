---
title: "Quadcopter: Motor Mixer Algorithm"
date: 2025-01-15T00:25:52-05:00
type: "post"
showTableOfContents: true
---

## Introduction

With a main goal of building a deep learning curve of the dynamic quadcopter, I built my own quadcopter based an the open source drone Carbon Quadcopter. For this I had to purchuse parts online, made a few necessary modification to assembling the whole quadcopter from ground up to the testing flight. Even thought it sounds there is an easy path from assembling to testing flight, I got stuck months in trying to fly it due to hardware or software issue, so the fun I was having little by little turn into frustration. From this frustration, I developed my skills to verify if certain implemented functionality is correct by applying physics and math, or just simulating the functionlaity in python using ideas that I accquired from quadcopter research papers.

One of the issue I faced was that my quadcopter flipped at every take off. After going through an extensive troubleshooting, I was wrongly convinced that the issue was the motor mixer algorithm without realizing that there are other functionilies that cause this flip issue, until weeks later I found out the issue was caused by ESCs misscalibration but I already have learned so much about motor mixer algorithm to the point where i derived it for this quadcopter to confirm that the motor mixer is actually correct. This is my main purpose here to share with you the derivation of the motor mixer from the drone geometric layout with the assumtion you already know the basic of physic and linear algebra.

## What you learned

This gives us a clear idea of how popular flights controller handle motor mixers, and later for the next post, this helps us to visually see with equations how must of the constant coffiecients are absorbed into the PID gains so that we only care about retunning the drone experimentially when its physical parameters (mass, arm length, propellers) are modified.

## Motor Mixer Algorithm

As shown in Fig.1, the motor mixer algoritm is an algorithm that mix the outputs of the controller, the thrust, roll torque, pitch torque and yaw torque, to produce a command for indivisual motor. Then we decide if we need to keep processing this commands using a thrust plot/graph before sending it to the ESC of each motor or sent the command as pwm directly to the ESCS. Thus this mixer algorithm determine the necessary thrust each motor need to produce to obtain the desired motion of the quadcopter.

## Find Motor Mixer Algorithm iteratively

## Find Motor Mixer Algorithm mathematically

A better method to find the motor mixer algorithm is derived it from the quadcopter motor coordinate geometric layout, then we apply the definition of total thrust and torque to form a matrix motor mixer, then we normalize this matrix to end up with a mixer algorithm as abtained with iterative approach.

it is known that the the thrust force is directly proportional to the square of the angular velocity of the rotor with a thrust coefficient $k\_{T}$:
$$T\_{i}  = k\_{T}\omega\_{i} ^{2}\qquad i=1,2,3,4\tag{1}$$

Therefore, the total vertical thrust is the summation of the thrust contribute by each rotor:

$$
T\_{total} = T\_{1}+T\_{2}+T\_{3}+T\_{4}\tag{3}
$$

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
$$\tau\_{x} = \sum()$$

$$\tau\_{y}  = \sum()$$

For this drone the commands from the mixer are the pwm signals sent directly to the ESCs limited between 1000us ad 1800us scaled according to the hardware pwm generator.

The meticulus manual of Carbon Quadcopter is designed to help beginners to quickly build a quadcopter but also there is open section where we can commit to find out why certain functionality works as is.
