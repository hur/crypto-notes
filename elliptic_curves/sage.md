# Elliptic Curves in Sage

An elliptic curve of form $ Y^2 = X^3 + AX + B $  over the finite field $\mathbb{F}_p$ can be defined as
```python
E = EllipticCurve(GF(p), [A, B])
```

Points and their operations are as follows:

```python
# define points
P = E(x, y)
Q = E(x2, y2)
# addition
P + Q
# subtraction
P - Q
# multiplication
5*P
```

Points in sage are represented in Standard Projective Coordinates where $(x, y, z)$ represents the affine point $(x / z, y / z)$ 

```python
E = EllipticCurve(GF(13), [14, 14])
P = E(0, 1)
# P : (0 : 1 : 1) --- z = 1 as expected
# A point not on the curve would have third coordinate zero e.g.
P - P # zero / infty
# (0 : 1 : 0)
```

Other useful operations:

```python
# Check if a point is on the curve
E.is_on_curve(x1, y1)
# Find corresponding y-coordinates given an x-coordinate
E.lift_x(x1)
```

