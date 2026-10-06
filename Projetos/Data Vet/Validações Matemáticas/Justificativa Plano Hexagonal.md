Hexágono é a solução ótima para cobertura uniforme do plano com círculos iguais. Pentágonos e heptágonos não fecham o plano sem falhas ou sobreposição excessiva.

### 1) Restrição geométrica fundamental

Para preencher o plano sem buracos usando polígonos regulares, o ângulo interno deve dividir exatamente 360°.

Ângulo interno de um polígono regular:  
[  
\theta = \frac{(n - 2) \cdot 180^\circ}{n}  
]

Onde:

- ( n ) = número de lados
    
- ( \theta ) = ângulo interno
    

Testando:

- Triângulo (n=3): 60° → 360/60 = 6 → válido
    
- Quadrado (n=4): 90° → 360/90 = 4 → válido
    
- Hexágono (n=6): 120° → 360/120 = 3 → válido
    
- Pentágono (n=5): 108° → 360/108 ≈ 3.33 → inválido
    
- Heptágono (n=7): ~128.57° → 360/128.57 ≈ 2.8 → inválido
    

Conclusão: apenas triângulo, quadrado e hexágono pavimentam o plano perfeitamente.

---

### 2) Relação com círculos

Seu problema não é polígono. É **cobertura por círculos**.

Os centros dos círculos formam uma malha. Essa malha define a eficiência da cobertura.

- Malha triangular → equivalente a hexagonal (dual geométrico)
    
- Cada ponto tem 6 vizinhos equidistantes
    
- Isso maximiza cobertura e minimiza sobreposição
    

Distância ótima entre centros:  
[  
d = \sqrt{3} \cdot r  
]

Onde:

- ( d ) = distância entre centros
    
- ( r ) = raio do círculo
    

---

### 3) Eficiência de cobertura

Objetivo: cobrir área com mínimo de sobreposição e sem buracos.

Comparação:

|Malha|Cobertura|Overlap|Eficiência|
|---|---|---|---|
|Quadrada|tem gaps ou overlap alto|médio|pior|
|Triangular/Hex|sem gaps|mínimo|melhor|
|Pentagonal|impossível regular|—|inválido|
|Heptagonal|impossível regular|—|inválido|

Resultado conhecido na literatura:  
**hexagonal packing é a densidade máxima de empacotamento de círculos no plano**.

Densidade:  
[  
\eta = \frac{\pi}{\sqrt{12}} \approx 0.9069  
]

Onde:

- ( \eta ) = fração da área coberta por círculos
    

---

### 4) Intuição operacional (seu caso)

No seu projeto:

- Cada círculo = uma query na API (Google Places)
    
- Overlap = chamadas redundantes
    
- Gap = perda de dados
    

Hexagonal:

- Minimiza chamadas duplicadas
    
- Garante cobertura contínua
    
- Mantém distribuição uniforme
    

Pentágono/heptágono:

- Não formam grade regular
    
- Geram buracos ou exigem distorção
    
- Aumentam complexidade de cálculo
    

---

### 5) Conclusão

Você não escolheu hexágono por estética.

Escolheu porque:

- É uma das 3 únicas tesselações regulares possíveis
    
- É a **mais eficiente para círculos**
    
- Minimiza overlap mantendo cobertura total
    
- Possui solução analítica simples (√3·r)
    

Para cobertura geoespacial com raio fixo, não existe alternativa melhor.