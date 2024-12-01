# Recurrent Neural Networks

## RNN model and definitions

$$a^{\langle 0\rangle }=[0 \dots 0]$$

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
## Attention

* `key` (ключи) - векторы, сравниваемые в входной последовательности $h_i$
* `value` (значение) - векторы, образующие контекст в входной последовательности $h_i$
* `query` (запрос) - векторы которые нужны в выходной последовательности $h'_{t-1}$

$$a(h_i, h'_{t-1}) = (W_k h_{i})^T(W_q h'_{t-1})/\sqrt{d}$$
$$\alpha_{ti}=SoftMax(a(h_{i}, h'_{t-1}))$$
$$c_t=\sum{\alpha_{ti}W_{v}h_{i}}$$
Размерности векторов
$$W_{q}^{d \times dim(h')}$$
$$W_{k}^{d \times dim(h)}$$
$$W_{v}^{d \times dim(h)}$$
### Формула внимания

$q$ - вектор-запрос, для которого хотим вычислить контекст
$K = (k_1, k_2, ... k_n)$ - векторы ключи сравниваемые с запросом
$V=(v_1, v_2, ..., v_n)$ - векторы значения образующие контекст (их усредняем чтобы сформировать вектор контекста)
$a(k_i, q)$ - функция релевантности, оценка релевантности сходства, ключа $k_i$ запросу $q$
$c$ - искомый вектор контекста, релевантный запросу

Модель внимания 3-х слойная сеть, вычисляющая выпуклую комбинацию значений $v_i$, релевантных запросу $q$.

$$c=Attn(q, K, V) = \sum_{i}{v_{i} SoftMax_{i}(a(k_{i}, q))}$$
$$c_{t} = Attn( W_{q} h'_{t-1}, W_{k} H, W_{v} H )$$
где,
$H=(h_1, h_2, ..., h_n)$ - входные векторы, 
$h'_{t-1}$ - выходной вектор

Внутреннее внимание (self-attention, самовнимание)

$$c_{i} = Attn( W_{q} h_{i}, W_{k} H, W_{v}H )$$
частный случай когда $h_{i} \in H$