# learn1_databricks

Guia rápido e organizado de comandos, estrutura e arquitetura Medallion no Databricks.

---

## Arquitetura Medallion (Medalhão)

A arquitetura Medallion organiza o fluxo de dados em camadas progressivas:

Staging → Bronze → Silver → Gold

markdown
Copiar código

### Staging
- Área temporária para receber arquivos brutos.
- Utiliza **Volumes** para armazenar CSV, JSON, Parquet, etc.
- Dados crus, sem tratamento.

### Bronze
- Primeira camada de tabelas Delta.
- Dados brutos organizados, versionados e auditáveis.
- Preserva o formato mais próximo do original.

### Silver
- Dados limpos, padronizados e enriquecidos.
- Conversões de tipos, remoção de duplicatas, joins, padronização.

### Gold
- Camada analítica final.
- Tabelas otimizadas para consumo em dashboards e modelos.

---

## Estrutura de Catálogo no Databricks

O Databricks organiza objetos na seguinte hierarquia:

Catalog → Schema → Tables / Views / Volumes

shell
Copiar código

### Estrutura recomendada:

```
minha_empresa # Catalog
├── 00_staging # Schema com Volumes
│ └── Volumes/
│ └── arqs/
│ └── landing/
├── 01_bronze # Tabelas Delta brutas
├── 02_silver # Tabelas limpas
└── 03_gold # Tabelas analíticas
```

yaml
Copiar código

### Significado de cada componente

| Item   | Descrição |
|--------|-----------|
| Catalog | Agrupa schemas e objetos relacionados ao projeto |
| Schema | Equivalente a um database |
| Volume | Espaço para armazenar arquivos (CSV, JSON, Parquet, etc.) |
| Table  | Tabelas Delta organizadas por camada |

---

## Comandos Úteis

### Criar catálogo
```sql
CREATE CATALOG minha_empresa;
```

Criar schemas
```sql
CREATE SCHEMA minha_empresa.00_staging;
CREATE SCHEMA minha_empresa.01_bronze;
CREATE SCHEMA minha_empresa.02_silver;
CREATE SCHEMA minha_empresa.03_gold;
```

Criar um Volume no Staging
```sql
CREATE VOLUME minha_empresa.00_staging.arqs;
```

Listar arquivos do Volume
```python
%python
dbutils.fs.ls('/Volumes/minha_empresa/00_staging/arqs')
```

Ingestão Bronze (Exemplo)
1. Ler arquivo do Staging
```
%python
raw_path = "/Volumes/minha_empresa/00_staging/arqs/dados.csv"

df = spark.read.csv(raw_path, header=True, inferSchema=True)
```


2. Criar tabela Bronze
```
python
df.write.format("delta") \
    .mode("overwrite") \
    .saveAsTable("minha_empresa.01_bronze.dados")
```

Transformação Silver
```
python
df_silver = (
    df.withColumn("salario", df.salario.cast("double"))
      .dropDuplicates()
)

df_silver.write.format("delta") \
    .mode("overwrite") \
    .saveAsTable("minha_empresa.02_silver.dados")
```

Gold (Agregações)
```
sql
CREATE OR REPLACE TABLE minha_empresa.03_gold.salario_medio AS
SELECT cidade, AVG(salario) AS salario_medio
FROM minha_empresa.02_silver.dados
GROUP BY cidade;
```

Objetivo do Repositório
Este repositório serve como:

Laboratório de aprendizado no Databricks

Guia de referência da arquitetura Medallion

Exemplos práticos de ingestão e transformação

Base para pipelines reais Bronze → Silver → Gold

Próximos Passos
Criar pipelines automatizados

Implementar testes de qualidade (expectations)

Criar Workflows

Conectar a camada Gold ao Power BI




