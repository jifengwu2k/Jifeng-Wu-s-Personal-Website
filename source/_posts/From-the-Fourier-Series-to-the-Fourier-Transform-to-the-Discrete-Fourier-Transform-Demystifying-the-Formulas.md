---
title: " From the Fourier Series to the Fourier Transform to the Discrete Fourier Transform: Demystifying the Formulas"
date: 2023-09-04
updated: 2026-02-14
categories:
  - Theory, Data Structures, Algorithms, Programming Languages, Design Patterns
tags:
  - Essays
---

In realms as broad as electrical engineering, acoustics, optics, signal processing, quantum mechanics, and econometrics, the Fourier Series, Fourier Transform, and Discrete Fourier Transform play a pivotal role in analyzing signals by allowing us to decompose them into simpler components. Many articles present their formulas or dive into their intuition and applications. However, what seems to be missing is a blog post that explains the derivation of their formulas in a way that is both clear and accessible, requiring no more than a rudimentary understanding of calculus.

# Fourier Series

## Standard Form of the Fourier Series

Our journey begins with the [Fourier Series](https://en.wikipedia.org/wiki/Fourier_series) - a method to represent periodic functions as a sum of sine and cosine waves.

Let $x(t)$ be a periodic function with period $T$. The *standard form of the Fourier series* for $x(t)$ is given by:

$$x(t) = \frac{a_0}{2} + a_1 \cos{\frac{2\pi}{T} t} + b_1 \sin{\frac{2\pi}{T} t} + a_2 \cos{\frac{4\pi}{T} t} + b_2 \sin{\frac{4\pi}{T} t} + \dots $$

To solve for $a_0, a_1, b_1, \dots$, we first observe the .

Thus, we can multiply both sides of the equation by $cos{\frac{2k\pi}{T} t}$ or $\sin{\frac{2k\pi}{T} t}$ $(k \in \{0, 1, 2, \dots, n\})$, and then integrate over one period $[-\frac{T}{2}, \frac{T}{2})$ to obtain:

$$a_k = \frac{2}{T} \int_{-\frac{T}{2}}^{\frac{T}{2}}{x(t) \cos{\frac{2k\pi}{T} t} dt}$$

$$b_k = \frac{2}{T} \int_{-\frac{T}{2}}^{\frac{T}{2}}{x(t) \sin{\frac{2k\pi}{T} t} dt}$$

## Exponential Form of the Fourier Series

Using Euler's formula $e^{ix} = \cos{x} + i \sin{x}$, we can derive:

$$\cos{x} = \frac{e^{ix} + e^{-ix}}{2}$$

$$\sin{x} = -i \frac{e^{ix} - e^{-ix}}{2}$$

Substituting these representations of $\cos{x}$ and $\sin{x}$ into $a_k$ and $b_k$, we get:

$$a_k = \frac{2}{T} \int_{-\frac{T}{2}}^{\frac{T}{2}}{x(t) \frac{e^{i \frac{2k\pi}{T} t} + e^{-i \frac{2k\pi}{T} t}}{2} dt}$$

$$b_k = \frac{2}{T} \int_{-\frac{T}{2}}^{\frac{T}{2}}{-i x(t) \frac{e^{i \frac{2k\pi}{T} t} - e^{-i \frac{2k\pi}{T} t}}{2} dt}$$

And:

$$a_k \cos{\frac{2k\pi}{T} t} + b_k \sin{\frac{2k\pi}{T} t} = a_k \frac{e^{i \frac{2k\pi}{T} t} + e^{-i \frac{2k\pi}{T} t}}{2} - i b_k \frac{e^{i \frac{2k\pi}{T} t} - e^{-i \frac{2k\pi}{T} t}}{2} = \frac{a_k - i b_k}{2} e^{i \frac{2k\pi}{T} t} + \frac{a_k + i b_k}{2} e^{-i \frac{2k\pi}{T} t}$$

And:

$$\frac{a_k - i b_k}{2} = \frac{1}{T} \int_{-\frac{T}{2}}^{\frac{T}{2}}{x(t) e^{-i \frac{2k\pi}{T} t} dt}$$

$$\frac{a_k + i b_k}{2} = \frac{1}{T} \int_{-\frac{T}{2}}^{\frac{T}{2}}{x(t) e^{i \frac{2k\pi}{T} t} dt}$$

Furthermore, if we let:

$$c_k = \frac{a_k - i b_k}{2} = \frac{1}{T} \int_{-\frac{T}{2}}^{\frac{T}{2}}{x(t) e^{-i \frac{2k\pi}{T} t} dt}$$

Substituting $k \leftarrow -k$ into the expression for $c_k$, we will obtain:

$$c_{-k} = \frac{1}{T} \int_{-\frac{T}{2}}^{\frac{T}{2}}{x(t) e^{i \frac{2k\pi}{T} t} dt} = \frac{a_k + i b_k}{2}$$

Thus:

$$a_k \cos{\frac{2k\pi}{T} t} + b_k \sin{\frac{2k\pi}{T} t}  = \frac{a_k - i b_k}{2} e^{i \frac{2k\pi}{T} t} + \frac{a_k + i b_k}{2} e^{-i \frac{2k\pi}{T} t}
= c_k e^{i \frac{2k\pi}{T} t} + c_{-k} e^{-i \frac{2k\pi}{T} t}$$

And by substituting $k \leftarrow 0$ into the expression for $c_k$, we get:

$$c_0 = \frac{1}{T} \int_{-\frac{T}{2}}^{\frac{T}{2}}{x(t) dt} = \frac{a_0}{2}$$

Therefore:

$$x(t) = \frac{a_0}{2} + \sum_{k=1}^{n}{(a_k \cos{\frac{2k\pi}{T} t} + b_k \sin{\frac{2k\pi}{T} t})}  = c_0 + \sum_{k=1}^{n}{(c_k e^{i \frac{2k\pi}{T} t} + c_{-k} e^{-i \frac{2k\pi}{T} t})} = \sum_{k=-n}^{n}{c_k e^{i \frac{2k\pi}{T} t}}$$

Where:

$$c_k= \frac{1}{T} \int_{-\frac{T}{2}}^{\frac{T}{2}}{x(t) e^{-i \frac{2k\pi}{T} t} dt}$$

This is the *exponential form of the Fourier series*. It is more concise than the standard form of the Fourier series and is used more often in practice.

# Fourier Transform

The Fourier transform is a generalization of the Fourier series, which can analyze the effect of a frequency in *any function* (which may not necessarily be a periodic function). In this section, we will present how it can be derived from the exponential form of the Fourier series.

Given a periodic function $x(t)$ with period $T$, the exponential form of the Fourier series of $x(t)$ is as follows:

$$x(t) = \sum_{k=-n}^{n}{c_k e^{i \frac{2k\pi}{T} t}}$$

Where:

$$c_k= \frac{1}{T} \int_{-\frac{T}{2}}^{\frac{T}{2}}{x(t) e^{-i \frac{2k\pi}{T} t} dt}$$

Let's say that the period $T$ is associated with a frequency known as the [*fundamental frequency*](https://en.wikipedia.org/wiki/Fundamental_frequency) $f_0 = \frac{1}{T}$. Given $f_0$, we can rewrite the previous Fourier series as:

$$x(t) = \sum_{k=-n}^{n}{c_k e^{i 2\pi k f_0 t}}$$

Where:

$$c_k= \frac{1}{T} \int_{-\frac{T}{2}}^{\frac{T}{2}}{x(t) e^{-i 2\pi k f_0 t} dt}$$

For a non-periodic function, we can consider it as a periodic function with $T \rightarrow +\infty$. In this case, the fundamental frequency $f_0$ is an infinitesimal quantity; therefore, *we can consider that any frequency $f$ can be expressed as an integer multiple of the fundamental frequency, and the difference between two neighboring frequencies is the fundamental frequency $f_0$*. In this case, the fundamental frequency $f_0$ can be expressed as a differential of the frequency $f$, i.e., $df$.

In this case, for a possibly non-periodic function $x(t)$:

$$x(t) = \sum_{k=-\infty}^{\infty}{c_k e^{i 2\pi k (df)t}}$$

$$c_k= (df) \int_{-\infty}^{\infty}{x(t) e^{-i 2\pi k (df) t} dt}$$

Thus, $x(t)$ can be represented as:

$$x(t) = \sum_{k=-\infty}^{\infty}{[(df) \cdot \int_{-\infty}^{\infty}{x(t) e^{-i 2\pi k (df) t} dt} \cdot e^{i 2\pi k (df) t}]}$$

By considering $f \leftarrow k (df)$, [we can transform the summation into a definite integral](https://maninbocss.medium.com/summation-and-the-definite-integral-235663ef5ec3):

$$x(t)= \int_{-\infty}^{\infty}{ [(\int_{-\infty}^{\infty}{x(t) e^{-i 2\pi f t} dt}) e^{i 2\pi f t}] df}$$

Let:

$$X(f) = \int_{-\infty}^{\infty}{x(t) e^{-i 2\pi f t} dt}$$

Then $x(t)$ can be represented as:

$$x(t) = \int_{-\infty}^{\infty}{ X(f) e^{i 2\pi f t} df}$$

These two equations are very important.

- If we know $x(t)$ (i.e., the value of $x(t)$ at any time $t$), through $X(f) = \int_{-\infty}^{\infty}{x(t) e^{-i 2\pi f t} dt}$, we can compute *the relative magnitude of any frequency $f$ over the whole time period*.
- At the same time, if we know $X(f)$ (i.e., the relative magnitude of any frequency $f$ over the whole time period), by means of $x(t) = \int_{-\infty}^{\infty}{ X(f) e^{i 2\pi f t} df}$, we can calculate *the value of $x(t)$ at any time $t$*.

We refer to $X(f)$ as the *Fourier transform* of $x(t)$, also known as the *spectrum* of $x(t)$, and to $x(t)$ as the *inverse Fourier transform* of $X(f)$.

# Discrete Fourier Transform

When we process signals with computers, as computers cannot store a continuous infinite function, we usually take $N$ samples of the original signal $x(t)$ at a certain time interval $\Delta t$, obtaining an array $x[0:N-1]$.

Using $x[0:N-1]$ to estimate the Fourier transform $X(f)$ of the sampled function $x(t)$, we get:

$$X(f) = \int_{-\infty}^{\infty}{x(t) e^{-i 2\pi f t} dt} \approx \int_{0}^{N \Delta t}{x(t) e^{-i 2\pi f t} dt} \approx \sum_{m=0}^{N - 1}{x(m \Delta t) e^{-i 2\pi f m \Delta t}}$$

If we *assume these samples have spanned a period of the original signal, e.g. $T = N \Delta t$, and that we only consider frequencies satisfying $f = k \frac{1}{N \Delta t} (k \in \{0, 1, \dots, N - 1\})$*, we get:

$$X(k \frac{1}{N \Delta t}) \approx \sum_{n=0}^{N - 1}{x(n \Delta t) e^{-i 2\pi k \frac{1}{N \Delta t} n \Delta t}} = \sum_{n=0}^{N - 1}{x(n \Delta t) e^{-i 2\pi \frac{k}{N} n}} = \sum_{n=0}^{N - 1}{x[n] e^{-i 2\pi \frac{k}{N} n}}$$
  
Let:

$$X[k] = \sum_{n=0}^{N - 1}{x[n] e^{-i 2\pi \frac{k}{N} n}} (k \in \{0, 1, \dots, N - 1\})$$

We call such an array of $N$ discrete numbers $X[0:N-1]$ the *discrete Fourier transform* of $x[0:N-1]$, which is a *discrete* frequency domain representation of $x[0:N-1]$.

Using $X[0:N-1]$, we can restore $x[0:N-1]$:

$$x[n] = \frac{1}{N} \sum_{k=0}^{N - 1}{X[k] e^{i 2\pi \frac{k}{N} n}} (n \in \{0, 1, \dots, N - 1\})$$

We call $x[0:N-1]$ the *inverse discrete Fourier transform* of $X[0:N-1]$. This is analogous to $X(f)$ being the Fourier transform of $x(t)$ and $x(t)$ being the inverse Fourier transform of $X(f)$ in the continuous case.
