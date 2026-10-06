## 1. Definição Formal

Dado um raio fixo $r > 0$, deseja-se escolher centros $\{c_i\} \subset \mathbb{R}^2$ tais que:

$$\mathbb{R}^2 = \bigcup_i B(c_i, r)$$

onde:

$$B(c, r) = \{ x \in \mathbb{R}^2 : \|x - c\| \leq r \}$$

Ou seja, todo ponto do plano deve estar a distância $\leq r$ de pelo menos um centro.

---

## 2. Métrica de Eficiência — Densidade de Cobertura

Define-se a **densidade de cobertura** como:

$$\delta = \frac{\text{área total dos discos por célula fundamental}}{\text{área da célula fundamental}}$$

Menor densidade implica menor sobreposição média e maior eficiência geométrica.

---

## 3. Resultado Ótimo no Plano (2D)

A densidade mínima possível é:

$$\delta^* = \frac{2\pi}{3\sqrt{3}} \approx 1{,}209$$

Essa densidade é atingida quando os centros formam uma **malha triangular (hexagonal)**, com distância entre vizinhos:

$$d = \sqrt{3}\, r$$

---

## 4. Justificativa Geométrica

Considere três centros formando um triângulo equilátero de lado $d$.

A região mais distante de qualquer centro dentro dessa célula é o **circuncentro** do triângulo.

No triângulo equilátero:

$$R_{\triangle} = \frac{d}{\sqrt{3}}$$

Para garantir cobertura:

$$\frac{d}{\sqrt{3}} \leq r \implies d \leq \sqrt{3}\, r$$

Para minimizar sobreposição, escolhe-se o maior valor permitido:

$$d = \sqrt{3}\, r$$

---

## 5. Diferença entre Cobertura e Empacotamento

| Problema | Objetivo | Distância ótima |
|---|---|---|
| Empacotamento | Não sobrepor, maximizar fração ocupada | $d = 2r$ |
| Cobertura | Cobrir tudo, minimizar redundância média | $d = \sqrt{3}\, r$ |

---

## 6. Aplicação a um Retângulo $[0, W] \times [0, H]$

### Variáveis

- $W$: largura
- $H$: altura
- $r$: raio
- $d = \sqrt{3}\, r$
- $d_y = \dfrac{3r}{2}$
- $\text{offset} = \dfrac{d}{2} = \dfrac{\sqrt{3}}{2}\, r$
- $\text{margin} = r$

### Coordenadas dos Centros

**Linha** $k$:

$$y_k = y_0 + k \cdot \frac{3r}{2}$$

**Coluna** $j$:

$$x_{j,k} = x_0 + j \cdot \sqrt{3}\, r + (k \bmod 2) \cdot \frac{\sqrt{3}}{2}\, r$$

Gerar centros na região expandida:

$$[-r,\; W+r] \times [-r,\; H+r]$$

---

## 7. Estimativa do Número de Círculos

$$N \approx \frac{2}{3\sqrt{3}} \cdot \frac{WH}{r^2}$$

Devido às bordas, o valor real será ligeiramente maior.

---

## 8. Pré-Requisitos Matemáticos

### 8.1 Cálculo
- Funções reais
- Integrais
- Noções de cálculo vetorial

### 8.2 Álgebra Linear
- Vetores em $\mathbb{R}^2$
- Determinantes (área da célula fundamental)
- Redes (*lattices*)

### 8.3 Geometria Euclidiana
- Triângulo equilátero
- Circunferência circunscrita
- Distância euclidiana

### 8.4 Geometria Discreta
- Redes no plano
- Densidade de cobertura
- Argumentos variacionais

---

## 9. Referências Clássicas

1. L. Fejes Tóth — *Lagerungen in der Ebene, auf der Kugel und im Raum* (1953)
2. C. A. Rogers — *Covering and Packing in Euclidean Space* (1964)
3. J. H. Conway, N. J. A. Sloane — *Sphere Packings, Lattices and Groups*
4. K. Böröczky Jr. — *Finite Packing and Covering* (2004)