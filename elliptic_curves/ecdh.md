# Elliptic Curve Diffie-Hellman 

## Key Exchange

### Public part

Alice and Bob agree on an elliptic curve $E(\mathbb{F}_p)$ and a particular point $G \in E(\mathbb{F}_p)$ called the generator point.

Alice chooses secret integer $n_A$ and computes $Q_A = n_A G$. 

Bob chooses secret integer $n_B$ and computes $Q_B = n_B G$.

$Q_A$ and $Q_B$ are exchanged between the parties - due to the hardness of ECDLP, onlooker Eve cannot calculate $n_A$ or $n_b$ in reasonable time.

### Private part

Alice calculates $n_AQ_B$ and Bob calculates $n_BQ_A$. Due to the associativity of scalar multiplication, $S = n_AQ_B = n_BQ_A$. $S$ is the shared secret. In fact,

$$n_AQ_B = (n_An_B)G = n_BQ_A$$

The y-value of $S$ can be discarded - only the $x$-coordinate matters (see: [ECC4: Efficient Exchange](https://cryptohack.org/challenges/ecc/)) That is, using either y-value, the computation would result in $\pm n_{A/B}Q_{A/B}$. 

## Attacks

### Pohlig-Hellman

Pohlig-Hellman reduces the problem by computing the discrete logarithm in the prime order subgroups of $\langle G\rangle$. 

It works as follows:

1. Write $G$ as $G = p_1^{e_1}p_2^{e_2}...p_r^{e_r}$ (prime factorization)
2. Compute $n \equiv n_i \mod p_i^{e_i}$ for $i \in [1, r]$ where the $p_i$'s are a mutually coprime set of integers such that $0 \leq n_i < p_i$.
   1. Each $n_i$ can be expressed in base $p$ (where $p$ is the largest primal divisor of $|G|$, the order of $G$)
      1. $n_i = z_0 + z_1p +z_2p^2 + ... + z_{e-1}p^{e-1}$ where $z_i \in [0, p-1]$. 
   2. Let $G_0 = \frac{|G|}{p_i}$ and $Q_0 = \frac{|G|}{p_i}Q$. 
   3. Using the fact that $|G_0| = p$, we obtain $Q_0 = nG_0 = z_0G_0$
   4. Then $z_i$ is computer by solving ECDLP in $\langle G_i \rangle$, i.e. solving $Q_i = z_i G_0$.

#### Exploiting in Sage

```python
# You should be able to just do
n_B = G.discrete_log(Q_A) # operation='+'?..
# to solve Q_A = n_B * G for n. Sage can use pohlig-hellman where appropriate

# More manually though:
G.order() # check if small prime subgroups
factors = [f[0] ** f[1] for f in factor(G.order())] # convert the factorization object to list
dlogs = []
for factor in factors:
   	G_0 = G * ZZ(G.order() / factor) # int() should work too
    Q_0 = Q_A * ZZ(G.order() / factor)
    dlog = discrete_log(Q_0, G_0, operation='+') # why operation='+'?
    dlogs.append(dlog)
n_A = crt(dlogs, factors) # solve crt 
```

### Smart's Attack

if `E.order() == p` or equivalently `E.trace_of_frobenius() == 1` , then we can perform [Smart's Attack](https://wstein.org/edu/2010/414/projects/novotney.pdf):

#### Exploiting in Sage

```python
def SmartAttack(P,Q,p):
    """
    Q = nP in GF(p)
    Smart's attack that works against the canonical lift case by randomizing the lift
    """
    E = P.curve()
    Eqp = EllipticCurve(Qp(p, 2), [ ZZ(t) + randint(0,p)*p for t in E.a_invariants() ])

    P_Qps = Eqp.lift_x(ZZ(P.xy()[0]), all=True)
    for P_Qp in P_Qps:
        if GF(p)(P_Qp.xy()[1]) == P.xy()[1]:
            break

    Q_Qps = Eqp.lift_x(ZZ(Q.xy()[0]), all=True)
    for Q_Qp in Q_Qps:
        if GF(p)(Q_Qp.xy()[1]) == Q.xy()[1]:
            break

    p_times_P = p*P_Qp
    p_times_Q = p*Q_Qp

    x_P,y_P = p_times_P.xy()
    x_Q,y_Q = p_times_Q.xy()

    phi_P = -(x_P/y_P)
    phi_Q = -(x_Q/y_Q)
    k = phi_Q/phi_P
    return ZZ(k)
```

