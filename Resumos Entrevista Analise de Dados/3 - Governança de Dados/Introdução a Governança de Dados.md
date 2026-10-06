## 1. O que é Governança de Dados

**Governança de Dados** é o conjunto de processos, papéis, regras e controles usados para garantir que os dados de uma empresa sejam:

- confiáveis;
    
- seguros;
    
- bem documentados;
    
- fáceis de encontrar;
    
- usados corretamente;
    
- aderentes a leis e políticas internas.
    

No contexto de um **analista de dados**, governança significa saber **de onde o dado vem, quem é responsável por ele, o que ele representa, como pode ser usado e se ele é confiável para análise**.

Exemplo simples:

> Antes de criar um dashboard de vendas, é necessário saber qual tabela representa a venda oficial, quem mantém essa informação, quais campos são sensíveis e se os dados estão atualizados.

---

## 2. Por que Governança de Dados é importante

Sem governança, é comum acontecer:

|Problema|Exemplo|
|---|---|
|Métricas diferentes para o mesmo indicador|Financeiro calcula receita de um jeito, comercial de outro|
|Dados duplicados|Várias tabelas com clientes parecidos|
|Falta de confiança|Ninguém sabe se a tabela está correta|
|Risco legal|Dados pessoais expostos sem controle|
|Dificuldade de análise|Nomes de colunas confusos e sem documentação|
|Baixa rastreabilidade|Ninguém sabe de onde veio o dado do dashboard|

A governança tenta evitar exatamente esses problemas.

---

# 3. Principais conceitos

## 3.1 Data Owner

O **Data Owner** é o responsável de negócio por um conjunto de dados.

Ele normalmente não é a pessoa que programa ou mantém a tabela tecnicamente. Ele é quem entende o significado do dado dentro da área de negócio.

Exemplo:

|Dado|Possível Data Owner|
|---|---|
|Dados de clientes|Gerente de CRM|
|Dados financeiros|Coordenador financeiro|
|Dados de vendas|Gerente comercial|
|Dados de RH|Gerente de recursos humanos|

Responsabilidades comuns do **Data Owner**:

- definir quem pode acessar os dados;
    
- validar regras de negócio;
    
- aprovar o uso dos dados;
    
- garantir que o dado tenha um responsável claro;
    
- decidir classificações como confidencial, público ou sensível.
    

Exemplo prático:

> O Data Owner da área comercial define que “receita líquida” deve considerar descontos, devoluções e impostos.

---

## 3.2 Data Steward

O **Data Steward** é quem cuida da qualidade, documentação e padronização dos dados no dia a dia.

Ele atua mais próximo da operação dos dados, ajudando a garantir que os dados estejam corretos, bem descritos e utilizáveis.

Responsabilidades comuns do **Data Steward**:

- documentar tabelas e colunas;
    
- manter o glossário de dados;
    
- validar qualidade dos dados;
    
- identificar inconsistências;
    
- ajudar na padronização de nomes e regras;
    
- apoiar analistas e engenheiros no entendimento dos dados.
    

Exemplo prático:

> O Data Steward documenta que a coluna `customer_status` pode ter os valores `active`, `inactive` e `blocked`, explicando o significado de cada um.

---

## 3.3 Diferença entre Data Owner e Data Steward

|Papel|Foco principal|Exemplo|
|---|---|---|
|**Data Owner**|Responsabilidade de negócio|Define quem pode acessar dados de clientes|
|**Data Steward**|Qualidade e documentação|Documenta os campos da tabela de clientes|

Resumo direto:

> **Data Owner decide e responde pelo dado. Data Steward organiza, documenta e acompanha a qualidade do dado.**

---

# 4. PII Data Governance

## 4.1 O que é PII

**PII** significa **Personally Identifiable Information**, ou seja, informações que podem identificar uma pessoa.

Em português, está relacionado a **dados pessoais**.

Exemplos de PII:

|Tipo de dado|Exemplo|
|---|---|
|Nome|João Silva|
|CPF|123.456.789-00|
|E-mail|[joao@email.com](mailto:joao@email.com)|
|Telefone|+55 11 99999-9999|
|Endereço|Rua X, número Y|
|Data de nascimento|10/05/1990|
|Documento|RG, CNH, passaporte|

Também existem dados sensíveis, como:

|Tipo|Exemplo|
|---|---|
|Saúde|Histórico médico|
|Biometria|Impressão digital|
|Origem racial ou étnica|Informação declarada pelo usuário|
|Dados financeiros|Conta bancária, cartão|
|Dados de menores|Informações de crianças ou adolescentes|

---

## 4.2 O que é PII Data Governance

**PII Data Governance** é a governança aplicada especificamente a dados pessoais e sensíveis.

O objetivo é controlar:

- onde esses dados estão;
    
- quem pode acessar;
    
- para qual finalidade podem ser usados;
    
- por quanto tempo devem ser armazenados;
    
- como devem ser protegidos;
    
- como devem ser removidos ou anonimizados.
    

Exemplo:

> Um analista não deveria acessar CPF completo de clientes se precisa apenas calcular quantidade de clientes por região.

---

## 4.3 Boas práticas para dados PII

|Prática|Explicação|
|---|---|
|Classificação de dados|Identificar se o campo é público, interno, confidencial ou sensível|
|Controle de acesso|Permitir acesso apenas a quem realmente precisa|
|Mascaramento|Exibir CPF como `***.***.***-00`|
|Anonimização|Remover a possibilidade de identificar a pessoa|
|Criptografia|Proteger dados armazenados ou trafegando|
|Auditoria|Registrar quem acessou os dados|
|Retenção de dados|Definir por quanto tempo os dados podem ficar armazenados|

---

# 5. Data Lineage

## 5.1 O que é Data Lineage

**Data Lineage** é a rastreabilidade do dado.

Ele mostra:

- de onde o dado veio;
    
- por quais transformações passou;
    
- em quais tabelas foi salvo;
    
- quais dashboards ou relatórios usam esse dado.
    

Exemplo simples:

```text
Sistema de Vendas
        ↓
Tabela raw_sales
        ↓
Tratamento no Spark
        ↓
Tabela fact_sales
        ↓
Dashboard de Receita
```

Nesse caso, o lineage mostra o caminho do dado desde o sistema original até o dashboard final.

---

## 5.2 Por que Data Lineage é importante

|Uso|Exemplo|
|---|---|
|Entender origem dos dados|Saber se a receita vem do ERP ou do CRM|
|Analisar impacto de mudanças|Saber quais dashboards quebram se uma coluna mudar|
|Investigar erro|Descobrir onde uma métrica começou a ficar incorreta|
|Auditoria|Mostrar como determinado dado foi calculado|
|Compliance|Demonstrar controle sobre dados sensíveis|

Exemplo prático:

> Se a coluna `total_revenue` estiver errada em um dashboard, o lineage ajuda a descobrir se o problema veio da origem, da transformação ou da camada analítica.

---

# 6. Data Catalog

## 6.1 O que é Data Catalog

Um **Data Catalog** é uma ferramenta ou repositório usado para organizar e documentar os dados disponíveis na empresa.

Ele funciona como um “catálogo pesquisável” de tabelas, colunas, descrições, responsáveis e classificações.

Exemplo de informações em um catálogo:

|Informação|Exemplo|
|---|---|
|Nome da tabela|`gold.fact_sales`|
|Descrição|Tabela analítica de vendas|
|Dono|Área Comercial|
|Steward|Time de Dados|
|Classificação|Interno|
|Colunas|`sale_id`, `customer_id`, `amount`, `sale_date`|
|Atualização|Diária|
|Origem|ERP|
|Dashboards consumidores|Dashboard de Receita|

---

# 7. Data Quality

## 7.1 O que é Data Quality

**Data Quality** significa qualidade dos dados.

Um dado com boa qualidade deve ser:

|Dimensão|Explicação|Exemplo|
|---|---|---|
|Completude|Não faltam dados importantes|Cliente sempre tem `customer_id`|
|Consistência|Não há conflito entre sistemas|Status do cliente é igual no CRM e no DW|
|Validade|Segue regras esperadas|CPF tem formato válido|
|Unicidade|Não há duplicidade indevida|Um cliente não aparece 5 vezes|
|Atualidade|Está atualizado|Venda de hoje aparece no dashboard|
|Acurácia|Representa corretamente a realidade|Valor da venda está correto|

Exemplo:

> Uma tabela de clientes com CPF duplicado, e-mails inválidos e data de nascimento nula possui problemas de qualidade.

---

# 8. Data Classification

**Data Classification** é a classificação dos dados conforme sensibilidade e risco.

Exemplo:

|Classificação|Descrição|Exemplo|
|---|---|---|
|Público|Pode ser divulgado|Dados públicos do site|
|Interno|Uso interno da empresa|Relatórios operacionais|
|Confidencial|Acesso restrito|Receita, margem, contratos|
|Sensível|Alto risco legal ou pessoal|CPF, saúde, dados bancários|

Essa classificação ajuda a definir controles de acesso, criptografia, mascaramento e políticas de uso.

---

# 9. Access Control

**Access Control** é o controle de quem pode acessar quais dados.

Exemplo:

|Perfil|Acesso permitido|
|---|---|
|Analista de marketing|Dados agregados de campanhas|
|Analista financeiro|Receita, custos, margem|
|RH|Dados de funcionários|
|Engenheiro de dados|Acesso técnico controlado|
|Usuário externo|Normalmente sem acesso direto|

Boas práticas:

- aplicar menor privilégio possível;
    
- evitar acesso direto a dados sensíveis;
    
- separar acesso por área;
    
- usar grupos e papéis;
    
- auditar acessos.
    

---

# 10. Glossário de Dados

O **Glossário de Dados** define termos de negócio usados pela empresa.

Exemplo:

|Termo|Definição|
|---|---|
|Cliente ativo|Cliente que comprou nos últimos 90 dias|
|Receita bruta|Valor total antes de descontos e impostos|
|Receita líquida|Valor após descontos, devoluções e impostos|
|Churn|Cliente que deixou de consumir|
|Ticket médio|Receita dividida pelo número de pedidos|

O glossário evita que cada área use definições diferentes para o mesmo indicador.

---

# 11. Metadata

**Metadata** significa “dados sobre os dados”.

Exemplos de metadados:

|Tipo|Exemplo|
|---|---|
|Técnico|Nome da tabela, tipo da coluna, partição|
|Operacional|Última atualização, volume de linhas|
|Negócio|Definição do campo, responsável|
|Segurança|Classificação, política de acesso|
|Lineage|Origem e transformações|

Exemplo:

```text
Coluna: customer_id
Tipo: string
Descrição: Identificador único do cliente
Classificação: Interno
Origem: CRM
Owner: Área Comercial
```

---

# 12. Resumo geral

|Conceito|Definição resumida|
|---|---|
|Governança de Dados|Regras, papéis e processos para controlar o uso dos dados|
|Data Owner|Responsável de negócio pelo dado|
|Data Steward|Responsável por qualidade, documentação e padronização|
|PII|Dados que identificam uma pessoa|
|PII Governance|Controle específico sobre dados pessoais e sensíveis|
|Data Lineage|Caminho do dado da origem até o consumo|
|Data Catalog|Catálogo pesquisável de dados da empresa|
|Data Quality|Medida de confiabilidade dos dados|
|Data Classification|Classificação dos dados por sensibilidade|
|Access Control|Controle de acesso aos dados|
|Glossário de Dados|Definição padronizada dos termos de negócio|
|Metadata|Informações que descrevem os dados|

---

# 13. Visão prática para analista de dados

Para um analista de dados, governança aparece principalmente quando ele precisa responder perguntas como:

|Pergunta|Conceito relacionado|
|---|---|
|Posso usar essa tabela?|Access Control / Data Owner|
|Esse campo contém dado pessoal?|PII / Data Classification|
|Essa métrica está correta?|Data Quality|
|Quem define essa regra de negócio?|Data Owner|
|Onde encontro a descrição da tabela?|Data Catalog|
|De onde veio esse dado?|Data Lineage|
|O que significa essa coluna?|Metadata / Glossário|
|Posso mostrar esse dado no dashboard?|PII Governance / Access Control|

---

## Resumo final

**Governança de Dados** é o que garante que os dados sejam usados com confiança, segurança e responsabilidade.

Em termos simples:

> Governança define quem é responsável pelo dado, quem pode acessá-lo, como ele deve ser documentado, como sua qualidade é medida e como seu uso deve ser controlado.

Para um analista de dados, os pontos mais importantes são:

|Prioridade|Conceito|
|---|---|
|1|Saber se o dado é confiável|
|2|Entender a origem do dado|
|3|Saber quem é o responsável|
|4|Verificar se há dados pessoais ou sensíveis|
|5|Usar definições oficiais de métricas|
|6|Garantir que dashboards e análises estejam corretos|