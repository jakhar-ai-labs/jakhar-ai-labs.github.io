

layout: post
title: "The Illusion of Chaos: Why Random Functions Always Repeat"
date: 2026-10-13
categories: programming computer-science data-science
tags: [algorithms, python, prng, math, machine-learning]
---

If you ask a computer to pick five random numbers, it feels like pure chance. But behind the scenes, computers are strictly deterministic machines: given the exact same starting state and instructions, they will always produce the exact same outcome.

So how does a `random()` function exist at all? And why does initializing it with a specific integer produce the identical sequence every single time?

---

## 1. True Randomness vs. Pseudo-Randomness

Computers cannot "guess" or flip an actual coin. Unless connected to specialized hardware that captures physical environmental noise (such as thermal fluctuation or atmospheric noise), standard programming languages use **Pseudo-Random Number Generators (PRNGs)**.

A PRNG is a mathematical formula that takes an initial input value (called a **seed**) and performs arithmetic operations to compute the next value. Because the math is fixed:

> If the starting seed is the same, the output sequence is 100% predictable.

---

## 2. Seeing It in Action

Let's demonstrate this in Python using the standard `random` module:

```python
import random

# Run 1: Setting seed to 42
random.seed(42)
first_run = [random.randint(1, 100) for _ in range(5)]
print("First Run: ", first_run)

# Run 2: Resetting the seed back to 42
random.seed(42)
second_run = [random.randint(1, 100) for _ in range(5)]
print("Second Run:", second_run)

```

### The Output:

```text
First Run:  [82, 15, 4, 95, 36]
Second Run: [82, 15, 4, 95, 36]

```

No matter what machine you run this on, setting `random.seed(42)` in Python produces the exact same sequence: `[82, 15, 4, 95, 36]`.

---

## 3. Why the Algorithm Itself Guarantees Repeatability

To understand why repeating a seed reproduces the exact sequence, look at what the algorithm actually is: a **Deterministic Finite State Machine**.

A PRNG does not invent numbers out of thin air. It maintains an internal **state vector** and updates it using fixed mathematical functions:

* **State Transition ($f$):** A deterministic function that turns the current state into the next state: $S_{n+1} = f(S_n)$.
* **Output Transform ($g$):** A function that extracts a usable integer from that state: $\text{Output}_n = g(S_n)$.

Because both $f$ and $g$ are mathematical functions with zero external inputs, **giving them the same state will always yield the exact same next state and output**.

The **seed** is simply the initialization value used to set up the very first state ($S_0$). Once $S_0$ is locked in, the entire sequence is predetermined.

---

### The Real-World Workhorse: The Mersenne Twister (MT19937)

Modern languages like Python, Ruby, PHP, and C++ (`std::mt19937`) default to the **Mersenne Twister**.

Here is how its algorithm works under the hood:

1. **State Expansion:** You feed it one integer (e.g., `42`), and the algorithm's initialization routine mathematically expands that seed into an internal state array of **624 integers (19,937 bits)**.
2. **Bit Twisting:** When you request numbers, it scrambles those bits using bitwise XORs (`^`), bit-shifts (`>>`, `<<`), and matrix operations ("twisting").
3. **Period Length:** It generates a cycle of $2^{19937} - 1$ numbers before it ever repeats.

Step 1 is a fixed mathematical formula. Passing the integer `42` runs the exact same bitwise initialization routine, generating the exact same 624-integer internal state every single time.

---

### Inspecting the Internal State in Python

You can inspect this state array directly to verify this:

```python
import random

# Seed with 42
random.seed(42)
state_1 = random.getstate()

# Seed with 42 again
random.seed(42)
state_2 = random.getstate()

print("Are internal states identical?", state_1 == state_2)
print("First 3 state values:", state_1[1][:3])

```

**Output:**

```text
Are internal states identical? True
First 3 state values: (2147483648, 1162985472, 3962657805)

```

---

## 4. What Happens When You Don't Provide a Seed?

If you call `random.randint(1, 100)` without specifying a seed, the runtime automatically picks one from an unpredictable entropy source:

* High-precision OS system clock time (down to nanoseconds).
* System entropy pools (`/dev/urandom` on Unix-like systems).

Because the system clock or entropy changes constantly, each run starts at a different state.

---

## 5. Why Do Data Scientists Use Random Seeds with a Fixed Integer?

If randomness is meant to be unpredictable, why do data scientists explicitly pass a fixed integer like `random_state=42` or `torch.manual_seed(0)` in almost every script?

In machine learning and data science, model performance depends heavily on randomized operations:

* Splitting datasets into training and testing sets.
* Initializing neural network weights before gradient descent.
* Selecting bootstrap samples and features in ensemble models (e.g., Random Forests).

If you don't fix the seed, every run produces a different dataset split and different initial weights. That introduces noise: if your model's accuracy jumps from 85% to 88%, **did your model actually improve, or did you just get a lucky train/test split?**

```python
from sklearn.model_selection import train_test_split

# Fixing random_state ensures every researcher gets the exact same train/test split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

```

By fixing the seed to a particular integer, data scientists ensure:

1. **Scientific Reproducibility:** Anyone who clones your code or runs your notebook gets the exact same model weights, evaluation metrics, and curves.
2. **Fair Experimentation (A/B Testing):** You isolate genuine architecture and hyperparameter improvements from random initialization luck.
3. **Deterministic Debugging:** If a pipeline fails on a specific edge-case batch during training, you can rerun the exact sequence to reproduce and debug the error.

---

## A Note on Security

Because PRNG outputs are deterministic and deduceable once enough values are observed, **standard PRNGs should never be used for security purposes** (passwords, encryption keys, or authentication tokens).

For security-critical tasks, always use a **Cryptographically Secure PRNG (CSPRNG)**, such as Python's `secrets` module or Node.js's `crypto.randomBytes()`.

```

