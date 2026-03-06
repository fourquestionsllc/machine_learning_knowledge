A **p-value** is a concept from Statistics used in **hypothesis testing** to measure how strong the evidence is against a **null hypothesis**.

---

# 1. Simple Intuition

The **p-value** answers this question:

> **If the null hypothesis were true, how likely is it to observe results at least as extreme as the data we got?**

* **Small p-value** → the observed result is unlikely under the null hypothesis
* **Large p-value** → the result is consistent with the null hypothesis

---

# 2. Example (Coin Flip)

Suppose the **null hypothesis** is:

```text
The coin is fair.
```

You flip the coin **10 times** and get **9 heads**.

Now ask:

> If the coin is fair, what is the probability of getting 9 or more heads?

That probability is the **p-value**.

If the p-value is small (for example **0.01**), then:

* it is very unlikely to happen with a fair coin
* we may reject the null hypothesis

---

# 3. Typical Threshold (Significance Level)

Researchers usually choose a **significance level**:

```text
α = 0.05
```

Decision rule:

| p-value  | conclusion                     |
| -------- | ------------------------------ |
| p ≤ 0.05 | reject null hypothesis         |
| p > 0.05 | fail to reject null hypothesis |

Important:
**Failing to reject does NOT mean the hypothesis is true.**

---

# 4. Visual Intuition

Imagine a distribution of possible outcomes if the null hypothesis is true.

```text
|---------|---------|---------|
 unlikely    common     unlikely
```

The **p-value measures how far the observed result lies in the tail**.

Extreme outcomes → small p-value.

---

# 5. Example in A/B Testing

In an **A/B experiment**:

```text
Version A conversion rate = 10%
Version B conversion rate = 12%
```

Hypothesis:

```text
H0: A and B have the same conversion rate
```

If the calculated **p-value = 0.02**

Interpretation:

* there is a **2% chance** of observing this difference if the two versions are actually the same
* we reject the null hypothesis

---

# 6. Common Misinterpretation

❌ Incorrect:

> p-value = probability the hypothesis is wrong

✅ Correct:

> p-value = probability of observing the data (or more extreme) assuming the null hypothesis is true.

---

# 7. Simple Analogy

Think of a **court trial**:

```text
Null hypothesis → defendant is innocent
```

Evidence produces a **p-value**.

* **small p-value** → evidence strongly contradicts innocence
* **large p-value** → evidence not strong enough

Rejecting the null hypothesis is like **convicting the defendant**.

---

# 8. Short One-Sentence Explanation

A **p-value** is:

> the probability of observing results as extreme as the data, assuming the null hypothesis is true.

---

✅ **Quick interview explanation**

You could say:

> A p-value measures how likely the observed data would occur if the null hypothesis were true. A small p-value indicates strong evidence against the null hypothesis, so we may reject it.
