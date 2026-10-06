# Apache Hudi - Conceitos Fundamentais

---

## 1. VISÃO GERAL DOS CONCEITOS

### Hierarquia dos Conceitos

```
HoodieRecord
├── HoodieKey
│   ├── recordKey (Identifica unicamente o registro)
│   └── partitionPath (Localização lógica do registro)
├── currentLocation (Onde está agora)
├── newLocation (Para onde vai)
└── payload (Dados reais)
```

### Por que Esses Conceitos Existem?

O Hudi precisa de uma forma **consistente e eficiente** para:

- ✅ **Identificar unicamente** cada registro na tabela
- ✅ **Localizar fisicamente** onde os registros estão armazenados
- ✅ **Organizar dados** em partições para performance
- ✅ **Fazer upserts eficientes** (encontrar e atualizar registros existentes)
- ✅ **Manter histórico** e versionamento de dados

---

## 2. HoodieKey - O DNA do Registro

### Definição

**HoodieKey** é a **identidade única** de cada registro no Hudi. É composto por duas partes obrigatórias:

```java
public class HoodieKey {
    private final String recordKey;     // Chave única do registro
    private final String partitionPath; // Partição onde está localizado
}
```

### Por que Dois Componentes?

#### recordKey (Chave do Registro)

- **Função**: Identifica **unicamente** o registro dentro de sua partição
- **Analogia**: Como o CPF de uma pessoa - único no sistema
- **Exemplo**: `user_123`, `order_456789`, `product_ABC001`

#### partitionPath (Caminho da Partição)

- **Função**: Define **onde** o registro deve ser armazenado fisicamente
- **Analogia**: Como o endereço/bairro onde a pessoa mora
- **Exemplo**: `year=2023/month=08/day=15`, `region=US/state=CA`

### Exemplo Prático - E-commerce

```sql
-- Registro de pedido
INSERT INTO orders_table VALUES (
    12345,           -- order_id
    'user_999',      -- customer_id  
    150.50,          -- amount
    '2023-08-15',    -- order_date
    'completed'      -- status
);
```

**HoodieKey resultante**:

- `recordKey`: `"12345"` (baseado em order_id)
- `partitionPath`: `"year=2023/month=08/day=15"` (baseado em order_date)

---

## 3. API KeyGenerator - O Extrator de Chaves

### Função Principal

O **KeyGenerator** é responsável por **extrair automaticamente** o HoodieKey dos dados de entrada, transformando registros "crus" em registros identificáveis pelo Hudi.

### Fluxo de Transformação

```
Raw Data → KeyGenerator → HoodieKey → HoodieRecord
```

### Implementações Disponíveis

#### 1. SimpleKeyGenerator (Mais Comum)

```java
// Configuração
hoodie.datasource.write.recordkey.field=order_id
hoodie.datasource.write.partitionpath.field=order_date

// Resultado
recordKey = "12345"
partitionPath = "2023-08-15"  // valor direto do campo
```

#### 2. ComplexKeyGenerator (Chaves Compostas)

```java
// Configuração para chave composta
hoodie.datasource.write.recordkey.field=customer_id,order_id
hoodie.datasource.write.partitionpath.field=region,order_date

// Resultado
recordKey = "customer_id:user_999,order_id:12345"
partitionPath = "region:US,order_date:2023-08-15"
```

#### 3. TimestampBasedKeyGenerator (Partições Temporais)

```java
// Configuração
hoodie.datasource.write.recordkey.field=transaction_id
hoodie.datasource.write.partitionpath.field=timestamp
hoodie.deltastreamer.keygen.timebased.timestamp.type=EPOCHMILLISECONDS
hoodie.deltastreamer.keygen.timebased.output.dateformat=yyyy/MM/dd

// Input: timestamp = 1692144000000 (epoch)
// Resultado
recordKey = "txn_789"
partitionPath = "2023/08/15"  // formatado automaticamente
```

#### 4. CustomKeyGenerator (Personalizado)

```java
public class MyCustomKeyGenerator extends KeyGenerator {
    @Override
    public HoodieKey getKey(GenericRecord record) {
        // Lógica personalizada
        String recordKey = extractCustomKey(record);
        String partitionPath = calculatePartition(record);
        return new HoodieKey(recordKey, partitionPath);
    }
}
```

### Configurações Importantes

#### Múltiplos Campos para recordKey

```properties
# Chave composta de múltiplos campos
hoodie.datasource.write.recordkey.field=user_id,product_id,timestamp

# Resultado: user_id:123,product_id:ABC,timestamp:1692144000
```

#### Hive Style Partitions

```properties
hoodie.datasource.write.hive_style_partitioning=true
hoodie.datasource.write.partitionpath.field=year,month,day

# Sem hive style: 2023/08/15
# Com hive style: year=2023/month=08/day=15
```

---

## 4. HoodieRecord - O Registro Completo

### Estrutura Completa

```java
public class HoodieRecord<T> {
    private HoodieKey key;           // Identidade (recordKey + partitionPath)
    private T data;                  // Payload (dados reais)
    private HoodieRecordLocation currentLocation;  // Onde está agora
    private HoodieRecordLocation newLocation;      // Para onde vai
    private HoodieOperation operation;             // INSERT/UPDATE/DELETE
}
```

### Campos Detalhados

#### HoodieKey (Já Explicado)

- **recordKey**: Identifica o registro
- **partitionPath**: Localização lógica

#### currentLocation

- **Função**: Indica onde o registro **atualmente existe** na tabela
- **Formato**: `<commit_timestamp>_<filegroup_id>`
- **Exemplo**: `"20230815123045_fg001"`
- **null**: Se for um registro completamente novo (INSERT)

#### newLocation

- **Função**: Indica onde o registro **será escrito**
- **Definido durante**: Processo de particionamento de registros
- **Uso**: Pelo write handle para saber em qual arquivo escrever

#### operation (HoodieOperation)

```java
public enum HoodieOperation {
    INSERT,    // Registro novo
    UPSERT,    // Insert ou Update (detectado automaticamente)
    DELETE     // Remoção do registro
}
```

### Exemplo Prático de Transformação

```java
// 1. Dados de entrada (JSON, Avro, etc.)
{
  "order_id": 12345,
  "customer_id": "user_999", 
  "amount": 150.50,
  "order_date": "2023-08-15",
  "status": "pending"
}

// 2. KeyGenerator extrai HoodieKey
HoodieKey key = new HoodieKey(
    recordKey: "12345",
    partitionPath: "year=2023/month=08/day=15"
);

// 3. HoodieRecord é criado
HoodieRecord record = new HoodieRecord(
    key: key,
    data: originalData,  // payload
    currentLocation: null,  // será populado pela indexação
    newLocation: null,      // será definido no particionamento
    operation: UPSERT
);
```

---

## 5. Payload - Os Dados Reais

### Definição

O **payload** é o conteúdo real dos dados do registro - todas as colunas e valores que você quer armazenar, excluindo os metadados do Hudi.

### Tipos de Payload

#### 1. OverwriteWithLatestAvroPayload (Padrão)

```java
// Estratégia: Registro mais recente sempre vence
// Baseado no preCombineField (geralmente timestamp)

// Registro existente: amount=100, timestamp=1000
// Novo registro: amount=150, timestamp=2000
// Resultado: amount=150, timestamp=2000 (mais recente vence)
```

#### 2. DefaultHoodieRecordPayload

```java
// Similar ao OverwriteWithLatestAvroPayload
// Mais otimizado para casos comuns
```

#### 3. PartialUpdateAvroPayload

```java
// Permite updates parciais - apenas campos alterados
// Útil para tabelas com muitas colunas onde só algumas mudam

// Registro existente: {name="João", age=30, city="SP", salary=5000}
// Update parcial: {age=31, salary=5500}  
// Resultado: {name="João", age=31, city="SP", salary=5500}
```

#### 4. Custom Payload

```java
public class MyCustomPayload implements HoodieRecordPayload {
    @Override
    public Option<IndexedRecord> combineAndGetUpdateValue(
            IndexedRecord currentValue, 
            Schema schema) {
        // Lógica personalizada de merge
        // Ex: somar valores, concatenar strings, etc.
        return Option.of(mergedRecord);
    }
}
```

### preCombineField - Campo de Ordenação

```sql
CREATE TABLE orders_hudi (
    order_id BIGINT,
    amount DECIMAL(10,2),
    last_updated TIMESTAMP
) USING HUDI
TBLPROPERTIES (
    primaryKey = 'order_id',
    preCombineField = 'last_updated'  -- Usado para resolver conflitos
);
```

**Como Funciona**:

- Quando múltiplas versões do mesmo registro existem
- O Hudi usa `preCombineField` para determinar qual é mais recente
- Registro com maior valor no `preCombineField` vence

---

## 6. FLUXO COMPLETO - EXEMPLO PRÁTICO

### Cenário: Sistema de Pedidos com Updates

#### Entrada de Dados

```json
[
  {
    "order_id": "12345",
    "customer_id": "user_999",
    "amount": 100.0,
    "status": "pending", 
    "order_date": "2023-08-15",
    "last_updated": "2023-08-15T10:00:00Z"
  },
  {
    "order_id": "12345",  // Mesmo pedido, status atualizado
    "customer_id": "user_999", 
    "amount": 100.0,
    "status": "completed",
    "order_date": "2023-08-15", 
    "last_updated": "2023-08-15T14:30:00Z"
  }
]
```

#### Configuração KeyGenerator

```properties
hoodie.datasource.write.recordkey.field=order_id
hoodie.datasource.write.partitionpath.field=order_date  
hoodie.datasource.write.precombine.field=last_updated
hoodie.datasource.write.payload.class=OverwriteWithLatestAvroPayload
```

#### Processamento Interno

**Registro 1**:

```java
HoodieKey key1 = new HoodieKey("12345", "2023-08-15");
HoodieRecord record1 = new HoodieRecord(
    key: key1,
    data: {order_id=12345, status="pending", last_updated="10:00:00Z", ...},
    currentLocation: null,  // novo registro
    operation: UPSERT
);
```

**Registro 2** (mesmo recordKey):

```java
HoodieKey key2 = new HoodieKey("12345", "2023-08-15");  // Mesma chave!
HoodieRecord record2 = new HoodieRecord(
    key: key2,
    data: {order_id=12345, status="completed", last_updated="14:30:00Z", ...},
    currentLocation: "20230815100500_fg001",  // localizado pela indexação
    operation: UPSERT
);
```

#### Merge de Payloads

```java
// OverwriteWithLatestAvroPayload compara last_updated
// "2023-08-15T14:30:00Z" > "2023-08-15T10:00:00Z"
// Registro 2 vence

Resultado final: {
    order_id: "12345",
    status: "completed",  // atualizado
    last_updated: "2023-08-15T14:30:00Z"
}
```

---

## 7. CONFIGURAÇÕES E BOAS PRÁTICAS

### Escolhendo recordKey

```properties
# ✅ BOM: Chave natural única
hoodie.datasource.write.recordkey.field=user_id

# ✅ BOM: Chave composta quando necessário  
hoodie.datasource.write.recordkey.field=user_id,product_id

# ❌ EVITAR: Campos que podem mudar
hoodie.datasource.write.recordkey.field=email  # email pode mudar!

# ❌ EVITAR: Campos com baixa cardinalidade
hoodie.datasource.write.recordkey.field=status  # poucos valores únicos
```

### Escolhendo partitionPath

```properties
# ✅ BOM: Partição temporal para dados time-series
hoodie.datasource.write.partitionpath.field=date

# ✅ BOM: Partição geográfica para dados regionais
hoodie.datasource.write.partitionpath.field=region

# ✅ BOM: Partição hierárquica
hoodie.datasource.write.partitionpath.field=year,month,day

# ❌ EVITAR: Campos com alta cardinalidade
hoodie.datasource.write.partitionpath.field=user_id  # muitas partições!
```

### Configurando preCombineField

```properties
# ✅ BOM: Timestamp de modificação
hoodie.datasource.write.precombine.field=updated_at

# ✅ BOM: Versão/sequencial
hoodie.datasource.write.precombine.field=version

# ❌ EVITAR: Campos que não indicam ordem temporal
hoodie.datasource.write.precombine.field=amount  # valor não indica recência
```

---

## 8. RESUMO E IMPORTÂNCIA

### Por que Esses Conceitos São Fundamentais?

1. **HoodieKey** = **Identidade única** que permite upserts eficientes
2. **KeyGenerator** = **Automação** da extração de chaves dos dados
3. **HoodieRecord** = **Container completo** com dados e metadados
4. **Payload** = **Dados reais** + **lógica de merge** personalizada

### Fluxo Resumido

```
Raw Data → KeyGenerator → HoodieKey → HoodieRecord → Index → Write
    ↓            ↓           ↓           ↓          ↓       ↓
JSON/CSV → recordKey + → Unique ID → Full Record → Location → Storage
          partitionPath                + payload    Tagged
```

### Benefícios

- ✅ **Upserts O(1)**: Localização direta via chaves
- ✅ **Particionamento automático**: Performance otimizada
- ✅ **Consistência garantida**: Merge determinístico
- ✅ **Flexibilidade**: KeyGenerators e Payloads customizáveis
- ✅ **Escalabilidade**: Funciona com bilhões de registros

Esses conceitos são a **base de tudo** no Hudi - entendê-los bem é fundamental para usar o sistema efetivamente!