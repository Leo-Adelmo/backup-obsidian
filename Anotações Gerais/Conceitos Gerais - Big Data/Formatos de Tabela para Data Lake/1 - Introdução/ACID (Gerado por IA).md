## ACID - Propriedades Fundamentais de Transações

ACID é um acrônimo que define **quatro propriedades fundamentais** que garantem a confiabilidade e consistência das transações em sistemas de banco de dados e, mais recentemente, em data lakes modernos.

### A - Atomicidade (Atomicity)

**O que é:** Uma transação é uma unidade indivisível - ou todas as operações são executadas com sucesso, ou nenhuma delas é aplicada.

**Na prática:**

- Se você está transferindo R$ 100 da conta A para conta B, **ambas** as operações (débito e crédito) devem ocorrer
- Se uma falhar, a outra é automaticamente desfeita (rollback)
- Não existe estado intermediário onde o dinheiro "desaparece"

**Exemplo em data lakes:**

```
Transação: Inserir 1000 registros em uma tabela
- Se 999 inserções funcionam mas 1 falha
- TODA a transação é desfeita
- A tabela permanece no estado original
```

### C - Consistência (Consistency)

**O que é:** O banco de dados deve sempre manter suas regras de integridade e restrições, passando de um estado válido para outro estado válido.

**Na prática:**

- Todas as constraints, chaves estrangeiras e regras de negócio são respeitadas
- Dados não podem violar as regras definidas no schema
- O sistema nunca fica em estado inválido

**Exemplo:**

```
Regra: Saldo de conta não pode ser negativo
- Tentativa de saque de R$ 1000 com saldo de R$ 500
- Transação é rejeitada para manter consistência
- Saldo permanece R$ 500 (estado válido)
```

### I - Isolamento (Isolation)

**O que é:** Transações concorrentes não devem interferir umas nas outras, como se cada uma fosse executada sequencialmente.

**Na prática:**

- Múltiplos usuários podem trabalhar simultaneamente
- Uma transação não vê mudanças de outras transações não finalizadas
- Evita problemas como "dirty reads" e "phantom reads"

**Exemplo:**

```
Usuário A: Lendo relatório de vendas
Usuário B: Inserindo nova venda
- A não vê a venda de B até que B confirme (commit)
- Garante que A tenha uma visão consistente dos dados
```

### D - Durabilidade (Durability)

**O que é:** Uma vez que uma transação é confirmada (commit), suas alterações são permanentes, mesmo em caso de falha do sistema.

**Na prática:**

- Dados são persistidos em armazenamento não-volátil
- Sobrevivem a quedas de energia, crashes do sistema
- Logs de transação garantem recuperação

**Exemplo:**

```
Após confirmar uma compra online:
- Mesmo se o servidor falhar imediatamente depois
- A compra permanece registrada
- Sistema pode recuperar o estado após reinicialização
```

## ACID em Diferentes Contextos

### Bancos de Dados Tradicionais

- **PostgreSQL, MySQL, Oracle:** Suporte nativo completo
- Implementação através de WAL (Write-Ahead Logging)
- Controle de concorrência com locks

### Data Lakes Modernos

- **Delta Lake, Iceberg, Hudi:** Trazem ACID para big data
- Desafio maior devido ao armazenamento distribuído
- Implementação através de metadados e versionamento

### NoSQL

- **MongoDB, Cassandra:** Suporte variável
- Frequentemente trade-offs entre ACID e performance/escalabilidade
- Alguns oferecem ACID apenas em operações específicas

## Por que ACID é Importante?

**Confiabilidade:** Garante que os dados estão sempre em estado válido

**Recuperação:** Sistema pode se recuperar de falhas mantendo integridade

**Concorrência:** Múltiplos usuários podem trabalhar simultaneamente com segurança

**Compliance:** Essencial para sistemas financeiros, médicos e outros setores regulamentados

## Limitações e Trade-offs

**Performance:** Operações ACID podem ser mais lentas devido às verificações

**Escalabilidade:** Mais difícil de implementar em sistemas distribuídos

**Complexidade:** Requer mais recursos computacionais e de armazenamento

**CAP Theorem:** Em sistemas distribuídos, há trade-offs entre Consistência, Disponibilidade e Tolerância a Partições

ACID representa um dos conceitos mais fundamentais em sistemas de dados, sendo a base para a confiabilidade que esperamos de qualquer sistema que gerencia informações críticas.