# Collective Sky Mathmatical Equation

Here lies the core of the project, using just a few circles we built the biggest spawn logo on 2b2t

<ins>**Go watch [SalC1's YouTube video](https://www.youtube.com/watch?v=ngsx4yCfqok) about the project if you havent already**</ins>

So, how did a few lines of maths let us accomplish this massive project, well all is explained below is perfect nerdy detail.

This writeup explains the core mathematical logic behind the `isBlockLogo` method, which was used to determine whether a specific block coordinate lies within the collective sky logo.

# Understanding `isBlockLogo`

---

## Parameters and Setup

- **Coordinates:** \((x, y)\) - the position to test.
- **Radius \(R\):** 8192 (defines the main circle size).
- **Margin \(m\):** 255 (defines thickness of the rings).
  
We calculate two squared radius bounds:

```java
final double radius = 8192;
final double margin = 255;
final double smaller = Math.pow(radius - margin, 2);
final double bigger = Math.pow(radius + margin, 2);
```

Where:

$$
r_{\text{small}}^2 = (R - m)^2, \quad r_{\text{big}}^2 = (R + m)^2
$$

---

## Main Concept

The function tests if a point lies **within one of several ring-shaped regions** (annuli). Each ring is defined by two concentric circles - an inner radius and an outer radius - creating a "thickness" for the ring.

For a point $(x, y)$, the distance squared from a center $(x_c, y_c)$ is:

$$
d^2 = (x - x_c)^2 + (y - y_c)^2
$$

A point lies within the ring if:

$$
r_{\text{small}}^2 < d^2 < r_{\text{big}}^2
$$

---

## Rings and Their Centers

### 1. Rings Centered at the Origin

Two main ring checks centered at $(0,0)$:

Outer ring near radius $2R$:

  $$
  (2R - m)^2 < x^2 + y^2 < (2R + m)^2
  $$

Inner ring near radius $R$:

  $$
  r_{\text{small}}^2 < x^2 + y^2 < r_{\text{big}}^2
  $$

Corresponding code snippet:

```java
final double toCenter = x * x + y * y;
final double radius = 8192;
final double margin = 255;
final double smaller = Math.pow(radius - margin, 2);
final double bigger = Math.pow(radius + margin, 2);

boolean mainCheck = (toCenter > Math.pow(2 * radius - margin, 2) && toCenter < Math.pow(2 * radius + margin, 2))
                 || (toCenter > smaller && toCenter < bigger);
```

---

### 2. Rings Centered at Offset Points

The function also checks six additional rings centered around six specific offset points:

* Compute the horizontal offset:

$$
O = \sqrt{R^2 - \left(\frac{R}{2}\right)^2} = \frac{\sqrt{3}}{2} R
$$

* Offset centers are:

$$
(0, \pm R), \quad (\pm O, \pm \frac{R}{2})
$$

For each offset center $(x_c, y_c)$, the point $(x,y)$ is tested with:

$$
r_{\text{small}}^2 < (x - x_c)^2 + (y - y_c)^2 < r_{\text{big}}^2
$$

Code snippet for distance checking:

```java
private static boolean checkDistance(double x, double z, double smaller, double bigger) {
    double toCenter = x * x + z * z;
    return toCenter > smaller && toCenter < bigger;
}
```

Example usage for an offset center at $(0, R)$:

```java
boolean isInOffsetRing = checkDistance(x, y - radius, smaller, bigger);
```

Full offset checks:

```java
final double magicOffset = Math.sqrt(radius * radius - Math.pow(radius / 2, 2));

boolean offsetCheck =
    checkDistance(x, y - radius, smaller, bigger) ||
    checkDistance(x, y + radius, smaller, bigger) ||
    checkDistance(x + magicOffset, y - radius / 2, smaller, bigger) ||
    checkDistance(x - magicOffset, y - radius / 2, smaller, bigger) ||
    checkDistance(x + magicOffset, y + radius / 2, smaller, bigger) ||
    checkDistance(x - magicOffset, y + radius / 2, smaller, bigger);
```

---

## Exclusion Zone

Any point **inside the inner radius** circle is automatically excluded:

$$
\sqrt{x^2 + y^2} < R - m \implies \text{return false}
$$

Code snippet:

```java
if(Math.sqrt(x * x + y * y) < (radius - margin)){
    return false;
}
```

---

## Full Condition Summary

Putting it all together, the `isBlockLogo` returns `true` if:

* The point lies within any of the defined rings (main rings or offset rings),
* **AND** it is not inside the exclusion zone (the inner circle).

Mathematically:

$$
\text{return true} \iff \left(
\begin{aligned}
& (2R - m)^2 < x^2 + y^2 < (2R + m)^2 \\
& \quad \text{OR} \quad r_{\text{small}}^2 < x^2 + y^2 < r_{\text{big}}^2 \\
& \quad \text{OR} \quad \exists (x_c, y_c) \in \text{offsetCenters}, \quad r_{\text{small}}^2 < (x - x_c)^2 + (y - y_c)^2 < r_{\text{big}}^2
\end{aligned}
\right)
\quad \text{and} \quad
\sqrt{x^2 + y^2} \geq R - m
$$

Final equation:

$$
\text{return true} \iff \left(
\begin{aligned}
& (2 \cdot 8192 - 255)^2 < x^2 + y^2 < (2 \cdot 8192 + 255)^2 \\
& \quad \text{OR} \quad (8192 - 255)^2 < x^2 + y^2 < (8192 + 255)^2 \\
& \quad \text{OR} \quad \exists (x_c, y_c) \in \text{offsetCenters}, \quad (8192 - 255)^2 < (x - x_c)^2 + (y - y_c)^2 < (8192 + 255)^2
\end{aligned}
\right)
\quad \text{and} \quad
\sqrt{x^2 + y^2} \geq 8192 - 255
$$

Where `offsetCenters` is:

$$
\{
\begin{aligned}
&(0, 8192) \\
&(0, -8192) \\
&\left(\frac{\sqrt{3}}{2} \cdot 8192,\ -\frac{8192}{2}\right) \\
&\left(-\frac{\sqrt{3}}{2} \cdot 8192,\ -\frac{8192}{2}\right) \\
&\left(\frac{\sqrt{3}}{2} \cdot 8192,\ +\frac{8192}{2}\right) \\
&\left(-\frac{\sqrt{3}}{2} \cdot 8192,\ +\frac{8192}{2}\right)
\end{aligned}
\}
$$

Numerically:

$$
\{
\begin{aligned}
&(0, 8192) \\
&(0, -8192) \\
&(7093.13,\ -4096) \\
&(-7093.13,\ -4096) \\
&(7093.13,\ +4096) \\
&(-7093.13,\ +4096)
\end{aligned}
\}
$$

Code snippet showing the full return logic:

```java
public static boolean isBlockLogo(int x1, int y1) {
    final double radius = 8192;
    final double margin = 255;
    final double x = x1 + 0.5, y = y1 + 0.5;

    final double smaller = Math.pow(radius - margin, 2);
    final double bigger = Math.pow(radius + margin, 2);

    final double toCenter = x * x + y * y;
    final double magicOffset = Math.sqrt(radius * radius - Math.pow(radius / 2, 2));

    if (Math.sqrt(toCenter) < radius - margin) {
        return false; // inside exclusion zone
    }

    boolean mainCheck = (toCenter > Math.pow(2 * radius - margin, 2) && toCenter < Math.pow(2 * radius + margin, 2))
            || (toCenter > smaller && toCenter < bigger)
            || checkDistance(x, y - radius, smaller, bigger)
            || checkDistance(x, y + radius, smaller, bigger)
            || checkDistance(x + magicOffset, y - radius / 2, smaller, bigger)
            || checkDistance(x - magicOffset, y - radius / 2, smaller, bigger)
            || checkDistance(x + magicOffset, y + radius / 2, smaller, bigger)
            || checkDistance(x - magicOffset, y + radius / 2, smaller, bigger);

    return mainCheck;
}
```

---
