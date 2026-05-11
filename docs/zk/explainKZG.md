
We start from:

$
e(\pi,; [Z(s)]_2) = e(C - [I(s)]_1,; H)
$

from the [KZG commitment](https://dankradfeist.de/ethereum/2020/06/16/kate-polynomial-commitments.html).

---

# 🧩 1. What is the **commitment**?

### ✔ Commitment

$
C = [f(s)]_1
$

* This is the commitment to the polynomial $f(x)$
* Lives in $G_1$

👉 Meaning:

> “I commit to a polynomial $f(x)$ without revealing it.”

> “Note: The `private inputs` is one part to derive the commitment, and the commitment is just a result, so the `private inputs` never reveals.”

---

# 📦 2. What is the **proof**?

### ✔ Proof

$
\pi = \left[\frac{f(s) - f(z)}{s - z}\right]_1
$

* Also a point in $G_1$

👉 Meaning:

> “Here is evidence that (f(x) - f(z)) is divisible by ((x - z)).”

---

# 🔑 3. What is the **verifying key $VK$**?

The verifier needs:

### ✔ Verifying key consists of:

* $[s]_2$ (or enough powers to compute $[Z(s)]_2)$
* $H \in G_2$ (generator)

From this, the verifier computes:

$
[Z(s)]_2 = [s - z]_2
$

👉 So VK = “structured reference string” in $G_2$

---

# 📍 4. What are the **public inputs**?

These are values the verifier knows:

### ✔ Public inputs:

* $z$: evaluation point
* $y = f(z)$: claimed value
* $C$: commitment

From these, the verifier builds:

$
[I(s)]_1 = [y]_1
$

---

# 🔐 5. What are the **secrets**?

There are two kinds of secrets:

---

## 🟣 (A) Prover’s secret

* The polynomial $f(x)$
* (or underlying witness in SNARK context)

👉 Not revealed

---

## 🔵 (B) Setup secret (toxic waste)

* $s$

👉 Critical:

* Nobody should know $s$
* But everyone has:

  * $[s]_1, [s]_2, [s^2]_1, \dots$

---

# ⚙️ 6. What each side of the equation represents

## Left side:

$
e(\pi,; [Z(s)]_2)
$

* Uses:

  * proof $\pi$
  * VK element $[Z(s)]_2$

👉 Encodes:

> “Quotient × divisor”

---

## Right side:

$
e(C - [I(s)]_1,; H)
$

* Uses:

  * commitment $C$
  * public value $y$

👉 Encodes:

> “$f(s) - y$”

---

# 🧠 7. What is actually being verified?

The equation checks:

$
f(s) - y = (s - z)\cdot q(s)
$

Which implies:

$
f(x) - y \text{ is divisible by } (x - z)
$

👉 Therefore:
$
f(z) = y
$

---

# 🎯 8. Clean mapping table

| Component     | Symbol         | Role                  | Known by             |
| ------------- | -------------- | --------------------- | -------------------- |
| Commitment    | $C = [f(s)]_1$ | binds polynomial      | public               |
| Proof         | $\pi$          | quotient witness      | prover → verifier    |
| Verifying key | $[s]_2, H$     | enables pairing check | public               |
| Public input  | $z, y$         | evaluation claim      | public               |
| Prover secret | $f(x)$         | hidden polynomial     | prover               |
| Setup secret  | $s$            | trapdoor              | nobody (after setup) |

---

# ⚡ 9. One-line intuition

* **Commitment $C$**: “I fixed a polynomial”
* **Public input $(z, y)$**: “I claim $f(z)=y$”
* **Proof $\pi$**: “Here’s the division witness”
* **VK**: “Tools to check at hidden point $s$”

And the pairing equation ensures:

> The claim is true **without revealing $f(x)$**
