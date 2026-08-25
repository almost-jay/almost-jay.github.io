Here is a classic orbital mechanics question where ignoring the Oberth effect leads to a massive calculation error.

---

## The Scenario: Escaping Earth's Gravity

> **Question:**
> A spacecraft is in Low Earth Orbit (LEO) at an altitude where its orbital speed is $v = 7{,}800 \text{ m/s}$ and Earth's local escape velocity is $v_{\text{esc}} = 11{,}000 \text{ m/s}$.
> The craft fires its main engine to deliver a burn of **$\Delta v = 4{,}000 \text{ m/s}$**.
> **What will the spacecraft’s final departure speed ($v_\infty$) be once it has completely escaped Earth's gravity well?**

---

## Approach 1: WITHOUT Accounting for Oberth (Naive Model)

A simple flat calculation treats $\Delta v$ like a bank balance. It assumes you first use $3{,}200 \text{ m/s}$ of your burn just to reach escape velocity ($11{,}000 - 7{,}800 = 3{,}200$), and whatever is left over is your final cruise speed:

$$v_\infty = \Delta v - (v_{\text{esc}} - v)$$

$$v_\infty = 4{,}000 \text{ m/s} - (11{,}000 \text{ m/s} - 7{,}800 \text{ m/s})$$

$$v_\infty = 4{,}000 - 3{,}200 = \mathbf{800 \text{ m/s}}$$

* **Naive Answer:** $\mathbf{800 \text{ m/s}}$ (or $0.8 \text{ km/s}$)

---

## Approach 2: WITH Oberth Effect (Energy Conservation Model)

Because the engine fires at a high initial speed ($7{,}800 \text{ m/s}$), the craft converts its fuel mass into kinetic energy at maximum efficiency.

### Step 1: Calculate post-burn velocity at periapsis ($v_{\text{burn}}$)

$$v_{\text{burn}} = v + \Delta v = 7{,}800 + 4{,}000 = 11{,}800 \text{ m/s}$$

### Step 2: Use the hyperbolic excess velocity formula

$$v_\infty = \sqrt{v_{\text{burn}}^2 - v_{\text{esc}}^2}$$

$$v_\infty = \sqrt{(11{,}800)^2 - (11{,}000)^2}$$

$$v_\infty = \sqrt{139{,}240{,}000 - 121{,}000{,}000} = \sqrt{18{,}240{,}000}$$

* **Oberth-Correct Answer:** $\mathbf{4{,}271 \text{ m/s}}$ (or $4.27 \text{ km/s}$)

---

## Comparison of Results

| Model | Predicted Final Speed ($v_\infty$) | Error |
| --- | --- | --- |
| **Without Oberth** | $800 \text{ m/s}$ | Off by **$81.3\%$** |
| **With Oberth** | **$4{,}271 \text{ m/s}$** | Accurate physics |

### Why the difference is so dramatic:

Notice that the Oberth-correct answer ($4{,}271 \text{ m/s}$) is **greater than the raw burn itself** ($4{,}000 \text{ m/s}$).

Without factoring in the Oberth effect, you would miscalculate the final speed by more than **$3.4 \text{ km/s}$**, leading to a complete mission failure when planning interplanetary trajectories to Mars or Jupiter.