---
title: Fourier revision
description: Quick revision for Fourier series and transforms
pubDate: 2026-11-24
author: shubhajeet-das
tags:
  - signals
  - fourier
  - exam-tips
subject: AI31203
cover: cover.png
coverAlt: Sine waves at odd harmonics adding up to an approximate square wave
draft: false
---

Since my freshman year, I kept hearing professors talk about **Fourier series** and **Fourier transforms** as if they were magic tools that unlocked half of engineering and physics. I always wanted to learn them properly—and this semester, I finally did.

Fourier analysis is a foundational pillar across communications, control systems, image processing, and electrodynamics. To preserve everything I learned (and build a quick revision guide before exams), I have divided this post into two parts:

1. **My Last-Minute Formula Sheet:** Transform pairs that aren't immediately obvious and take precious minutes to derive under exam pressure.
2. **Subtle but Important Insights:** The patterns, memory tricks, edge cases, and connections to differential/difference equations that truly make the theory click.

*(If you want to build real fluency with these concepts, working through the problems in* ***Oppenheim & Willsky's Signals and Systems*** *is hands-down the best way to get comfortable.)*

---

## Part I: Last-Minute Exam Formula Sheet

When you are walking into an exam hall, you don't want to burn ten minutes integrating sinc functions or impulse trains from scratch. Keep these essential transform pairs at your fingertips:

| Signal $x(t)$ or $x[n]$ | Fourier Transform $X(j\omega)$ or $X(e^{j\omega})$ | Notes / Conditions |
| :--- | :--- | :--- |
| $\displaystyle \sum_{n=-\infty}^{\infty} \delta(t - nT)$ | $\displaystyle \frac{2\pi}{T} \sum_{k=-\infty}^{\infty} \delta\left(\omega - \frac{2\pi k}{T}\right)$ | Periodic impulse train (CTFS coeffs: $a_k = \frac{1}{T}$) |
| $x(t) = \begin{cases} 1, & \lvert t \rvert < T_1 \\ 0, & \lvert t \rvert > T_1 \end{cases}$ | $\displaystyle \frac{2\sin(\omega T_1)}{\omega}$ | Rectangular pulse $\leftrightarrow$ sinc function |
| $\displaystyle \frac{\sin(Wt)}{\pi t}$ | $X(j\omega) = \begin{cases} 1, & \lvert \omega \rvert < W \\ 0, & \lvert \omega \rvert > W \end{cases}$ | Ideal low-pass filter (by duality!) |
| $e^{j\omega_0 t}$ | $2\pi\,\delta(\omega - \omega_0)$ | Complex exponential |
| $\cos(\omega_0 t)$ | $\pi\left[\delta(\omega - \omega_0) + \delta(\omega + \omega_0)\right]$ | Even real sinusoid |
| $\sin(\omega_0 t)$ | $\dfrac{\pi}{j}\left[\delta(\omega - \omega_0) - \delta(\omega + \omega_0)\right]$ | Odd real sinusoid |
| $e^{-at}u(t)$ | $\dfrac{1}{a + j\omega}$ | $\operatorname{Re}\lbrace a \rbrace > 0$ |
| $a^n u[n]$ | $\dfrac{1}{1 - a e^{-j\omega}}$ | $\lvert a \rvert < 1$ (discrete-time geometric decay) |

![Continuous-Time Fourier Transform Pair: Rectangular Pulse and Sinc Function](media/fig1_rect_sinc_pair.png)

---

## Part II: Subtle but Important Insights

### 1. The Four Fourier Representations (and How to Never Confuse Them)

Depending on whether a signal is **continuous or discrete** and **periodic or aperiodic**, we use one of four representations. Following **Oppenheim's convention**:

- **Continuous-Time Fourier Series (CTFS)** — periodic, continuous-time signals, with $T = \frac{2\pi}{\omega_0}$:

  $$
  x(t) = \sum_{k=-\infty}^{\infty} a_k e^{j k \frac{2\pi}{T} t}, \qquad a_k = \frac{1}{T} \int_{T} x(t) e^{-j k \frac{2\pi}{T} t} \, dt
  $$

- **Discrete-Time Fourier Series (DTFS)** — periodic, discrete-time signals, with $N = \frac{2\pi}{\Omega_0}$:

  $$
  x[n] = \sum_{k=\langle N \rangle} a_k e^{j k \frac{2\pi}{N} n}, \qquad a_k = \frac{1}{N} \sum_{n=\langle N \rangle} x[n] e^{-j k \frac{2\pi}{N} n}
  $$

- **Continuous-Time Fourier Transform (CTFT)** — aperiodic, continuous-time signals:

  $$
  X(j\omega) = \int_{-\infty}^{\infty} x(t) e^{-j\omega t} \, dt, \qquad x(t) = \frac{1}{2\pi} \int_{-\infty}^{\infty} X(j\omega) e^{j\omega t} \, d\omega
  $$

- **Discrete-Time Fourier Transform (DTFT)** — aperiodic, discrete-time signals:

  $$
  X(e^{j\omega}) = \sum_{n=-\infty}^{\infty} x[n] e^{-j\omega n}, \qquad x[n] = \frac{1}{2\pi} \int_{2\pi} X(e^{j\omega}) e^{j\omega n} \, d\omega
  $$

> **My Memory Trick for Signs & Scaling Factors:**
>
> 1. **Where does the scaling factor go?** For **series** (CTFS, DTFS), the normalization factor ($\frac{1}{T}$ or $\frac{1}{N}$) sits in the **analysis** equation (computing $a_k$). For **transforms** (CTFT, DTFT), the $\frac{1}{2\pi}$ factor always sits in the **synthesis** (inverse) equation.
> 2. **What about the sign in the exponent?** Going *forward* into the frequency domain (analysis) **always** has a **negative** sign ($e^{-j\cdots}$). Coming *back* to the time domain (synthesis) **always** has a **positive** sign ($e^{+j\cdots}$).

**Bonus — Duality Saves Integrals:** Notice the symmetry between the forward and inverse CTFT:

$$
x(t) \overset{\mathcal{F}}{\longleftrightarrow} X(j\omega) \implies X(jt) \overset{\mathcal{F}}{\longleftrightarrow} 2\pi\,x(-\omega)
$$

Whenever you encounter a nasty integral involving a $\frac{\sin(\cdot)}{(\cdot)}$ term in the time domain, you can skip the integration entirely by applying duality to a rectangular pulse in the frequency domain!

---

### 2. Periodicity, Discreteness, and Finite-Duration Signals

A deep structural symmetry governs all four transforms:

- **Discrete in one domain $\iff$ Periodic in the other domain.**
- **Continuous in one domain $\iff$ Aperiodic in the other domain.**

| Representation | Time Domain | Frequency Domain |
| :--- | :--- | :--- |
| **CTFS** | Continuous & Periodic (period $T$) | Discrete & Aperiodic ($a_k$) |
| **DTFS** | Discrete & Periodic (period $N$) | Discrete & Periodic (period $N$) |
| **CTFT** | Continuous & Aperiodic | Continuous & Aperiodic ($X(j\omega)$) |
| **DTFT** | Discrete & Aperiodic | Continuous & Periodic (period $2\pi$) |

#### Applying Fourier Series to Finite-Duration Signals

Technically, CTFS and DTFS apply only to periodic signals. However, we can also use them to represent a **finite-duration signal** defined on an interval $[0, T]$.

How? Construct a **periodic avatar** $\tilde{x}(t)$ by replicating copies of the finite signal end-to-end across $(-\infty, \infty)$, compute its Fourier series coefficients, and simply restrict your attention back to $[0, T]$ (ignoring the replicated copies outside). This exact technique shows up in electrostatics when solving **Laplace's equation in Cartesian coordinates** (see *Griffiths' Introduction to Electrodynamics*, Example 3.3).

![Periodic Extension of a Finite-Duration Signal](media/fig2_periodic_extension.png)

---

### 3. Orthogonality ("The Fourier Trick") & Roots of Unity

Every Fourier analysis equation relies on the orthogonality of complex exponentials—often affectionately called **the Fourier trick**:

1. **Continuous-time periodic exponentials:**

   $$
   \int_{T} e^{j(k - m)\frac{2\pi}{T} t} \, dt = T\,\delta[k - m]
   $$

2. **Continuous-time aperiodic exponentials (Dirac delta representation):**

   $$
   \int_{-\infty}^{\infty} e^{-j\omega t} \, dt = 2\pi\,\delta(\omega)
   $$

3. **Discrete-time periodic exponentials (roots of unity):**

   $$
   \sum_{n=\langle N \rangle} e^{j(k - m)\frac{2\pi}{N} n} = N\,\delta[(k - m) \bmod N]
   $$

Geometrically, for $k - m \not\equiv 0 \pmod N$, the terms $e^{j\frac{2\pi}{N}(k-m)n}$ are symmetric vectors around the unit circle in the complex plane—the **$N$-th roots of unity**—whose vector sum is always zero (for example, when $N = 3$ and $\omega = e^{j2\pi/3}$, $1 + \omega + \omega^2 = 0$).

---

### 4. The Dirac Delta Function & Generalized Derivatives

Just as a point charge at the origin produces a 3D delta divergence in electrostatics ($\vec{\nabla} \cdot \left(\frac{\hat{r}}{r^2}\right) = 4\pi\,\delta^{(3)}(\vec{r})$), the 1D Dirac impulse $\delta(t)$ models instantaneous events in signal processing. Keep three properties in mind:

1. **Sampling (Sifting) Property:** $x(t)\delta(t - t_0) = x(t_0)\delta(t - t_0)$
2. **Convolution Identity:** $x(t) * \delta(t - t_0) = x(t - t_0)$
3. **Time Scaling:** $\delta(at) = \dfrac{1}{\lvert a \rvert}\delta(t)$

#### Graphical Differentiation Rule

To differentiate a piecewise-smooth signal $x(t)$ with jump discontinuities:

- At every **jump discontinuity** at $t = t_0$, place a Dirac delta $\Delta x \cdot \delta(t - t_0)$ whose weight equals the jump height $\Delta x = x(t_0^{+}) - x(t_0^{-})$.
- Between discontinuities, take the **ordinary slope**.

**Worked Example:** Consider a signal $x(t)$ that is $0$ for $t < 0$, jumps to $2$ at $t = 0$, stays constant at $2$ on $(0, 1)$, slopes linearly down to $0$ on $(1, 2)$, jumps down to $-1$ at $t = 2$, slopes linearly up to $0$ on $(2, 3)$, and remains $0$ for $t > 3$. Differentiating piece by piece gives:

$$
\frac{dx(t)}{dt} = 2\delta(t) - \delta(t - 2) - 2\left[u(t - 1) - u(t - 2)\right] + \left[u(t - 2) - u(t - 3)\right]
$$

![Generalized Derivative of a Piecewise-Linear Signal with Jumps](media/fig3_dirac_derivative.png)

---

### 5. Impulse-Train Sampling, Aliasing, & CTFS via CTFT

![Series transform summary](media/transform_summary.jpeg)

#### Why Sampling Replicates the Spectrum

To sample a continuous-time signal $x(t)$ every $T_s$ seconds ($\omega_s = \frac{2\pi}{T_s}$), we multiply it by a periodic impulse train $p(t) = \sum_{n=-\infty}^{\infty} \delta(t - nT_s)$:

$$
x_p(t) = x(t)p(t) = \sum_{n=-\infty}^{\infty} x(nT_s)\delta(t - nT_s)
$$

Multiplication in time becomes convolution in frequency (scaled by $\frac{1}{2\pi}$):

$$
X_p(j\omega) = \frac{1}{2\pi}\big(X(j\omega) * P(j\omega)\big) = \frac{1}{2\pi} \int_{-\infty}^{\infty} X(j\theta) P(j(\omega - \theta)) \, d\theta
$$

Substituting $P(j\omega) = \frac{2\pi}{T_s} \sum_{k=-\infty}^{\infty} \delta\left(\omega - \frac{2\pi k}{T_s}\right)$ shifts copies of $X(j\omega)$ to every integer multiple of $\omega_s$:

$$
X_p(j\omega) = \frac{1}{T_s} \sum_{k=-\infty}^{\infty} X\left(j\left(\omega - \frac{2\pi k}{T_s}\right)\right)
$$

If $x(t)$ is bandlimited to $\lvert \omega \rvert < \omega_M$, adjacent copies remain cleanly separated (allowing perfect recovery via a low-pass filter) **if and only if** the **Nyquist criterion** holds:

$$
\omega_s > 2\omega_M
$$

If $\omega_s < 2\omega_M$, adjacent spectral copies overlap and corrupt each other—a distortion known as **aliasing**.

![Impulse-Train Sampling and the Nyquist Criterion]

#### Finding Periodic CTFS Coefficients by Sampling a Single-Pulse CTFT

Suppose you want the CTFS coefficients $a_k$ of a periodic square wave $x(t)$ with period $T$ and pulse width $2T_1$ (centered at $t = 0$). Instead of doing a separate series integral, take the **CTFT of a single period** $x_1(t)$:

$$
X_1(j\omega) = \int_{-T_1}^{T_1} 1 \cdot e^{-j\omega t} \, dt = \left[ \frac{e^{-j\omega t}}{-j\omega} \right]_{-T_1}^{T_1} = \frac{e^{-j\omega T_1} - e^{j\omega T_1}}{-j\omega} = \frac{2\sin(\omega T_1)}{\omega}
$$

Comparing this directly with $a_k = \frac{1}{T}\int_{-T/2}^{T/2} x(t) e^{-j k \omega_0 t}\,dt$ (where $\omega_0 = \frac{2\pi}{T}$), we see that **$a_k$ is simply $\frac{1}{T}$ times $X_1(j\omega)$ sampled at $\omega = k\omega_0$**:

$$
a_k = \frac{1}{T} X_1\left(j\frac{2\pi k}{T}\right) = \frac{\sin\left(\frac{2\pi k T_1}{T}\right)}{\pi k} \quad (k \neq 0), \qquad a_0 = \frac{2T_1}{T}
$$


#### Two Practical Exam Notes on Series & Integration

- **Geometric Series vs. Poisson Summation:** The standard geometric series $\sum_{n=0}^{\infty} z^n = \frac{1}{1 - z}$ converges **only when $\lvert z \rvert < 1$**. On the unit circle ($z = e^{-j\omega T}$, so $\lvert z \rvert = 1$), the doubly infinite sum does not converge to an ordinary function—instead, it yields a Dirac comb via the **Poisson summation identity**:

  $$
  \sum_{n=-\infty}^{\infty} e^{-j\omega T n} = \frac{2\pi}{T} \sum_{k=-\infty}^{\infty} \delta\left(\omega - \frac{2\pi k}{T}\right)
  $$

- **When Complex Integration Gets Messy:** Contour integration using Cauchy's residue theorem isn't always necessary (or easy) under time pressure. A safe, reliable fallback is to expand $e^{-j\omega t} = \cos(\omega t) - j\sin(\omega t)$ and integrate the real and imaginary parts separately.

---

### 6. Connection to LTI Systems & Difference Equations

Any discrete-time signal can be written as a weighted sum of shifted impulses:

$$
x[n] = \sum_{k=-\infty}^{\infty} x[k]\delta[n - k]
$$

Passing $x[n]$ through a **Linear Time-Invariant (LTI)** system $\mathcal{L}\lbrace\cdot\rbrace$ with impulse response $h[n] = \mathcal{L}\lbrace\delta[n]\rbrace$ gives:

$$
\begin{aligned}
y[n] &= \mathcal{L}\left\lbrace\sum_{k=-\infty}^{\infty} x[k]\delta[n - k]\right\rbrace \\
&\overset{\text{Linearity}}{=} \sum_{k=-\infty}^{\infty} x[k]\,\mathcal{L}\lbrace\delta[n - k]\rbrace \\
&\overset{\text{Time-Invariance}}{=} \sum_{k=-\infty}^{\infty} x[k]\,h[n - k] = x[n] * h[n]
\end{aligned}
$$

Convolution in the time domain transforms into **simple algebraic multiplication** in the frequency domain:

$$
Y(e^{j\omega}) = X(e^{j\omega})H(e^{j\omega}) \quad \text{(and in continuous time: } Y(j\omega) = X(j\omega)H(j\omega)\text{)}
$$

#### Worked Example: Solving a Difference Equation via DTFT

Find the causal impulse response $h[n]$ of an LTI system governed by:

$$
y[n] - \frac{1}{6}y[n - 1] - \frac{1}{6}y[n - 2] = x[n]
$$

1. **Take the DTFT of both sides** using the time-shift property ($y[n - k] \leftrightarrow e^{-j\omega k}Y(e^{j\omega})$):

   $$
   Y(e^{j\omega})\left(1 - \frac{1}{6}e^{-j\omega} - \frac{1}{6}e^{-j2\omega}\right) = X(e^{j\omega})
   $$

2. **Solve for the frequency response $H(e^{j\omega}) = \frac{Y(e^{j\omega})}{X(e^{j\omega})}$ and factor the denominator:**

   $$
   H(e^{j\omega}) = \frac{1}{1 - \frac{1}{6}e^{-j\omega} - \frac{1}{6}e^{-j2\omega}} = \frac{1}{\left(1 - \frac{1}{2}e^{-j\omega}\right)\left(1 + \frac{1}{3}e^{-j\omega}\right)}
   $$

3. **Apply partial fraction expansion:**

   $$
   H(e^{j\omega}) = \frac{3/5}{1 - \frac{1}{2}e^{-j\omega}} + \frac{2/5}{1 + \frac{1}{3}e^{-j\omega}}
   $$

4. **Take the inverse DTFT** (using $\frac{1}{1 - a e^{-j\omega}} \leftrightarrow a^n u[n]$):

   $$
   h[n] = \left[\frac{3}{5}\left(\frac{1}{2}\right)^n + \frac{2}{5}\left(-\frac{1}{3}\right)^n\right] u[n]
   $$

That is the real beauty of Fourier analysis: differential and difference equations become algebra, convolutions become products, and continuous-to-discrete sampling becomes intuitive geometry in the frequency domain.

You can also check out my video here: https://youtu.be/oqefVaRAkt4

Spotted a mistake or have a favourite trick that is missing? Use the comments below, or edit this post on GitHub.
