
**Pricing European and American Options with the Trinomial Tree Model in Python**
=================================================================================

In this project, I implemented a trinomial tree model for pricing **European** and **American** call and put options. This model extends the binomial approach by allowing the asset price to move up, down, or stay the same at each step, leading to more accurate pricing in uncertain markets.


---


**Model Overview**

We compute the fair price of options using a **recombining trinomial tree**. The asset price evolves by moving **up**, **down**, or **staying the same** at each step, under a **risk-neutral measure**.

---


**Parameters**

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

**Movement Factors**

We calculate the up/down multipliers:

$$
\int_0^\infty e^{-x^2} dx = \frac{\sqrt{\pi}}{2}
$$


```python
u = 1.151910
d = 0.868123
```

---

**🎲 Transition Probabilities**

Derived using a risk-neutral assumption:

$$ 
p_u = \left(\frac{e^{r\Delta t / 2} - e^{-\sigma\sqrt{\Delta t / 2}}}{e^{\sigma\sqrt{\Delta t / 2}} - e^{-\sigma\sqrt{\Delta t / 2}}}\right)^2
$$ 
$$ 
p_d = \left(\frac{e^{\sigma\sqrt{\Delta t / 2}} - e^{r\Delta t / 2}}{e^{\sigma\sqrt{\Delta t / 2}} - e^{-\sigma\sqrt{\Delta t / 2}}}\right)^2
$$ 
$$ 
p_m = 1 - p_u - p_d
$$ 

```python
p_u = 0.277334
p_d = 0.224084
p_m = 0.498582
```

---

**Stock Price Tree**

A stock price matrix `S` of shape (2N+1)*(N+1)  is constructed:

```python
def stock_price_trinom(S0, N, u, d):
    
    """
    Function that calculates the stock prices in the trinomial model
    
    Input: S0 -> Initial Stock Price
           u  -> upward movement
           d  -> downward movement
           N  -> Number of time periods
       
    Output: S -> Upper triangular matrix of Stock prices with tirnomial model
    """
    arr = np.zeros((2*N+1,N+1))
    for col in range(N + 1):
        for i in range(2*col + 1):
            j = max(0 , i - col)
            k = max(col - i, 0)
            arr[i   , col] = S0*(u**(k))*(d**(j))
    return arr    
```

this yields the upper triangular matrix simulating the development of price after 4 time steps

```python
[[100.         115.191      132.68966481 152.84655179 176.06547147]
 [  0.         100.         115.191      132.68966481 152.84655179]
 [  0.          86.8123     100.         115.191      132.68966481]
 [  0.           0.          86.8123     100.         115.191     ]
 [  0.           0.          75.36375431  86.8123     100.        ]
 [  0.           0.           0.          75.36375431  86.8123    ]
 [  0.           0.           0.          65.42500849  75.36375431]
 [  0.           0.           0.           0.          65.42500849]
 [  0.           0.           0.           0.          56.79695464]]
```

---

**💵 European Option Pricing**

Prices are computed via backward induction and based on :

```python
def priceTriEuro(N, K, r, S, p_u, p_d, p_m, d_t):
    """
    Function that calculates the European option price using trinomial model
    
    Input: N     -> Number of time periods
           K     -> Strike price
           r     -> Interst rate
           S     -> Upper triangular matrix
           p_u   -> probability for upward movement
           p_d   -> probability for downward movement
           p_m   -> probability for no change
           d_t   -> time step size
       
    Output: C -> European Call option 
            P -> European Put option 

    """
    # Add code here
    # European call option matrix
    C = np.zeros((2*N+1 , N+1))
    P = np.zeros((2*N+1 , N+1))
    C[: , N] = np.maximum(S[: , N] - K , 0) # payoff functions in the last step
    P[: , N] = np.maximum(K - S[: , N] , 0) # payoff functions in the last step
    for i in range(N - 1  , -1, -1):
        for j in range(2*i + 1):
            C[j , i] = math.exp(-r*d_t)*(p_u*C[j , i + 1] + p_m*C[j + 1 , i+1] + p_d*C[j+2 , i+ 1])
            P[j , i] = math.exp(-r*d_t)*(p_u*P[j , i+ 1] + p_m*P[j + 1 , i+ 1] + p_d*P[j+2 , i+ 1])
    return C , P , C[0 , 0] , P[0 , 0]
```

resulting into calculating the european put and call option prices.

```python
C = 
 [[10.20508408 20.09256307 35.15865143 54.08875897 76.06547147]
 [ 0.          8.49086296 18.30584026 33.93187367 52.84655179]
 [ 0.          2.35494685  6.54952838 16.43321032 32.68966481]
 [ 0.          0.          1.13955485  4.16064631 15.191     ]
 [ 0.          0.          0.          0.          0.        ]
 [ 0.          0.          0.          0.          0.        ]
 [ 0.          0.          0.          0.          0.        ]
 [ 0.          0.          0.          0.          0.        ]
 [ 0.          0.          0.          0.          0.        ]]


P = 

 [[ 5.32807446  1.22103717  0.          0.          0.        ]
 [ 0.          4.81034308  0.64585257  0.          0.        ]
 [ 0.         11.86210434  4.08054774  0.          0.        ]
 [ 0.          0.         11.858255    2.91844307  0.        ]
 [ 0.          0.         22.1672414  11.94548265  0.        ]
 [ 0.          0.          0.         23.39402799 13.1877    ]
 [ 0.          0.          0.         33.33277352 24.63624569]
 [ 0.          0.          0.          0.         34.57499151]
 [ 0.          0.          0.          0.         43.20304536]]
```

---

**🇺🇸 American Option Pricing**

Allows early exercise — take the max of intrinsic and expected discounted value:

```python
def priceTriAmer(N, K, r, S, p_u, p_d, p_m, d_t):
    """
    Function that calculates the American option price using trinomial model
    
    Input: N     -> Number of time periods
           K     -> Strike price
           r     -> Interst rate
           S     -> Upper triangular matrix
           p_u   -> probability for upward movement
           p_d   -> probability for downward movement
           p_m   -> probability for no change
           d_t   -> time step size
       
    Output: C -> American Call option 
            P -> American Put option 

    """
    C = np.zeros((2*N+1 , N+1))
    P = np.zeros((2*N+1 , N+1))
    C[: , N] = np.maximum(S[: , N] - K , 0)
    P[: , N] = np.maximum(K - S[: , N] , 0)
    for i in range(N - 1  , -1, -1):
        for j in range(2*i + 1):
            C_temp = math.exp(-r*d_t)*(p_u*C[j , i + 1] + p_m*C[j + 1 , i+1] + p_d*C[j+2 , i+ 1])
            C[j , i] = np.maximum(S[j , i] - K , C_temp)
            P_temp = math.exp(-r*d_t)*(p_u*P[j , i+ 1] + p_m*P[j + 1 , i+ 1] + p_d*P[j+2 , i+ 1])
            P[j , i] = np.maximum(K  - S[j , i], P_temp)
    return C , P , C[0 , 0] , P[0 , 0]
```

resulting into calculating the american put and call option prices.

```python
CA = 
 [[10.20508408 20.09256307 35.15865143 54.08875897 76.06547147]
 [ 0.          8.49086296 18.30584026 33.93187367 52.84655179]
 [ 0.          2.35494685  6.54952838 16.43321032 32.68966481]
 [ 0.          0.          1.13955485  4.16064631 15.191     ]
 [ 0.          0.          0.          0.          0.        ]
 [ 0.          0.          0.          0.          0.        ]
 [ 0.          0.          0.          0.          0.        ]
 [ 0.          0.          0.          0.          0.        ]
 [ 0.          0.          0.          0.          0.        ]]


PA = 

 [[ 5.84960493  1.28187335  0.          0.          0.        ]
 [ 0.          5.23990894  0.64585257  0.          0.        ]
 [ 0.         13.1877      4.35545092  0.          0.        ]
 [ 0.          0.         13.1877      2.91844307  0.        ]
 [ 0.          0.         24.63624569 13.1877      0.        ]
 [ 0.          0.          0.         24.63624569 13.1877    ]
 [ 0.          0.          0.         34.57499151 24.63624569]
 [ 0.          0.          0.          0.         34.57499151]
 [ 0.          0.          0.          0.         43.20304536]]
```

---

**Key Results**

```python

 N    E_P_ex    E_P_app      err_E_P    A_P_ex    A_P_app    err_A_P
----  --------  ---------  -----------  --------  ---------  ---------
   4    5.5735    5.32804  0.245458       6.0624    5.84958  0.212825
  16    5.5735    5.51129  0.0622119      6.0624    6.03113  0.0312749
  64    5.5735    5.55792  0.0155814      6.0624    6.07702  0.0146224
 256    5.5735    5.56962  0.00387872     6.0624    6.08734  0.0249399
 512    5.5735    5.57157  0.00192659     6.0624    6.08891  0.0265064
1024    5.5735    5.57255  0.000950343    6.0624    6.08965  0.0272517

```

| Option Type       | Price at t = 0 |
|-------------------|------------------------|
| European Call     | 10.21                  |
| European Put      | 5.33                   |
| American Call     | 10.21                  |
| American Put      | 5.33                   |

> As expected, American and European call prices match (no dividends).
