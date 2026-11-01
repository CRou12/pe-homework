---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.19.5
kernelspec:
  name: python3
  display_name: Python 3 (ipykernel)
  language: python
---

```{code-cell} ipython3
import numpy as np
import matplotlib.pyplot as plt
from IPython.display import SVG, display
%matplotlib ipympl
```

```{code-cell} ipython3
def grid_init(h,w):
    re = np.linspace(-2, 0.8, h)
    im = np.linspace(-1.4, 1.4, w)
    grid = np.meshgrid(re,im)
    return grid

    
def mandelbrot(h,w):
    maxi = 20
    [X,Y] = grid_init(h,w)
    C = X+1j*Y
    Z = 0
    Z += C
    alive  = np.ones((h,w), dtype=bool) 
    diverge = np.full((h,w),maxi,dtype=int)
    for i in range(maxi):
        module = np.abs(Z)
        Z[alive] = Z[alive]**2+C[alive]
        diverged_now = alive & (module > 2) 
        diverge[diverged_now] = i
        alive[diverged_now] = 0
    return diverge
```

```{code-cell} ipython3
def grid_init_2(h,w,x_min,x_max,y_min,y_max):
    re = np.linspace(x_min, x_max, h)
    im = np.linspace(y_min, y_max, w)
    grid = np.meshgrid(re,im)
    return grid

def mandelbrot_v2(h,w,x_min=-2,x_max=0.8,y_min=-1.4,y_max=1.4,maxi=20):
    [X,Y] = grid_init_2(h,w,x_min,x_max,y_min,y_max)
    C = X+1j*Y
    Z = 0
    Z += C
    alive  = np.ones((h,w), dtype=bool) 
    diverge = np.full((h,w),maxi,dtype=int)
    for i in range(maxi):
        module = np.abs(Z)
        Z[alive] = Z[alive]**2+C[alive]
        diverged_now = alive & (module > 2) 
        diverge[diverged_now] = i
        alive[diverged_now] = 0
    plt.plot(diverge)
    plt.savefig('diverge.svg')
    return
```

```{code-cell} ipython3
plt.imshow(mandelbrot(1000,1000))
plt.show()
```

```{code-cell} ipython3

```
