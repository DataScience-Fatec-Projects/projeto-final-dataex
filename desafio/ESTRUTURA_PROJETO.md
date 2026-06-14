# Projeto Final DataEx — Business Intelligence: Data-to-Insight Journey

> Transformar bancos de dados transacionais em decisões estratégicas através de
> arquitetura analítica.
> **Base escolhida:** FatecLog (logística — fato central: *Entrega*).
> **Entrega e apresentação:** 20/06/2026.

---

## 1. Visão Geral

O desafio é percorrer toda a jornada do dado — da origem transacional (OLTP) até o
dashboard final — convertendo tabelas normalizadas e complexas em um **modelo
dimensional (Star Schema)** legível por humanos e otimizado para ferramentas de BI.

O foco não é "como eu fiz", e sim **"o que esses dados dizem"** para o negócio.

| Etapa | Descrição | Saída |
|-------|-----------|-------|
| Seleção | Definição da base de dados fonte | Base FatecLog provisionada e populada |
| Modelagem | Desenho do Star Schema | Diagrama dimensional (Fato + Dimensões) |
| ETL | Carga e Transformação (M / SQL / Python) | Tabelas limpas e padronizadas |
| Dashboard | Criação de visuais e medidas DAX | Arquivo `.pbix` |
| Negócio | Apresentação de insights | Storytelling de negócio |

---

## 2. Objetivo e Escopo

- **Origem Transacional:** base real ou simulada que suporte operações de negócio.
  Aqui usamos a **FatecLog**, gerada com dados sintéticos (`faker` / SQL Server).
- **Valor Analítico:** converter as tabelas transacionais em um **Star Schema**
  que facilite a leitura humana e o consumo por ferramentas de BI (Power BI).

---

## 3. Critérios de Avaliação

| Critério | Peso | O que é avaliado |
|----------|:----:|------------------|
| **Modelagem** | 30% | Estruturação de Fatos e Dimensões. Qualidade das chaves e granularidade. |
| **ETL** | 20% | Processo de extração, limpeza e padronização dos dados brutos. |
| **Insights** | 50% | Capacidade de transformar dados em respostas para o negócio. |

> Os insights valem metade da nota — a entrega precisa **responder perguntas de
> negócio**, não apenas exibir gráficos.

---

## 4. Base Transacional (OLTP) — FatecLog

Modelo de origem normalizado, centrado na operação de entregas:

| Tabela | Papel | Campos principais |
|--------|-------|-------------------|
| `Cliente` | Quem contrata o frete | `nome_cliente`, `segmento`, `cidade`, `estado`, `data_cadastro` |
| `CentroDistribuicao` | De onde sai a carga | `nome_centro`, `cidade`, `estado`, `capacidade_operacional` |
| `Veiculo` | Como transporta | `placa`, `modelo`, `tipo_veiculo`, `capacidade_kg`, `ano_fabricacao`, `status_veiculo` |
| `Motorista` | Quem transporta | `nome_motorista`, `cidade`, `estado`, `data_admissao`, `categoria_cnh` |
| `Rota` | Origem → destino | `cidade_origem`/`estado_origem`, `cidade_destino`/`estado_destino`, `distancia_km` |
| `TipoCarga` | O que transporta | `descricao_tipo_carga`, `categoria` |
| `Entrega` | **Transação** (origem do fato) | FKs para todas acima + `data_saida`, `data_entrega`, `status_entrega`, `peso_carga_kg`, `valor_frete`, `custo_combustivel`, `tempo_estimado_horas`, `tempo_real_horas`, `quantidade_volumes`, `avaria`, `entrega_no_prazo` |

Volumetria de referência: ~200 clientes, 60 motoristas, 5 centros, 5 veículos,
6 rotas, 5 tipos de carga e **~15.000 entregas** (2 anos de histórico).

---

## 5. Modelagem Dimensional (Star Schema) — 30%

**Grão da fato:** *uma linha por entrega realizada*.

### 5.1 Tabela Fato

`FatoEntrega`
- **Chaves (Surrogate Keys):** `sk_cliente`, `sk_centro`, `sk_veiculo`,
  `sk_motorista`, `sk_rota`, `sk_tipo_carga`, `sk_data_saida`, `sk_data_entrega`
- **Métricas aditivas:** `valor_frete`, `custo_combustivel`,
  `margem` (= `valor_frete` − `custo_combustivel`), `peso_carga_kg`,
  `quantidade_volumes`
- **Métricas de desempenho:** `tempo_estimado_horas`, `tempo_real_horas`,
  `atraso_horas` (= `tempo_real` − `tempo_estimado`)
- **Flags / degeneradas:** `status_entrega`, `avaria` (0/1), `entrega_no_prazo` (0/1)

### 5.2 Dimensões (≥ 3 exigidas — aqui 7)

| Dimensão | Atributos descritivos |
|----------|-----------------------|
| `DimCliente` | nome, segmento, cidade, estado |
| `DimCentroDistribuicao` | nome do centro, cidade, estado, capacidade operacional |
| `DimVeiculo` | placa, modelo, tipo, capacidade (kg), ano, status |
| `DimMotorista` | nome, cidade, estado, categoria CNH, data de admissão |
| `DimRota` | cidade/estado origem, cidade/estado destino, distância (km) |
| `DimTipoCarga` | descrição, categoria |
| `DimCalendario` | data, ano, trimestre, mês, dia, dia da semana, fim de semana |

### 5.3 Regras de Modelagem (checklist)

- [ ] **Padrão Star Schema:** separação clara entre Fato e Dimensão.
- [ ] **Escopo mínimo:** ≥ 1 Fato e ≥ 3 Dimensões vinculadas. ✔ (7 dimensões)
- [ ] **Qualidade de dados:** tratamento de nulos, inconsistências e
      normalização de nomes (ex.: padronizar `cidade`/`estado`).
- [ ] **Performance:** chaves substitutas (*surrogate keys*) em todas as dimensões.
- [ ] **DimCalendario** dedicada para análises de sazonalidade.

---

## 6. ETL — 20%

Processo de **Extração → Limpeza/Padronização → Carga** (em M, SQL ou Python).

1. **Extração:** ler as tabelas OLTP da FatecLog.
2. **Limpeza:** tratar nulos, deduplicar, padronizar texto (caixa, acentos),
   normalizar siglas de estado.
3. **Transformação:** derivar `margem`, `atraso_horas`, flags; construir a
   `DimCalendario`.
4. **Surrogate Keys:** gerar chaves substitutas nas dimensões e mapear na fato.
5. **Carga:** popular dimensões primeiro, depois a `FatoEntrega`.

### Ecossistema Tecnológico (opções)

| Camada | Opções sugeridas |
|--------|------------------|
| Origem | SQL Server, MySQL, Postgres |
| Nuvem | Azure, AWS, Google Cloud, Fabric |
| ETL | Power Query, SQL Stored Procedure, Notebooks Python, Azure Data Factory, Azure Databricks |
| Análise | Power BI Desktop |

**Arquiteturas de referência:**
- **Power BI Only:** `DB → Power Query (ETL) → Power BI`
- **Data Warehouse:** `DB → ETL (SP/Python/Notebooks) → DW → Power BI`
- **Lakehouse:** `DB → ELT (ADF/Glue) → Lake → Bronze/Silver/Gold → Power BI`

---

## 7. Insights de Negócio — 50%

Não apenas gráficos, mas **respostas**. Perguntas-alvo para a FatecLog:

1. **Rentabilidade real:** quais rotas/segmentos/tipos de carga geram mais
   **margem** (frete − combustível), não apenas maior faturamento?
2. **Sazonalidade:** como o volume de entregas e os atrasos variam ao longo do
   ano (mês/trimestre/dia da semana)?
3. **Nível de serviço (SLA):** qual o % de entregas no prazo por centro, veículo
   e motorista? Onde estão os gargalos?
4. **Qualidade operacional:** qual a taxa de avaria por tipo de carga e veículo?
5. **Eficiência de custo:** custo de combustível por km / por kg transportado.

### Medidas DAX (exemplos)

- `Total Entregas = COUNTROWS(FatoEntrega)`
- `Faturamento Frete = SUM(FatoEntrega[valor_frete])`
- `Margem Total = SUM(FatoEntrega[valor_frete]) - SUM(FatoEntrega[custo_combustivel])`
- `% No Prazo = DIVIDE(SUM(FatoEntrega[entrega_no_prazo]), COUNTROWS(FatoEntrega))`
- `Atraso Médio (h) = AVERAGE(FatoEntrega[atraso_horas])`
- `Taxa de Avaria = DIVIDE(SUM(FatoEntrega[avaria]), COUNTROWS(FatoEntrega))`

---

## 8. Checklist de Entrega (20/06/2026)

- [ ] **Diagrama Dimensional** — desenho técnico do modelo de dados (PDF/IMG).
- [ ] **Arquivo PBIX** — dashboard completo com todas as medidas DAX.
- [ ] **Apresentação** — storytelling focado na visão de negócio.

---

## 9. Roadmap

Diagrama da jornada: [`jornada-roadmap.excalidraw`](./jornada-roadmap.excalidraw)

`Seleção → Modelagem (30%) → ETL (20%) → Dashboard → Negócio (50%)`

> *"Não me importa se o pato é macho, eu quero é o ovo."* — o que importa é o
> insight que o dado entrega ao negócio.
