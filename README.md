# NewUU Engineering Internship — Senior Track Task 2 Report

**Candidate:** Ozodbek Azimurodov  
**Institution:** School of Engineering, New Uzbekistan University  
**Task:** Training a Locomotion Policy in Isaac Lab (Newton Backend) & Investigating Command Range Changes  

---

## 1. Pre-Experiment Prediction (Question 1)

Before modifying the command range and running the wide-range training experiments, our theoretical hypothesis was structured as follows:

* **Sample Space Dilution:** Expanding the linear velocity ranges ($\text{lin\_vel\_x}$ and $\text{lin\_vel\_y}$) from $\pm 1.0\text{ m/s}$ to $\pm 2.0\text{ m/s}$ quadruples the 2D command space area ($4\text{ m}^2/\text{s}^2$ to $16\text{ m}^2/\text{s}^2$). Over a fixed training duration of 300 iterations, the agent receives significantly fewer samples per unit of velocity subspace.
* **Reward Sparsity & Vanishing Gradients:** The tracking reward relies on an exponential kernel ($\exp(-\Vert{}v_{\text{cmd}} - v_{\text{actual}}\Vert{}^2 / \sigma^2)$). When a policy with near-zero initial speed is commanded to hit $2.0\text{ m/s}$ straight out of initialization, the velocity error is extremely large, causing the reward signal to collapse toward zero and stalling gradient propagation early on.
* **Biomechanical & Stability Limits:** Forcing aggressive lateral movements ($\pm 2.0\text{ m/s}$ in $y$) and high-speed diagonal/backward trajectories pushes the Unitree Go2 beyond its natural trotting stability envelope. This creates a surge in tracking error (`error_vel_xy`) and trunk contact terminations (`base_contact`).

---

## 2. Experimental Results & TensorBoard Metrics (Question 2)

We evaluated three configurations using TensorBoard:
1. **Baseline [-1, 1] (300 Iterations):** Clean convergence with low tracking error and high survival rate[cite: 2, 4].
2. **Wide Range [-2, 2] (300 Iterations):** Suffered from severe under-fitting and instability due to the expanded command space.
3. **Wide Range [-2, 2] (900 Iterations / 3x):** Allowed extended learning time to recover policy stability under aggressive commands.

| Metric | Part A: Baseline [-1, 1] (300 iters) | Part B: Wide [-2, 2] (300 iters) | Part B: Wide [-2, 2] (900 iters - 3x) |
|---|---|---|---|
| **`error_vel_xy`** | ~0.18[cite: 4] | ~1.53[cite: 4] | ~1.49[cite: 4] |
| **`base_contact`** | ~0.09[cite: 3] | ~0.08 - 0.11[cite: 3] | ~0.046[cite: 3] |
| **`success_rate` / `time_out`** | ~0.98[cite: 2] | ~0.018[cite: 2] | ~0.024[cite: 2] |

---

## 3. Analysis: Did More Training Iterations Fix It? (Question 3)

* **What 3x iterations successfully resolved:** Extending training to 900 iterations drastically improved trunk stability. As observed in the `base_contact` metric, failures dropped down to $\sim 4.6\%$[cite: 3], outperforming even the baseline. This demonstrates that **sample starvation / under-fitting** was a major bottleneck in the 300-iteration wide run—the agent simply needed more temporal exposure to learn robust recovery reflexes under extreme commands.
* **What 3x iterations did NOT resolve:** The velocity tracking error (`error_vel_xy`) remained high near $\sim 1.49$[cite: 4]. This rules out training duration as the sole root cause and highlights a **fundamental physical constraint**: a standard quadruped trotting policy cannot physically sustain high-speed lateral sprinting ($\pm 2.0\text{ m/s}$ sideways) without specialized gait transitions or a velocity curriculum.

---

## 4. Repository Structure & Deliverables

* `part_a_baseline.pt` — Baseline model checkpoint (300 iters)
* `part_a_baseline.mp4` — Newton visualizer screen recording of baseline policy
* `part_b_wide_1x.pt` — Wide-range model checkpoint (300 iters)
* `part_b_wide_1x.mp4` — Newton visualizer screen recording of wide-range policy (1x)
* `part_b_wide_3x.pt` — Wide-range model checkpoint (900 iters)
* `part_b_wide_3x.mp4` — Newton visualizer screen recording of wide-range policy (3x)
* `tensorboard/` — Captured performance graphs (`error_vel_xy.png`, `base_contact.png`, `success_rate.png`)

[the link to videos](https://drive.google.com/drive/folders/1gRo6jGme5pEoyssPtsX95Cx6V419z-kf?usp=sharing)