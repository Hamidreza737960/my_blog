
**Pricing European and American Options with the Trinomial Tree Model in Python**
=================================================================================

In this project, I implemented a trinomial tree model for pricing **European** and **American** call and put options. This model extends the binomial approach by allowing the asset price to move up, down, or stay the same at each step, leading to more accurate pricing in uncertain markets.

---

**📌 Model Overview**

We compute the fair price of options using a **recombining trinomial tree**. The asset price evolves by moving **up**, **down**, or **staying the same** at each step, under a **risk-neutral measure**.

---

**🧮 Parameters**

The model parameters:

```python
sigma = 0.2   # Volatility
K     = 100   # Strike price
S0    = 100   # Initial stock price
N     = 4     # Number of time steps
r     = 0.05  # Risk-free interest rate
T     = 1     # Time to maturity in years
d_t   = T / N # Time step size
```

---

**⬆️⬇️ Movement Factors**

We calculate the up/down multipliers:

\[
u = e^{\sigma \sqrt{2 \Delta t}}, \quad d = e^{-\sigma \sqrt{2 \Delta t}}
\]

```python
u = 1.151910
d = 0.868123
```

---

**🎲 Transition Probabilities**

Derived using a risk-neutral assumption:

\[
p_u = \left(\frac{e^{r\Delta t / 2} - e^{-\sigma\sqrt{\Delta t / 2}}}{e^{\sigma\sqrt{\Delta t / 2}} - e^{-\sigma\sqrt{\Delta t / 2}}}\right)^2
\]
\[
p_d = \left(\frac{e^{\sigma\sqrt{\Delta t / 2}} - e^{r\Delta t / 2}}{e^{\sigma\sqrt{\Delta t / 2}} - e^{-\sigma\sqrt{\Delta t / 2}}}\right)^2
\]
\[
p_m = 1 - p_u - p_d
\]

```python
p_u = 0.277334
p_d = 0.224084
p_m = 0.498582
```

---

**📈 Stock Price Tree**

A stock price matrix `S` of shape \((2N+1) \times (N+1)\) is constructed:

```python
S =
[[100.         115.191      132.68966481 152.84655179 176.06547147]
 [  0.         100.         115.191      132.68966481 152.84655179]
 ...
 [  0.           0.           0.           0.          56.79695464]]
```

---

**💵 European Option Pricing**

Prices are computed via backward induction:

```python
C = 
[[10.20508408 20.09256307 35.15865143 54.08875897 76.06547147]
 [ 0.          8.49086296 18.30584026 33.93187367 52.84655179]
 ...
 [ 0.          0.          0.          0.          0.        ]]

P = 
[[ 5.32807446  1.22103717  0.          0.          0.        ]
 ...
 [ 0.          0.          0.          0.         43.20304536]]
```

---

**🇺🇸 American Option Pricing**

Allows early exercise — take the max of intrinsic and expected discounted value:

```python
CA = 
[[10.20508408 20.09256307 35.15865143 54.08875897 76.06547147]
 ...
 [ 0.          0.          0.          0.          0.        ]]

PA = 
[[ 5.32807446  1.22103717  0.          0.          0.        ]
 ...
 [ 0.          0.          0.          0.         43.20304536]]
```

---

**✅ Key Results**

| Option Type       | Price at \( t = 0 \) |
|-------------------|------------------------|
| European Call     | 10.21                  |
| European Put      | 5.33                   |
| American Call     | 10.21                  |
| American Put      | 5.33                   |

> As expected, American and European call prices match (no dividends).
