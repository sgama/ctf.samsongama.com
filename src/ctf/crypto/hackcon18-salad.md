# Hackcon2018 Salad Upgrades

```
Sure, I could toss them all using just one shift. But am I gonna?

CIPHERTEXT: e4uo{zo1b_1e_f0j4l10i}z0ce
```

---

Attempt Caesar

```bash
$ for i in {1..26}; do echo "e4uo{zo1b_1e_f0j4l10i}z0ce" | caesar $i; done
f4vp{ap1c_1f_g0k4m10j}a0df
g4wq{bq1d_1g_h0l4n10k}b0eg
h4xr{cr1e_1h_i0m4o10l}c0fh
i4ys{ds1f_1i_j0n4p10m}d0gi
j4zt{et1g_1j_k0o4q10n}e0hj
k4au{fu1h_1k_l0p4r10o}f0ik
l4bv{gv1i_1l_m0q4s10p}g0jl
m4cw{hw1j_1m_n0r4t10q}h0km
n4dx{ix1k_1n_o0s4u10r}i0ln
o4ey{jy1l_1o_p0t4v10s}j0mo
p4fz{kz1m_1p_q0u4w10t}k0np
q4ga{la1n_1q_r0v4x10u}l0oq
r4hb{mb1o_1r_s0w4y10v}m0pr
s4ic{nc1p_1s_t0x4z10w}n0qs
t4jd{od1q_1t_u0y4a10x}o0rt
u4ke{pe1r_1u_v0z4b10y}p0su
v4lf{qf1s_1v_w0a4c10z}q0tv
w4mg{rg1t_1w_x0b4d10a}r0uw
x4nh{sh1u_1x_y0c4e10b}s0vx
y4oi{ti1v_1y_z0d4f10c}t0wy
z4pj{uj1w_1z_a0e4g10d}u0xz
a4qk{vk1x_1a_b0f4h10e}v0ya
b4rl{wl1y_1b_c0g4i10f}w0zb
c4sm{xm1z_1c_d0h4j10g}x0ac
d4tn{yn1a_1d_e0i4k10h}y0bd
e4uo{zo1b_1e_f0j4l10i}z0ce
```

None match the flag format. Try Vignere Cipher.
Hint being "not just one shift"

```python
import string
import collections


cipher = 'e4uo{zo1b_1e_f0j4l10i}z0ce'

# key 12345...

result = ''
i = 1


for char in cipher:
    lowercase = collections.deque(string.lowercase)
    if char not in string.digits and char != '{' and char != '}' and char != '_':
        cipher_index = string.lowercase.index(char)
        lowercase.rotate(i)
        result += lowercase[cipher_index]
    else:
        result += char
    i += 1

print(result)
```
