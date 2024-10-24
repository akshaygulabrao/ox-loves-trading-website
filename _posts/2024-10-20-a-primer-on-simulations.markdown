---
layout: post
title: "4 DOF Dynamics"
date: 2024-10-20
categories: [robotics]
---

Modeling and simulation allow for the exploration and testing of algorithms in a virtual environment, providing quick, cheap, and immediate feedback. In this post, I will walk you through a mission scenario from start to finish of designing a missile guidance system that allows a missile to converge on a moving enemy target. By the end of this system, I will have a robust algorithm capable of seeking and destroying an enemy moving target. Note that we are allowed to set the velocity directly which isn't realistic, but we are going to relax this assumption a bit later into the series. In most flight dynamic models, we only get to control the acceleration.

# 2D environment fixed enemy movement

The state transition matrix $\textbf{A}$ is defined as:

$$
\mathbf{A} = \begin{bmatrix}
1 & 0 & 1 & 0 \\
0 & 1 & 0 & 1 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

- **State Vector**: The state vector for each entity (ally or enemy) is represented as $\mathbf{x} = [x,y,\dot{x}, \dot{y}]$

- **Matrix Structure**:
  - The first row $[1, 0, 1, 0]$ updates the $x$-position by adding the $x$-velocity.
  - The second row $[0, 1, 0, 1]$ updates the $y$-position by adding the $y$-velocity.
  - The third and fourth rows $[0, 0, 1, 0]$ and $[0, 0, 0, 1]$ ensure that the velocities remain unchanged during the state transition.

In the simulation loop, the state transition matrix is used to update the positions of the ally and enemy:

$$
\mathbf{x}_{\text{next}} = \mathbf{A} \cdot \mathbf{x}
$$

This operation updates the position components of the state vector by adding the respective velocity components, effectively simulating linear motion.

Here the ally converges on the enemy target by computing where the enemy would be assuming it's velocity was fixed and taking a step towards that direction. The ally and enemy have speed constraints, with the ally's maximum speed constrained to 20 units and the enemy's maximum speed constrained to 10 units.

```python
import numpy as np


ALLY_MAX_SPEED = 20
ENEMY_MAX_SPEED = 10

ally_speed = ALLY_MAX_SPEED
ally_heading = 0 * np.pi / 180
ally = np.array([0,0,0,0])
ally[2:] = ally_speed * np.array([np.cos(ally_heading), np.sin(ally_heading)])

enemy_heading = 180 * np.pi / 180
enemy_speed = ENEMY_MAX_SPEED
enemy = np.array([1000,1000,0,0])
enemy[2:] = enemy_speed * np.array([np.cos(enemy_heading), np.sin(enemy_heading)])

state_transition = np.eye(4)
state_transition[0, 2] = 1
state_transition[1, 3] = 1

for i in range(62):
    enemy_next = state_transition @ enemy
    dir2next = enemy_next[:2] - ally[:2]
    ally_heading = np.arctan2(dir2next[1], dir2next[0])
    if ally_speed > np.linalg.norm(dir2next):
        ally_speed = np.linalg.norm(dir2next)
    else:
        ally_speed = ALLY_MAX_SPEED
    ally[2:] = ally_speed * np.array([np.cos(ally_heading), np.sin(ally_heading)])
    if np.linalg.norm(ally[:2] - enemy[:2]) < 1:
        print(f"Goal reached in {i} steps")
        break
    ally = state_transition @ ally
    enemy = state_transition @ enemy
```

First we define the following terms:
- $x_t$ is x's state at time t
- $x_t(v)$ is x's velocity at time t (both components)
- $x_t(p)$ is x's position at time t (both components)
- $a$ is the ally
- $b$ is the enemy
- $\theta$ and the speed $v$ as follows:

$$\theta = \text{atan2}(\text{next}_y - \text{ally}_y, \text{next}_x - \text{ally}_x)$$

$$
a_{t+1}(v) = \begin{cases}
\|d\| & \text{if } \|d\| < v_{max} \\
v_{max} & \text{if } \|d\| \geq v_{max}
\end{cases}
$$

where:
- $\text{next}$ is $\textbf{A} \cdot \text{enemy}$
- $\lVert d \rVert$ is the magnitude of $[\dot{x}, \dot{y}]$

It's not enough to verify that the algorithm works just inside a game loop. It's common practice to write an animation to visualize performance. The repository with code to create animations are at [akshaygulabrao/motionplanning](https://github.com/akshaygulabrao/motion-planning).

<video width="640" height="360" controls>
    <source src="/videos/version0.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

# Change in angle constraints
Suppose we wanted the enemy to stay in the field of view at all times. With a 12.5° FoV, the ally must stay far enough from the drone that the angle created by the enemy and their predicted next position does not exceed 12.5°. To apply the constraint, compute the amount the angle difference between the ally's current heading and the enemy's current location. Then, scale the speed of the ally down s.t. we change the angle at an appropriate speed such that the distance traveled is the same. **Note** The ally will accidentally escape the field of view of the enemy which we will fix in the next part.

In this step, we define a new term:
- $\Theta$ be the maximum allowable change in angle at any given timestep

Let $\tilde{x}$ be the vector from the ally's current position to the enemy's predicted next position:

$$
\tilde{x} = b_{t+1}(p) - a_t(p)
$$

Let $\phi$ be the angle between the ally's current velocity and $\tilde{x}$:

$$
\phi = \cos^{-1}\left(\frac{a_t(v) \cdot \tilde{x}}{\lVert a_t(v) \rVert \lVert \tilde{x} \rVert}\right)
$$

Then we can set:

$$
a_{t+1}(\theta) = \begin{cases}
a_t(\theta) + \phi &\text{if}& |\phi| < \Theta\\
a_t(\theta) + \Theta  &\text{if}& |\phi| \ge \Theta\\
\end{cases}
$$

and 

$$
a_{t+1}(v) = \begin{cases}
\|d\| & \text{if } \|d\| < v_{max} \\
v_{max} & \text{if } \|d\| \geq v_{max} \text{ and } |\phi| < \Theta \\
\max(v_{min}, v_{max} \cdot \frac{\Theta}{|\phi|}) & \text{if } |\phi| \geq \Theta
\end{cases}
$$

Note that the speed cases are not strictly disjoint in certain circumstances. Only 1 is valid at any given time and the first part that meets the given condition is what the next state should be. 


And the ally's state at $t+1$

$$[x + v\cos\theta, y + v\sin\theta, v\cos\theta, v\sin\theta]$$

The pseudocode for this version is 

```python
    current_velocity = ally[2:]
    desired_velocity = enemy_next[:2] - ally[:2]
    
    cos_angle = dot(current_velocity, desired_velocity) / (norm(current_velocity) * norm(desired_velocity))
    angle_diff = np.arccos(np.clip(cos_angle, -1.0, 1.0))
    
    angle_diff *= np.sign(np.cross(current_velocity, desired_velocity))

    if abs(angle_diff) > MAX_ANGLE_CHANGE:
        ally_heading = ally_heading + MAX_ANGLE_CHANGE * np.sign(angle_diff)
    else:
        ally_heading = ally_heading + angle_diff

    if ally_speed > norm(desired_velocity):
        ally_speed = norm(desired_velocity)
    elif abs(angle_diff) > MAX_ANGLE_CHANGE:
        ally_speed = max(ALLY_MIN_SPEED, ALLY_MAX_SPEED / (abs(angle_diff) / MAX_ANGLE_CHANGE))
    else:
        ally_speed = ALLY_MAX_SPEED

    ally[2:] = ally_speed * np.array([np.cos(ally_heading), np.sin(ally_heading)])
```
Here is the next step of our dynamics model in action, respecting a maximum angle change constraint
<video width="640" height="640" controls>
    <source src="/videos/version1.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>