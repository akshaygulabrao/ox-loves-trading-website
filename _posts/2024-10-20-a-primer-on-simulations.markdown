---
layout: post
title: "Getting Started with Modeling & Simulation"
date: 2024-10-20
categories: [robotics]
---

Modeling and simulation allow for the exploration and testing of algorithms in a virtual environment, providing quick, cheap, and immediate feedback. In this post, I will walk you through a mission scenario from start to finish of designing a missile guidance system that allows a missile to converge on a moving enemy target. By the end of this system, I will have a robust algorithm capable of seeking and destroying an enemy moving target.

# 2D environment fixed enemy movement

The state transition matrix $\mathbf{A}$ is defined as:

$$
\mathbf{A} = \begin{bmatrix}
1 & 0 & 1 & 0 \\
0 & 1 & 0 & 1 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

- **State Vector**: The state vector for each entity (ally or enemy) is represented as $\mathbf{x} = [x_{\text{position}}, y_{\text{position}}, x_{\text{velocity}}, y_{\text{velocity}}]$

- **Matrix Structure**:
  - The first row $[1, 0, 1, 0]$ updates the $x$-position by adding the $x$-velocity.
  - The second row $[0, 1, 0, 1]$ updates the $y$-position by adding the $y$-velocity.
  - The third and fourth rows $[0, 0, 1, 0]$ and $[0, 0, 0, 1]$ ensure that the velocities remain unchanged during the state transition.

In the simulation loop, the state transition matrix is used to update the positions of the ally and enemy:

$$
\mathbf{x}_{\text{next}} = \mathbf{A} \cdot \mathbf{x}
$$

This operation updates the position components of the state vector by adding the respective velocity components, effectively simulating linear motion.

Here the ally converges on the enemy target by computing where the enemy would be assuming it's velocity was fixed and taking a step towards that direction.

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

It's not enough to verify that the algorithm works just inside a game loop. It's common practice to write an animation to visualize performance. The repository with code to create animations are at [akshaygulabrao/motionplanning](https://github.com/akshaygulabrao/motion-planning).

<video width="640" height="360" controls>
    <source src="/videos/version0.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

# Change in angle constraints
Sharp changes in angle are unrealistic in missile guidance systems. Suppose we don't allow more than a 12.5 degree shift in the ally's direction during each timestep. Let's plan a velocity and angle throttling mechanism to keep the missile on a feasible trajectory. The most obvious way to accomplish this is to throttle the speed by the amount the angle heading needs to change by. That way we still respect all the timestep constraints, but we are able to change directions in less steps.
