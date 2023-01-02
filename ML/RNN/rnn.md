# Recurrent Neural Networks

## RNN model and definitions

$$a^{\langle 0\rangle }=[0..0]$$

$$a^{\langle 1\rangle }=g_{1}(W_{aa}a^{\langle 0\rangle }+W_{ax}x^{\langle 1 \rangle}+b_{a})$$

$$\hat y^{\langle 1\rangle }=g_{2}(W_{ya}a^{\langle 1\rangle }+b_{y})$$

$$a^{\langle t\rangle }=g(W_{aa}a^{\langle t-1\rangle }+W_{ax}x^{\langle t\rangle }+b_a)$$

$$\hat y^{\langle t\rangle }=g(W_{ya}a^{\langle t\rangle }+b_y)$$

### Backpropagation through time

$$\mathcal{L}^{\langle t\rangle }(\hat{y}^{\langle t\rangle }, y^{\langle t\rangle })=-y^{\langle t\rangle }\log(\hat{y}^{\langle t\rangle })-(1-y^{\langle t\rangle })\log{1-\hat{y}^{\langle t\rangle }}$$

$$\mathcal{L}(\hat{y},y)=\sum_{t=1}^{T_{y}}\mathcal{L}^{\langle t\rangle }(\hat{y}^{\langle t\rangle },y^{\langle t\rangle })$$

## Gated Recurrent Unit

$$\tilde{c}^{\langle t\rangle }=\tanh(W_{c}[\Gamma_{r}*c^{\langle t-1\rangle },x^{\langle t\rangle }]+b_c)$$

$$\Gamma_{u}=\sigma(W_{u}[c^{\langle t-1\rangle },x^{\langle t\rangle }] + b_u)$$

$$\Gamma_r=\sigma(W_r[c^{\langle t-1\rangle },x^{\langle t\rangle }]+b_c)$$

$$c^{\langle t\rangle }=\Gamma_u*\tilde{c}^{\langle t\rangle }+(1-\Gamma_u)*c^{\langle t-1\rangle }$$
