---
# You can also start simply with 'default'
theme: default
# some information about your slides (markdown enabled)
title: Float
info:
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawingin
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: fade-out
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
seoMeta:
  # By default, Slidev will use ./og-image.png if it exists,
  # or generate one from the first slide if not found.
  ogImage: auto
  # ogImage: https://cover.sli.dev
---

# Float

ICS recitation

---

# Contents

<toc />

---

# Fractional Binary Numbers

- representation
$$f = \sum_{k=-j}^{i} b_k \cdot 2^{k}$$
- limitation
  - Can't exactly represent all numbers
  - Limited range of numbers


---

# IEEE Floating Point

IEEE Standard 754

- representation

<div class="text-sm">

$$f = (-1)^s \cdot M \cdot 2^E$$

|   |    |   |   |
|-----|------|------|---|
|MSB| 1 | $s = s$ | 0 <span class="text-gray-500">10001100 11011011011010000000000</span> |
|exp field | 8 / 11 | $E = exp - bias (+1)$ | <span class="text-gray-500">0</span> 10001100 <span class="text-gray-500">11011011011010000000000</span> |
|frac field | 23 / 52 |  $M = ?.frac$ |<span class="text-gray-500">0 10001100</span> 11011011011010000000000 |
</div>

- three types of numbers
  - normalized: exp not all 0 or 1, implicit leading 1
  - denormalized: exp all 0, implicit leading 0
  - special values: exp all 1, frac all 0 is Inf, else is NaN

---
level: 2
---

# Dynamic Range

tiny floating point example (s=0)

<table class="text-xs">
  <thead>
    <tr>
      <th>Type</th>
      <th>Representation</th>
      <th>E</th>
      <th>Value</th>
    </tr>
  </thead>
  <tbody>
  <tr>
    <td rowspan="4">Denormalized</td>
    <td>0 0000 000</td>
    <td>-6</td>
    <td>+0.0</td>
  </tr>
  <tr>
    <td>0 0000 001</td>
    <td>-6</td>
    <td>1/512</td>
  </tr>
  <tr>
    <td>...</td>
    <td>...</td>
    <td>...</td>
  </tr>
  <tr>
    <td>0 0000 111</td>
    <td>-6</td>
    <td>7/512</td>
  </tr>
  <tr>
    <td rowspan="3">Normalized</td>
    <td>0 0001 000</td>
    <td>-6</td>
    <td>8/512</td>
  </tr>
  <tr>
    <td>...</td>
    <td>...</td>
    <td>...</td>
  </tr>
  <tr>
    <td>0 1110 111</td>
    <td>7</td>
    <td>240.0</td>
  </tr>
  <tr>
    <td rowspan="2">Special</td>
    <td>0 1111 000</td>
    <td>/</td>
    <td>+Inf</td>
  </tr>
  <tr>
    <td>0 1111 ###</td>
    <td>/</td>
    <td>NaN</td>
  </tr>
  </tbody>
</table>

---

# Rounding

- Rounding Modes
  - round to nearest even 四舍六入五成双
  - round toward +Inf
  - round toward -Inf
  - round toward 0
- Rounding Binary Numbers
  - “Even” when least significant bit is 0
  - “Half way” when bits to right of rounding position = $100..._2$


---

# Multiplication

- Exact Result
  - $s = s_1 \oplus s_2$
  - $M = M_1 \cdot M_2$
  - $E = E_1 + E_2$
- Fixing
  - If M ≥ 2, shift M right, increment E
  - If E out of range, overflow
  - Round M

---

# Addition

Assume $E_1 ≥ E_2$

- Exact Result
  - $(-1)^s \cdot M = (-1)^{s_1} \cdot M_1 + (-1)^{s_2} \cdot M_2 \cdot 2^{E_2-E_1}$
  - $E = E_1$
- Fixing
  - If M ≥ 2, shift M right, increment E
  - if M < 1, shift M left k positions, decrement E by k <span class="text-red">(until M ≥ 1 greedily)</span>
    - 0 0000 001 + 0 0000 001 = 0 0000 010
    - 0 0010 001 + 1 0010 001 = 0 0000 000
  - If E out of range, overflow
  - Round M

---

# Floating Point in C

- C Guarantees Two Levels
  - float: IEEE 754 single precision (32 bits)
  - double: IEEE 754 double precision (64 bits)
- Conversions/Casting
  - int2float: don't overflow, but may rounding
  - int/float2double: exact
  - double2float: may overflow to inf, may rounding
  - float/double2int: rounding to zero, may overflow to *integer indefinite*
    - (int) +1e10 = -21483648

---

# Thinking

Why IEEE 754 ?

- Why denormalized
  - Why 0.M
    - represent 0, and all bits 0
    - +0.0 and -0.0, good or bad? IEEE defined 1 / -0 = -Inf, 1 / +0 = +Inf
  - Why 1-bias
    - smooth transition
- Why special
  - represent overflow
  - represent not a number
- Why arrange in this way
  - compare without floating point operation, but negative numbers?
