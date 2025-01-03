# torch

## Gradient flow of a simple example

```python
import torch


X = torch.linspace(0, 2 * torch.pi, 10)
Y = X.sin()

W = torch.randn((10, 10), requires_grad=True)

for i in range(5):
    p = W @ X
    loss = (p - Y).pow(2).mean()
    loss.backward()
    W.data -= 0.01 * W.grad
    W.grad = None

```

## Gradient flow of a simple linear equation

```bash
from torch import FloatTensor
from torch.autograd import Variable


# Define the leaf nodes
a = Variable(FloatTensor([4]))

weights = [Variable(FloatTensor([i]), requires_grad=True) for i in (2, 5, 9, 7)]

# unpack the weights for nicer assignment
w1, w2, w3, w4 = weights

b = w1 * a
c = w2 * a
d = w3 * b + w4 * c
L = (10 - d)

L.register_hook(lambda grad: print(grad))
d.register_hook(lambda grad: print(grad))
b.register_hook(lambda grad: print(grad))
c.register_hook(lambda grad: print(grad))
b.register_hook(lambda grad: print(grad))

L.backward()


for index, weight in enumerate(weights, start=1):
    gradient, *_ = weight.grad.data
    print(f"Gradient of w{index} w.r.t to L: {gradient}")
```
