# Guia do Projeto — Projeto Final DataEx

Guia de organização e colaboração para todos que vão trabalhar neste repositório.
Leia antes de pegar a primeira tarefa.

- **Organização:** [`DataScience-Fatec-Projects`](https://github.com/DataScience-Fatec-Projects)
- **Repositório:** [`projeto-final-dataex`](https://github.com/DataScience-Fatec-Projects/projeto-final-dataex)
- **Quadro (GitHub Project):** [Projeto Final DataEx #1](https://github.com/orgs/DataScience-Fatec-Projects/projects/1)
- **Base de dados definida:** **FatecLog** (logística — fato central: *Entrega*)
- **Entrega e apresentação:** **20/06/2026**

---

## 1. Sobre o projeto

Desafio de Business Intelligence: transformar um banco transacional (OLTP) em um
**modelo dimensional (Star Schema)** e entregar um dashboard em Power BI focado em
**insights de negócio**.

Detalhamento técnico completo (Star Schema, ETL, métricas, critérios de avaliação):
👉 [`desafio/ESTRUTURA_PROJETO.md`](./desafio/ESTRUTURA_PROJETO.md)
Roadmap visual: [`desafio/jornada-roadmap.excalidraw`](./desafio/jornada-roadmap.excalidraw)

**Pesos da avaliação:** Modelagem 30% · ETL 20% · **Insights 50%**.

---

## 2. Estrutura de pastas

```
projeto-final-dataex/
├── README.md                 # Apresentação do repositório
├── GUIA_DO_PROJETO.md        # Este guia (organização e colaboração)
└── desafio/
    ├── ESTRUTURA_PROJETO.md  # Estrutura técnica do projeto (FatecLog)
    ├── jornada-roadmap.excalidraw  # Roadmap visual da jornada
    └── Projeto Business Intelligence Desafio de Dados.pdf  # Brief oficial
```

Conforme o projeto evoluir, sugerimos organizar o código nestas pastas:

```
    ├── sql/        # Scripts OLTP, criação do modelo analítico e procedures de ETL
    ├── etl/        # Notebooks/scripts Python de extração, limpeza e carga
    ├── powerbi/    # Arquivo .pbix e medidas DAX
    └── docs/       # Diagrama dimensional (PDF/IMG) e apresentação
```

---

## 3. Organização no GitHub Project (quadro)

Todo o trabalho é coordenado pelo [Project #1](https://github.com/orgs/DataScience-Fatec-Projects/projects/1).
Cada tarefa é uma **issue** no repositório, vinculada ao quadro.

### Colunas (campo *Status*)

| Status | Significado |
|--------|-------------|
| **Backlog** | Tarefa registrada, ainda não priorizada. |
| **Ready** | Pronta para começar (dependências resolvidas). |
| **In progress** | Alguém está trabalhando nela (assine como *assignee*). |
| **In review** | PR aberto, aguardando revisão. |
| **Done** | Concluída e mergeada. |

### Labels de fase

As issues são classificadas pela etapa da jornada:

`fase: selecao` · `fase: modelagem` · `fase: etl` · `fase: dashboard` · `fase: insights` · `fase: entrega`

> A numeração das issues segue a jornada: Seleção → Modelagem → ETL → Dashboard → Insights → Entrega.

---

## 4. Fluxo de trabalho (Git)

### Branches

| Branch | Papel |
|--------|-------|
| `main` | Versão estável / entregável. **Protegida** — só recebe código via Pull Request revisado. |
| `develop` | Integração contínua do time. Base para as branches de trabalho. |
| `feat/...`, `etl/...`, `docs/...` | Branches de trabalho, uma por issue. |

### Convenção de nome de branch

```
<tipo>/<numero-da-issue>-<descricao-curta>
```

Tipos: `feat` (funcionalidade), `etl`, `sql`, `powerbi`, `docs`, `fix`, `chore`.

Exemplos:
- `etl/13-limpeza-padronizacao`
- `powerbi/17-medidas-dax-base`
- `docs/10-diagrama-dimensional`

### Passo a passo

1. Pegue uma issue no quadro e mova-a para **In progress**; atribua-se (*assignee*).
2. Crie a branch a partir de `develop`:
   ```bash
   git switch develop && git pull
   git switch -c etl/13-limpeza-padronizacao
   ```
3. Faça commits pequenos e descritivos.
4. Abra um **Pull Request** para `develop` e mova a issue para **In review**.
   Use `Closes #13` na descrição para fechar a issue ao mergear.
5. Após aprovação e merge, a issue vai para **Done**.
6. Periodicamente, abre-se um PR de release `develop → main`.

---

## 5. Convenções de commit e PR

Mensagens de commit no formato [Conventional Commits](https://www.conventionalcommits.org/):

```
<tipo>(<escopo opcional>): <descrição no imperativo>
```

Exemplos:
- `feat(etl): carrega FatoEntrega com surrogate keys`
- `docs: adiciona estrutura do projeto e roadmap`
- `fix(dax): corrige medida % No Prazo`

**Pull Requests** devem:
- Referenciar a issue (`Closes #N`);
- Descrever o que foi feito e como testar/validar;
- Ter pelo menos **1 revisão** antes do merge em `main`.

---

## 6. Padrões técnicos

- **Base:** FatecLog (não trocar — definida pelo time).
- **Modelo:** Star Schema com **1 tabela Fato** (`FatoEntrega`) e dimensões com prefixo `Dim`.
- **Chaves:** usar **surrogate keys** (`sk_*`) em todas as dimensões.
- **Qualidade:** tratar nulos, deduplicar e normalizar nomes/siglas de estado no ETL.
- **Nomenclatura:** tabelas e colunas em português, `snake_case`; fatos com prefixo `Fato`, dimensões com prefixo `Dim`.
- **Insights valem 50%:** toda análise deve responder uma **pergunta de negócio**, não apenas exibir um gráfico.

---

## 7. Definition of Done (DoD)

Uma tarefa só vai para **Done** quando:

- [ ] O código/artefato foi commitado na branch da issue;
- [ ] O PR foi revisado e aprovado;
- [ ] O resultado foi validado (dados conferem / medida correta / visual funciona);
- [ ] A issue foi fechada via `Closes #N`.

---

## 8. Links úteis

- [Estrutura técnica do projeto](./desafio/ESTRUTURA_PROJETO.md)
- [Roadmap (Excalidraw)](./desafio/jornada-roadmap.excalidraw)
- [Quadro do projeto (#1)](https://github.com/orgs/DataScience-Fatec-Projects/projects/1)
- [Brief oficial (PDF)](./desafio/Projeto%20Business%20Intelligence%20Desafio%20de%20Dados.pdf)
