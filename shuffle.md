---
marp: true
math: katex
---
# Random Shuffle

---
# Definition
Given a random number generator `random(l, u)` that generates integers in the range $[l, u]$ uniformly, and an array $a = (1, \ldots, n)$, a random shuffle uniformly shuffles the array into one of $n!$ possible permutations.

---
# Convention
Arrays in code will be 0-indexed, but tuples in math will be 1-indexed. This is somewhat confusing, but fits their context better.

---
# Fisher-Yates Shuffle Algorithm

```c++
void fisher_yates(int a[], int n) {
    for(int k = 0; k < n; k++) {
        int r = random(k, n - 1);
        swap(a[r], a[k]);
    }
}
```

---
# Standard Proof
Let $b = (b_1, \ldots, b_n)$ be an arbitrary permutation.

After the first swap, the probability of $b_1$ appearing at position $1$ is
$$
P(a_1 = b_1) = \frac{1}{n}
$$

After the second swap, the probability of $b_2$ appearing at position $2$ (as opposed to the other $n - 2$ elements) is
$$
P(a_2 = b_2 \;|\; a_1 = b_1) = \frac{1}{n - 1}
$$

More generally, after the $k$th swap, we have
$$
P(a_k = b_k \;|\; a_{k-1} = b_{k-1}, \ldots, a_1 = b_1) =
\frac{1}{n - k + 1}
$$

---
# Standard Proof
Therefore we have
$$
\begin{align*}
P(a = b) &=
P(a_1 = b_1) \\
&\times P(a_2 = b_2 \;|\; a_1 = b_1) \\
&\times P(a_3 = b_3 \;|\; a_2 = b_2, a_1 = b_1) \cdots \\
&= \frac{1}{n!}
\end{align*}
$$

This is an amazingly annoying proof.

---
# Alternative Proof
At each step $k$ there are $n - k + 1$ possible swaps, so there are $n!$ possible different executions of the algorithm.

Suppose there are two different sequences of swaps, $s$ and $t$. Let $k$ be the first step that they differ in, i.e.
$$
\begin{align*}
s_j &= t_j && \forall j < k \\
s_k &\neq t_k
\end{align*}
$$
Since the $k$th element will never be swapped again after the $k$th step, these two permutations must differ in the $k$th element. Thus the $n!$ possible results of the executions are unique.

Since these swaps are independently and uniformly generated, the entire sequence is also uniformly generated.

---
# Naive Shuffle
The alternative proof also proves why the naive algorithm is wrong
```c++
void naive(int a[], int n) {
    for(int k = 0; k < n; k++) {
        int r = random(0, n - 1);
        swap(a[r], a[k]);
    }
}
```
There are $n^n$ possible executions with equal probability. Since $n! \nmid n^n$, there is no way the results of the executions can be uniform.

---
# Swaps as Permutations
We can also represent swaps a permutation that permutes exactly the two elements
$$
\begin{align*}
\sigma_{ij}(a) &= \sigma_{ij}(\ldots,a_i,\ldots,a_j,\ldots) \\
&= (\ldots,a_j,\ldots,a_i,\ldots)
\end{align*}
$$

Furthermore, permutations may be composed
$$
(\sigma_{kl}\sigma_{ij})(a) = \sigma_{kl}(\sigma_{ij}(a))
$$

---
# Decomposing Fisher-Yates
In other words, the Fisher-Yates shuffle is the implementation of the product
$$
\sigma = \sigma_{ns_n}\cdots\sigma_{1s_1}
$$
where $s_k \in [k, n]$ is the sequence of random numbers.

As proven previously, such a permutation is uniformly sampled from all possible permutations.

This will become important later.

---
# Another Shuffling Algorithm
```c++
void back_shuffle(int a[], int n) {
    for(int k = 0; k < n; k++) {
        int r = random(0, k);
        swap(a[r], a[k]);
    }
}
```

---
# "Standard" Proof
An analogous standard proof to Fisher-Yates shuffle would be to prove an invariance and extend that invariance by induction.

We will see how that goes.

---
# Permutations as Bijections
Since permutations are bijections, we have that for any permutation $\sigma$
$$
a = b \Leftrightarrow \sigma(a) = \sigma(b) \\
P(a = b) = P(\sigma(a) = \sigma(b)) \\
$$

---
# Proof by Induction
Let $a_{1:k}$ denote the first $k$ elements $(a_1, \ldots, a_k)$. Let $\delta(a, j)$ denote the array without the $j$th element
$$
\delta(a, j) = (\ldots, a_{j-1}, a_{j+1}, \ldots)
$$
Let $b^k = (b^k_1,\ldots,b^k_k)$ denote an arbitrary permutation of length $k$.

Assume before step $k$, the first $k-1$ elements are uniformly permuted
$$
P(a_{1:k-1} = b^{k-1}) = \frac{1}{(k-1)!}
$$

This is trivially true for $k = 1$.

---
# Proof by Induction
Suppose $b^k_j = k$. Then after step $k$, we have
$$
\begin{align*}
P(a_{1:k} = b^k)
&= P(\sigma_{jk}(a_{1:k}) = \sigma_{jk}(b^k)) \\
&= P(a_j = k)P(\delta(\sigma_{jk}(a_{1:k}), k) = \delta(\sigma_{jk}(b^k), k) \;|\; a_j = k) \\
&= \frac{1}{k} P(\delta(\sigma_{jk}(a_{1:k}), k) = \delta(\sigma_{jk}(b^k), k) \;|\; a_j = k) && \tag{1}\\
&= \frac{1}{k} \cdot \frac{1}{(k-1)!} && \tag{2}\\
&= \frac{1}{k!}
\end{align*}
$$
where $(2)$ follows from $(1)$ by the induction hypothesis.

---
# That was Painful
Why was the proof so painful?

The source of all the difficulty is the computation of conditional probabilities. This requires proving invariance and the behaviour of each step.

Let's look at an alternate proof.

---
# Alternative Proof
At step $k$, there are $k$ equally possible swaps, so there are $n!$ equally possible executions.

But do different executions result in different permutations?

This is not so obviously true. At each step $k$, all previous elements may be swapped again with the $k$th element.

---
# Alternative Proof
Suppose there are two different sequences of swaps $s$ and $t$. Let $j$ be the first step they differ in. After step $j$, the two permutations differ on the position of $j$.

Let $i < j < k$ be any three indices. We observe that at step $k$ the $j$th element cannot get swapped with the $i$th element.

This means that the only way the resulting permutations can have $j$ be in the same position $k > j$ is if $s_k = s_j$ and $t_k = t_j$. However, if that were the case, we end up with the same problem with $k$ being in differing positions.

In other words, there is no way to "fix" a previous differing swap without creating a new difference. We therefore conclude the $n!$ possible executions result in unique permutations.

---
# Permutations Revisited
Recall that a swap can be represented by a permutation $\sigma_{ij}$. This permutation has an important property
$$
\sigma_{ij} = \sigma^{-1}_{ij}
$$

Recall also that Fisher-Yates can be written as
$$
\sigma = \sigma_{ns_n}\cdots\sigma_{1s_1}
$$
where $s_k \in [k, n]$. Similarly, `back_shuffle` can be written as
$$
\sigma = \sigma_{nt_n}\cdots\sigma_{1t_1}
$$
where $t_k \in [1, k]$.

---
# Permutations Revisited
Clearly, if a permutation is sampled uniformly from all possible permutations, so is its inverse permutation
$$
\begin{align*}
\sigma^{-1}
&= \sigma^{-1}_{1s_1}\cdots\sigma^{-1}_{ns_n} \\
&= \sigma_{1s_1}\cdots\sigma_{ns_n}
\end{align*}
$$

But this is exactly `back_shuffle` when performed on a reversed array!

---
# Why Should I Care?
Turns out, `back_shuffle` has the curious property that it doesn't need to look arbitrarily forward into future positions and only requires a single pass over future positions.

Sounds familiar?

That is the exact contract of a data stream. Or in C++ lingo, input iterators.

---
# Random Sample on Data Streams
A random sample of $d$ elements without replacement is exactly equivalent to taking the first $d$ elements of a permuted range.

Suppose we have a huge data stream, but a reasonable sample size $d$. We don't want to save the entire range when we only want the first $d$ elements of the permuted range.

This can be achieved by minor modifications to `back_shuffle`.

---
# Random Sample on Data Streams
```c++
using namespace std; // for brevity

template<input_iterator In, random_access_iterator Out,
         typename Distance, typename Rng>
auto random_sample(In first, In last, Out out, Distance d, Rng&& rng) {
    Distance k = 0;
    for(; first != last && k < d; ++first, ++k)
        out[k] = *first;
    shuffle(out, out + k, rng);
    for(; first != last; ++first, ++k) {
        auto s = uniform_int_distribution<Distance>(0, k)(rng);
        if(s < d)
            out[s] = *first
    }
    return out + min(k, d);
}
```
<sub>Taken from libc++ `std::sample` with modifications.</sub>

---
# Random Sample on Data Streams
In the first loop, we populate the first $d$ elements and shuffle them. Up to this point, this is the same as `back_shuffle` with copies.

In the second loop, we simply ignore every swap that doesn't modify the first $d$ elements.

If we don't care about the sample being randomly shuffled, we may even omit the `shuffle`.