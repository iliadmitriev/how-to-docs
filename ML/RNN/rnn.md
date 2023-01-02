# Recurrent Neural Networks

## RNN model and definitions

$$a^{<0>}=[0..0]$$

$$a^{<1>}=g_{1}(W_{aa}a^{<0>}+W_{ax}x^{1}+b_{a})$$

$$\hat y^{<1>}=g_{2}(W_{ya}a^{<1>}+b_{y})$$

$$a^{<t>}=g(W_{aa}a^{<t-1>}+W_{ax}x^{<t>}+b_a)$$

$$\hat y^{<t>}=g(W_{ya}a^{<t>}+b_y)$$

### Backpropagation through time

$$\mathcal{L}^{<t>}(\hat{y}^{<t>}, y^{<t>})=-y^{<t>}\log(\hat{y}^{<t>})-(1-y^{<t>})\log{1-\hat{y}^{<t>}}$$

$$\mathcal{L}(\hat{y},y)=\sum_{t=1}^{T_{y}}\mathcal{L}^{<t>}(\hat{y}^{<t>},y^{<t>})$$

## Gated Recurrent Unit

$$\tilde{c}^{<t>}=\tanh(W_{c}[\Gamma_{r}*c^{<t-1>},x^{<t>}]+b_c)$$

$$\Gamma_{u}=\sigma(W_{u}[c^{<t-1>},x^{<t>}] + b_u)$$

$$\Gamma_r=\sigma(W_r[c^{<t-1>},x^{<t>}]+b_c)$$

$$c^{<t>}=\Gamma_u*\tilde{c}^{<t>}+(1-\Gamma_u)*c^{<t-1>}$$
