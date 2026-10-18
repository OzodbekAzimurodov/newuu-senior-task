# NewUU Engineering Internship — Senior Track Task 2 Report

**Candidate:** Ozodbek Azimurodov  
**Institution:** School of Engineering, New Uzbekistan University  
**Task:** Training a Locomotion Policy in Isaac Lab (Newton Backend) & Investigating Command Range Changes  

---

## 1. Pre-Experiment Prediction (Question 1)

Before modifying the command range and running the wide-range training experiments, our theoretical hypothesis was structured as follows:

* **Sample Space Dilution:** Expanding the linear velocity ranges (`lin_vel_x` and `lin_vel_y`) from ±1.0 m/s to ±2.0 m/s quadruples the 2D command space area (4 m²/s² to 16 m²/s²). Over a fixed training duration of 300 iterations, the agent receives significantly fewer samples per unit of velocity subspace.
* **Reward Sparsity & Vanishing Gradients:** The tracking reward relies on an exponential kernel ($\exp(-\Vert{}v_{\text{cmd}} - v_{\text{actual}}\Vert{}^2 / \sigma^2)$). When a policy with near-zero initial speed is commanded to hit 2.0 m/s straight out of initialization, the velocity error is extremely large, causing the reward signal to collapse toward zero and stalling gradient propagation early on.
* **Biomechanical & Stability Limits:** Forcing aggressive lateral movements (±2.0 m/s in $y$) and high-speed diagonal/backward trajectories pushes the Unitree Go2 beyond its natural trotting stability envelope. This creates a surge in tracking error (`error_vel_xy`) and trunk contact terminations (`base_contact`).

---

## 2. Experimental Results & TensorBoard Metrics (Question 2)

We evaluated three configurations using TensorBoard:
1. **Baseline [-1, 1] (300 Iterations):** Clean convergence with low tracking error and high survival rate.
2. **Wide Range [-2, 2] (300 Iterations):** Suffered from severe under-fitting and instability due to the expanded command space.
3. **Wide Range [-2, 2] (900 Iterations / 3x):** Allowed extended learning time to recover policy stability under aggressive commands.

| Metric | Part A: Baseline [-1, 1] (300 iters) | Part B: Wide [-2, 2] (300 iters) | Part B: Wide [-2, 2] (900 iters - 3x) |
|---|---|---|---|
| **`error_vel_xy`** | ~0.18 | ~1.53 | ~1.49 |
| **`base_contact`** | ~0.09 | ~0.08 - 0.11 | ~0.046 |
| **`success_rate` / `time_out`** | ~0.98 | ~0.018 | ~0.024 |

---

## 3. Analysis: Did More Training Iterations Fix It? (Question 3)

* **What 3x iterations successfully resolved:** Extending training to 900 iterations drastically improved trunk stability. As observed in the `base_contact` metric, failures dropped down to ~4.6%, outperforming even the baseline. This demonstrates that **sample starvation / under-fitting** was a major bottleneck in the 300-iteration wide run—the agent simply needed more temporal exposure to learn robust recovery reflexes under extreme commands.
* **What 3x iterations did NOT resolve:** The velocity tracking error (`error_vel_xy`) remained high near ~1.49. This rules out training duration as the sole root cause and highlights a **fundamental physical constraint**: a standard quadruped trotting policy cannot physically sustain high-speed lateral sprinting (±2.0 m/s sideways) without specialized gait transitions or a velocity curriculum.

---

## 4. Repository Structure & Deliverables

* `part_a_baseline.pt` — Baseline model checkpoint (300 iters)
* `part_a_baseline.mp4` — Newton visualizer screen recording of baseline policy
* `part_b_wide_1x.pt` — Wide-range model checkpoint (300 iters)
* `part_b_wide_1x.mp4` — Newton visualizer screen recording of wide-range policy (1x)
* `part_b_wide_3x.pt` — Wide-range model checkpoint (900 iters)
* `part_b_wide_3x.mp4` — Newton visualizer screen recording of wide-range policy (3x)
* `tensorboard/` — Captured performance graphs (`error_vel_xy.png`, `base_contact.png`, `success_rate.png`)

[Video Demonstrations & Recordings](https://drive.google.com/drive/folders/1gRo6jGme5pEoyssPtsX95Cx6V419z-kf?usp=sharing)