# Legal AI Assistant

## Multi-Agent Legal Research, Evidence Verification and Document Drafting

**Legal AI Assistant** — проект интеллектуальной системы для автоматизации первичного юридического исследования и подготовки юридических документов.

Система объединяет несколько исследовательских контуров:

1. **External Legal Sources Agent** — поиск и получение информации из внешних официальных и разрешённых источников.
2. **Internal Legal Corpus Agent** — поиск по собственной проверенной юридической базе.
3. **Global Legal Practice Agent** — поиск зарубежной правовой практики на языке оригинала, когда это требуется юридическим запросом.
4. **Research Orchestrator & Legal Answer Agent** — объединение, фильтрация, проверка и интерпретация результатов всех исследовательских контуров.

Финальный результат — не просто список найденных документов и не свободный ответ LLM.

Система должна выдавать:

* релевантный юридический ответ;
* применимые нормы;
* судебную практику;
* условия и исключения;
* альтернативные позиции;
* точные цитаты из источников;
* кликабельные ссылки на первоисточники;
* информацию о редакции и дате документа;
* ограничения и недостающие данные;
* при необходимости — проект юридического документа.

---

# 1. Проблема

Первичное юридическое исследование включает большое количество рутинных операций:

```text
Юридический вопрос
        ↓
Поиск законодательства
        ↓
Поиск судебной практики
        ↓
Проверка редакции нормы
        ↓
Поиск дополнительных источников
        ↓
Чтение документов
        ↓
Сопоставление позиций
        ↓
Извлечение аргументов
        ↓
Проверка цитат
        ↓
Формирование правовой позиции
        ↓
Подготовка документа
```

При использовании обычной LLM появляется дополнительная проблема:

```text
LLM
 ↓
правдоподобный ответ
 ↓
вымышленная ссылка?
 ↓
несуществующий судебный акт?
 ↓
неправильная редакция нормы?
```

Для юридического применения такая ошибка критична.

В августе 2026 года Право.ru сообщало о деле, в котором Суд по интеллектуальным правам обнаружил в процессуальном документе 12 ссылок на несуществующие судебные акты либо выводы, которых не было в указанных документах; в результате был назначен судебный штраф.

Другой материал Право.ru от сентября 2026 года показывает, что суды отдельно обращают внимание на проверяемость источников и достоверность сведений, полученных с помощью ИИ.

Поэтому центральная задача проекта:

> **LLM не должна быть источником юридического факта. Источником является проверяемый документ, а LLM используется для поиска, сопоставления и формирования анализа на основе evidence.**

---

# 2. Цель проекта

Создать систему, которая автоматически выполняет большую часть первичного юридического исследования:

```text
Question
   ↓
Research Planning
   ↓
Multi-Source Retrieval
   ↓
Filtering
   ↓
Reranking
   ↓
Evidence Extraction
   ↓
Legal Reasoning
   ↓
Verification
   ↓
Structured Answer
   ↓
Document Drafting
```

Юрист при этом получает не набор поисковых результатов, а подготовленную доказательную основу для дальнейшей профессиональной оценки.

---

# 3. Главная концепция

Legal AI Assistant использует четыре специализированных агента.

```text
                         LAWYER
                           │
                           ▼
                ┌─────────────────────┐
                │       AGENT 4       │
                │  RESEARCH           │
                │  ORCHESTRATOR       │
                └──────────┬──────────┘
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
        ┌─────────┐   ┌─────────┐   ┌─────────┐
        │ AGENT 1 │   │ AGENT 2 │   │ AGENT 3 │
        │External │   │Internal │   │ Global  │
        │Sources  │   │ Corpus  │   │Practice │
        └────┬────┘   └────┬────┘   └────┬────┘
             │             │             │
             ▼             ▼             ▼
        Official       Internal       Foreign
        sources        knowledge      sources
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    EVIDENCE LAYER
                           │
                           ▼
                    VERIFICATION
                           │
                           ▼
                    FINAL ANSWER
                           │
                           ▼
                   DOCUMENT DRAFTING
```

---

# 4. Agent 1 — External Legal Sources Agent

## Назначение

Получение актуальной информации из внешних источников.

Приоритет:

1. официальные источники;
2. государственные правовые порталы;
3. официальные судебные источники;
4. официальные публикации государственных органов;
5. другие заранее разрешённые источники.

В качестве одного из базовых источников российского законодательства может использоваться Официальный интернет-портал правовой информации.

Портал содержит официальное опубликование правовых актов, тексты актов с изменениями и интегрированный банк «Законодательство России».

## Pipeline

```text
External Source
       ↓
Fetch
       ↓
Parse
       ↓
Normalize
       ↓
Validate
       ↓
Metadata Extraction
       ↓
Version Detection
       ↓
Content Hash
       ↓
Provenance
       ↓
Local Corpus
```

Документ сохраняется вместе с:

```text
document_id
title
document_type
number
adoption_date
publication_date
effective_from
effective_to
revision_date
source
source_url
retrieved_at
content_hash
parser_version
text
```

## Важный принцип

Внешний источник не становится автоматически доверенным только потому, что его нашла система.

Сначала:

```text
External Document
       ↓
Validation
       ↓
Metadata
       ↓
Version
       ↓
Provenance
       ↓
Evidence
```

---

# 5. Agent 2 — Internal Legal Corpus Agent

## Назначение

Работа с собственной накопленной и проверенной юридической базой.

Внутренний корпус может содержать:

```text
INTERNAL LEGAL CORPUS

├── legislation
├── court_decisions
├── legal_documents
├── previous_research
├── company_documents
└── private_corpus
```

Для MVP:

```text
SQLite
+
FTS5
+
FAISS
+
Embeddings
```

Для масштабирования:

```text
PostgreSQL
+
Qdrant / другой vector store
```

## Retrieval

Используются два независимых поиска.

### Lexical

```text
Query
 ↓
SQLite FTS5
 ↓
BM25
 ↓
Candidate Documents
```

### Semantic

```text
Query
 ↓
Embedding
 ↓
FAISS
 ↓
Semantic Candidates
```

После объединения:

```text
Lexical Candidates
        +
Semantic Candidates
        ↓
Candidate Pool
        ↓
CrossEncoder
        ↓
Reranking
        ↓
Top Evidence
```

---

# 6. Agent 3 — Global Legal Practice Agent

Этот агент запускается **не для каждого запроса**, а когда пользовательский запрос действительно требует международного сравнения.

Примеры:

> «Сравни российское регулирование с немецким».

> «Есть ли аналогичная практика в ЕС?»

> «Покажи подход английских судов».

> «Найди зарубежную судебную практику по этому вопросу».

## Особенность

Поиск выполняется на языке соответствующей правовой системы.

```text
Russian Query
      ↓
Legal Query Transformation
      ↓
English / German / other language
      ↓
Foreign Retrieval
      ↓
Original Source
      ↓
Exact Quote
      ↓
Translation
      ↓
Original Source Link
```

Система должна сохранять:

* оригинальный текст;
* перевод;
* язык;
* источник;
* ссылку;
* дату;
* юрисдикцию;
* реквизиты документа.

Перевод не заменяет оригинальную цитату.

---

# 7. Agent 4 — Research Orchestrator

Это центральный компонент системы.

Он не является просто LLM-чатом.

Он отвечает за:

1. понимание запроса;
2. определение юрисдикции;
3. определение даты;
4. определение требуемых источников;
5. построение research plan;
6. запуск нужных агентов;
7. объединение результатов;
8. фильтрацию;
9. reranking;
10. построение evidence map;
11. проверку утверждений;
12. формирование результата;
13. подготовку документа.

---

# 8. Query Understanding

Например, юрист пишет:

> «Может ли работодатель взыскать с работника ущерб, причинённый в мае 2024 года? Найди российское законодательство и судебную практику. Дополнительно сравни с Германией».

Agent 4 преобразует запрос в структурированную задачу:

```text
jurisdiction = Russia

legal_area = labor_law

event_date = 2024-05

needs_legislation = true

needs_case_law = true

needs_foreign_practice = true

foreign_jurisdiction = Germany

task_type = legal_research
```

---

# 9. Dynamic Agent Routing

Не каждый запрос требует всех трёх исследовательских агентов.

Например:

```text
"Что означает статья 238 ТК РФ?"
```

может использовать:

```text
Agent 2
   ↓
Internal Corpus
   ↓
Agent 4
```

Если локальной информации недостаточно:

```text
Agent 2
   ↓
insufficient evidence
   ↓
Agent 1
```

Если юрист попросил немецкую практику:

```text
Agent 3
```

Таким образом:

```text
Simple Query
    ↓
minimum required agents

Complex Query
    ↓
multiple research agents
```

Это снижает latency и вычислительные затраты.

---

# 10. Multi-Source Retrieval

Полный pipeline:

```text
                         USER QUERY
                              │
                              ▼
                     QUERY UNDERSTANDING
                              │
                              ▼
                       RESEARCH PLAN
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
          External         Internal         Global
           Agent             Agent           Agent
              │               │               │
              ▼               ▼               ▼
        Official sources    Local DB      Foreign sources
              │               │               │
              └───────────────┼───────────────┘
                              ▼
                       Candidate Pool
                              │
                              ▼
                     Hybrid Retrieval
                              │
                              ▼
                      CrossEncoder
                       Reranking
                              │
                              ▼
                      Evidence Builder
                              │
                              ▼
                       Legal Reasoning
                              │
                              ▼
                     Answer Verification
```

---

# 11. Version-Aware Retrieval

Юридический поиск должен учитывать не только документ, но и его редакцию.

Например:

```text
event_date = 2024-05-15
```

Имеются:

```text
Version A
effective_from = 2023-01-01
effective_to   = 2024-07-01

Version B
effective_from = 2024-07-01
```

Для события 15.05.2024 выбирается Version A.

Формально:

```text
effective_from <= event_date
AND
event_date < effective_to
```

Это позволит реализовать:

* исторический retrieval;
* хранение редакций;
* отслеживание изменений;
* переиндексацию изменившихся документов;
* проверку временной применимости нормы.

---

# 12. Provenance

Каждый документ должен иметь происхождение.

```text
source_type
source_name
source_url
retrieved_at
publication_date
revision_date
document_id
content_hash
parser_version
```

Результат можно проследить:

```text
Answer
  ↓
Claim
  ↓
Evidence
  ↓
Document
  ↓
Document Version
  ↓
Source
  ↓
Retrieved At
```

Это делает результат воспроизводимым.

---

# 13. Evidence Builder

Evidence Builder не должен генерировать цитаты.

Он должен **извлекать точный фрагмент из сохранённого документа**.

```text
document_id
document_title
article
revision_date
source_url
exact_quote
source_text
relevance_score
retrieval_metadata
```

Пример:

```text
CLAIM

Работодатель вправе ...

EVIDENCE

"точный фрагмент документа..."

SOURCE

ТК РФ, статья ...

VERSION

Редакция от ...

SOURCE URL

[Открыть первоисточник]
```

---

# 14. Claim → Evidence Graph

Каждое существенное утверждение связывается с evidence.

```text
CLAIM 1
 ├── Evidence A
 └── Evidence B

CLAIM 2
 └── Evidence C

CLAIM 3
 └── NO SUFFICIENT EVIDENCE
```

Если evidence недостаточно:

```text
Claim
  ↓
Evidence not found
  ↓
Additional retrieval
  ↓
Evidence found?
 ├── YES → verify → include
 └── NO  → uncertainty / exclude
```

Это принципиально важно.

Система не должна превращать отсутствие evidence в уверенное юридическое утверждение.

---

# 15. Answer Verification

После генерации ответа запускается отдельная проверка.

## Проверяем:

### Citation verification

Существует ли указанная цитата в исходном документе?

### Metadata verification

Совпадают ли:

* название;
* номер;
* статья;
* дата;
* суд;
* номер дела?

### URL verification

Ведёт ли ссылка на соответствующий источник?

### Evidence support

Действительно ли приведённый фрагмент подтверждает утверждение?

### Version verification

Используется ли редакция, действовавшая на нужную дату?

### Coverage

Есть ли существенные утверждения без evidence?

---

# 16. Важное ограничение

Техническая проверка цитаты не доказывает юридическую правильность интерпретации.

Например:

```text
Citation exists
       ↓
YES

Metadata correct
       ↓
YES

Quote exists
       ↓
YES
```

Это ещё не означает:

```text
Legal interpretation = definitely correct
```

Поэтому система должна разделять:

```text
Source Verification
```

и

```text
Legal Interpretation
```

---

# 17. Формат итогового ответа

Юрист получает:

```text
1. Краткий вывод

2. Применимые нормы

3. Условия применения

4. Исключения

5. Судебная практика

6. Основная правовая позиция

7. Альтернативные позиции

8. Факторы, которые могут изменить вывод

9. Доказательная база

10. Источники

11. Ограничения анализа
```

Каждое существенное утверждение должно быть связано с evidence.

---

# 18. Подготовка юридических документов

Legal AI Assistant должен уметь использовать результат исследования для создания документов.

```text
Research
   ↓
Applicable Law
   ↓
Evidence
   ↓
Legal Position
   ↓
Document Structure
   ↓
Draft
   ↓
Verification
   ↓
Final Document
```

Потенциальные типы документов:

* договор;
* дополнительное соглашение;
* претензия;
* ответ на претензию;
* исковое заявление;
* отзыв;
* ходатайство;
* жалоба;
* правовое заключение;
* юридическая записка;
* письмо;
* сравнительный анализ редакций документа.

---

# 19. Безопасная генерация документов

Документ не должен строиться исключительно на «памяти» LLM.

Pipeline:

```text
User Request
      ↓
Research
      ↓
Verified Evidence
      ↓
Legal Position
      ↓
Document Draft
      ↓
Citation Verification
      ↓
Document
```

Если в документе обнаруживается неподтверждённое юридическое утверждение:

```text
Unsupported Claim
       ↓
Re-retrieval
       ↓
Evidence?
   ├── YES → update
   └── NO  → remove / flag
```

---

# 20. Технологический стек MVP

Проект можно начать без платных API.

## Backend

```text
Python
FastAPI
Pydantic
```

## Retrieval

```text
SQLite
SQLite FTS5
FAISS
Sentence Transformers
E5 embeddings
CrossEncoder
```

## LLM

Локальная модель через:

```text
Ollama
```

## UI

```text
Gradio
```

## Evaluation

```text
pytest
scikit-learn
custom evaluation scripts
```

## Documents

```text
PyMuPDF
python-docx
BeautifulSoup
OCR — при необходимости
```

## Deployment

```text
Docker
GitHub
```

---

# 21. Бесплатный MVP

Первую версию можно реализовать локально.

```text
Local Computer

├── Python
├── SQLite
├── FTS5
├── FAISS
├── E5
├── CrossEncoder
├── Ollama
└── Gradio
```

Не требуется сразу:

* облачный GPU;
* Kubernetes;
* микросервисная архитектура;
* большой vector database;
* платная LLM;
* коммерческая юридическая база.

---

# 22. MVP Corpus

Не нужно начинать со всего российского законодательства.

Для доказательства архитектуры достаточно ограниченного проверенного корпуса.

Например:

```text
MVP

100–500 нормативных документов
+
ограниченный корпус судебных актов
+
10–30 иностранных документов
+
30–50 экспертно подготовленных тестовых запросов
```

После проверки pipeline корпус постепенно расширяется.

---

# 23. Golden Dataset

Для оценки качества создаётся набор юридических вопросов.

Каждый вопрос содержит:

```text
query
jurisdiction
date
expected_documents
expected_chunks
expected_evidence
expected_source
```

Например:

```text
Query:
...

Jurisdiction:
Russia

Event date:
2024-05-15

Relevant documents:
...

Relevant evidence:
...

Expected source:
...
```

---

# 24. Метрики

## Retrieval

```text
Precision@K
Recall@K
MRR
NDCG@K
```

## Reranking

```text
Recall@K
MRR
NDCG@K
```

## Evidence

```text
Citation Accuracy
Citation Completeness
Source Correctness
URL Correctness
Evidence Support
```

## Generation

```text
Groundedness
Factual Consistency
Legal Basis Coverage
Conditions Coverage
Exception Coverage
Unsupported Claim Rate
```

## Data Pipeline

```text
Sync Success Rate
Documents Added
Documents Changed
Version Correctness
Parsing Errors
```

## Performance

```text
Latency
RAM
Index Size
Inference Time
```

---

# 25. Source Freshness

Система должна знать, насколько свеж её корпус.

```text
last_successful_sync
last_document_update
documents_added
documents_changed
documents_removed
sync_status
```

Если внешний источник временно недоступен:

```text
External Source
      ↓
ERROR
      ↓
Keep previous verified corpus
      ↓
Record failure
      ↓
Retry later
```

Предыдущая проверенная версия не должна автоматически удаляться.

---

# 26. Архитектура данных

```text
                    SOURCES
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       External      Internal      Global
          │            │            │
          └────────────┼────────────┘
                       ▼
                    INGESTION
                       │
                       ▼
                  NORMALIZATION
                       │
                       ▼
             VERSION + PROVENANCE
                       │
                       ▼
                LOCAL LEGAL CORPUS
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
           FTS5              Embeddings
             │                   │
             ▼                   ▼
          BM25                 FAISS
             │                   │
             └─────────┬─────────┘
                       ▼
                  RERANKING
                       │
                       ▼
                  EVIDENCE
                       │
                       ▼
                  REASONING
                       │
                       ▼
                 VERIFICATION
                       │
              ┌────────┴────────┐
              ▼                 ▼
           ANSWER           DOCUMENT
```

---

# 27. Структура GitHub

```text
legal-ai-assistant/
│
├── ingestion/
│   ├── adapters/
│   │   ├── base.py
│   │   ├── pravo.py
│   │   ├── judicial.py
│   │   └── global_sources.py
│   ├── parsers/
│   ├── normalizers/
│   └── versioning/
│
├── agents/
│   ├── external_agent.py
│   ├── internal_agent.py
│   ├── global_agent.py
│   └── orchestrator.py
│
├── retrieval/
│   ├── lexical.py
│   ├── semantic.py
│   ├── hybrid.py
│   └── reranker.py
│
├── evidence/
│   ├── builder.py
│   ├── claim_graph.py
│   └── verifier.py
│
├── reasoning/
│   ├── planner.py
│   ├── analyzer.py
│   └── answer.py
│
├── drafting/
│   ├── templates/
│   ├── generator.py
│   └── verifier.py
│
├── storage/
│   ├── database.py
│   └── vector_store.py
│
├── evaluation/
│   ├── dataset.py
│   ├── retrieval_metrics.py
│   ├── evidence_metrics.py
│   └── evaluation.py
│
├── api/
│   └── main.py
│
├── ui/
│   └── app.py
│
├── tests/
│
├── data/
│
├── docs/
│
├── Dockerfile
├── requirements.txt
└── README.md
```

---

# 28. Roadmap

## Phase 1 — Research Core

```text
[ ] Project structure
[ ] SQLite schema
[ ] Document model
[ ] Source metadata
[ ] Basic ingestion
[ ] Text normalization
```

## Phase 2 — Internal Retrieval

```text
[ ] SQLite FTS5
[ ] BM25
[ ] E5 embeddings
[ ] FAISS
[ ] Hybrid retrieval
[ ] CrossEncoder
```

## Phase 3 — External Agent

```text
[ ] Source adapters
[ ] External retrieval
[ ] Metadata extraction
[ ] Version detection
[ ] Provenance
[ ] Content hash
```

## Phase 4 — Global Agent

```text
[ ] Language detection
[ ] Legal query transformation
[ ] Foreign retrieval
[ ] Original-language evidence
[ ] Translation
```

## Phase 5 — Evidence Layer

```text
[ ] Exact quote extraction
[ ] Claim extraction
[ ] Claim → Evidence graph
[ ] Citation verification
[ ] URL verification
[ ] Version verification
```

## Phase 6 — Orchestrator

```text
[ ] Query understanding
[ ] Research planning
[ ] Agent routing
[ ] Candidate merging
[ ] Relevance filtering
[ ] Legal reasoning
```

## Phase 7 — Document Drafting

```text
[ ] Document templates
[ ] Research-grounded drafting
[ ] Citation verification
[ ] DOCX export
[ ] PDF export
```

## Phase 8 — Evaluation

```text
[ ] Golden Dataset
[ ] Retrieval metrics
[ ] Evidence metrics
[ ] Generation metrics
[ ] Regression tests
```

## Phase 9 — UI

```text
[ ] Gradio interface
[ ] Source panel
[ ] Evidence panel
[ ] Citation links
[ ] Research status
[ ] Document generation
```

---

# 29. Как запустить MVP

## Требования

Минимально:

```text
Python 3.11+
Git
8–16 GB RAM
```

Для локальной LLM желательно:

```text
16+ GB RAM
```

GPU не является обязательным для первого прототипа.

---

## Установка

Клонировать репозиторий:

```bash
git clone <repository-url>
cd legal-ai-assistant
```

Создать виртуальное окружение:

```bash
python -m venv .venv
```

Активировать:

### Windows

```bash
.venv\Scripts\activate
```

### Linux / WSL

```bash
source .venv/bin/activate
```

Установить зависимости:

```bash
pip install -r requirements.txt
```

---

# 30. Запуск локальной LLM

Установить Ollama.

После установки проверить:

```bash
ollama --version
```

Затем загрузить выбранную локальную модель:

```bash
ollama pull <model>
```

Конкретная модель выбирается после тестирования на юридическом Golden Dataset.

Критерии выбора:

* качество русского языка;
* качество reasoning;
* factual consistency;
* способность работать с evidence;
* latency;
* RAM/VRAM;
* лицензия.

---

# 31. Запуск API

```bash
uvicorn api.main:app --reload
```

После запуска API предоставляет endpoint для research-запросов.

Пример логики:

```text
POST /research
```

Вход:

```json
{
    "query": "Юридический вопрос",
    "jurisdiction": "RU",
    "event_date": "2024-05-15",
    "include_foreign": false
}
```

---

# 32. Запуск UI

```bash
python ui/app.py
```

Интерфейс должен позволять:

```text
1. Ввести юридический вопрос
2. Указать юрисдикцию
3. Указать дату события
4. Запросить судебную практику
5. Запросить зарубежную практику
6. Получить research result
7. Открыть evidence
8. Перейти к первоисточнику
9. Создать юридический документ
```

---

# 33. Пример пользовательского сценария

Юрист:

> Может ли работодатель взыскать с работника ущерб, причинённый имуществу компании в мае 2024 года? Найди российское законодательство и судебную практику. Дополнительно сравни с Германией.

Система:

```text
QUERY UNDERSTANDING

Jurisdiction: Russia
Event date: 2024-05
Legal area: Labour Law
Case law: required
Foreign practice: Germany
```

Далее:

```text
AGENT 1
Russian external sources

AGENT 2
Internal legal corpus

AGENT 3
German legal practice
```

После чего Agent 4:

```text
Merge
 ↓
Deduplicate
 ↓
Filter
 ↓
Rerank
 ↓
Evidence
 ↓
Verify
 ↓
Reason
```

Результат:

```text
LEGAL CONCLUSION

...

LEGAL BASIS

...

CASE LAW

...

GERMAN COMPARISON

...

EVIDENCE

...

LIMITATIONS

...
```

И:

```text
[Open source]
[Open exact citation]
[Create legal memo]
[Create draft claim]
```

---

# 34. Что проект не обещает

Legal AI Assistant не должен заявлять:

* 100% юридическую точность;
* полное покрытие законодательства;
* полное покрытие судебной практики;
* абсолютную актуальность;
* отсутствие ошибок LLM;
* замену юриста;
* автоматическое принятие юридического решения.

Корректная формулировка:

> **Legal AI Assistant автоматически выполняет первичное юридическое исследование на основе подключённого и версионируемого корпуса источников, использует внешние источники для получения и актуализации информации, формирует доказательную базу и технически проверяет согласованность результата. Окончательная юридическая оценка и решение остаются за профессиональным пользователем.**

---

# 35. Что реально реализуется бесплатно

На первом этапе без коммерческих API можно реализовать:

```text
Python
SQLite
FTS5
FAISS
E5 embeddings
CrossEncoder
Ollama
Gradio
FastAPI
pytest
Docker
```

Также можно построить:

```text
Agent 1
Agent 2
Agent 3
Agent 4

+
Hybrid Retrieval
+
Reranking
+
Evidence Builder
+
Claim Verification
+
Versioning
+
Provenance
+
Document Drafting
```

Ограничение бесплатного MVP — прежде всего **размер и полнота корпуса**, вычислительные ресурсы и доступность автоматизированного получения конкретных внешних источников.

---

# 36. Что потребуется при финансировании

После доказательства MVP система может масштабироваться.

### Infrastructure

```text
Local SQLite
      ↓
PostgreSQL
```

```text
FAISS
      ↓
Qdrant / другой production vector store
```

### Models

```text
Small local LLM
      ↓
larger local / hosted / hybrid models
```

### Data

```text
Small verified corpus
      ↓
Large continuously updated corpus
```

### Processing

```text
Manual / scheduled ingestion
      ↓
Distributed ingestion
      ↓
Incremental indexing
```

### Enterprise

```text
Public Legal Corpus
        +
Private Company Corpus
        +
RBAC
        +
Audit Logs
        +
Data Isolation
```

---

# 37. Private Legal Corpus

Отдельный enterprise-контур:

```text
Company Documents
       ↓
PDF / DOCX / Scan
       ↓
Parser / OCR
       ↓
Cleaning
       ↓
Chunking
       ↓
Metadata
       ↓
Embeddings
       ↓
Private Index
```

Публичный и частный корпуса не должны смешиваться без явных правил доступа.

```text
PUBLIC CORPUS
       │
       │
       ├── legislation
       └── case law

PRIVATE CORPUS
       │
       ├── contracts
       ├── claims
       ├── correspondence
       └── internal documents
```

---

# 38. Security

Production-версия должна предусматривать:

* authentication;
* RBAC;
* audit logging;
* private corpus isolation;
* access control;
* encrypted storage;
* controlled external API access;
* temporary-file cleanup;
* source provenance;
* document-level permissions.

Конфиденциальные документы не должны автоматически отправляться во внешние LLM/API.

---

# 39. Главный технический принцип

```text
LLM ≠ Source of Law
```

Вместо этого:

```text
SOURCE
  ↓
DOCUMENT
  ↓
VERSION
  ↓
EVIDENCE
  ↓
LEGAL REASONING
  ↓
VERIFICATION
  ↓
ANSWER
```

LLM выполняет интеллектуальную обработку.

Источник юридического факта остаётся проверяемым документом.

---

# 40. Конкурентная среда

Российский рынок уже имеет сильные системы правовой информации и AI-сервисы.

Поэтому Legal AI Assistant не позиционируется как «первый AI для юристов».

## КонсультантПлюс

КонсультантПлюс в 2026 году развивает три AI-сервиса: «Задать вопрос», «Глубокий поиск» и «Проверка договоров». «Глубокий поиск» анализирует НПА и судебную практику и формирует структурированный правовой анализ с рекомендациями и ссылками.

## ГАРАНТ / ИСКРА

ИСКРА уже умеет:

* отвечать на вопросы по российскому законодательству;
* создавать правовые документы;
* анализировать судебную практику;
* работать с нормативно-технической документацией;
* продолжать диалог;
* сохранять результаты.

Сервис позиционируется как AI-решение, работающее на многомиллионном информационном банке ГАРАНТ.

## Caselook

Caselook специализируется на поиске и анализе судебной практики. Публично заявлены около 140 млн документов, многочисленные фильтры, мониторинг практики, AI-поиск, AI-сводки, AI-доводы и AI-ассистент для анализа судебных актов.

## Casebook

Casebook относится к смежному сегменту судебной аналитики: мониторинг дел, анализ судебной активности и информации о компаниях, работа с судебными данными и оценкой рисков.

## Официальные государственные источники

Официальный интернет-портал правовой информации предоставляет официальное опубликование правовых актов, тексты актов с изменениями и интегрированный банк «Законодательство России».

---

# 41. Реалистичное отличие Legal AI Assistant

Важно не утверждать:

> «У конкурентов этого нет».

У крупных систем может существовать функциональность, которая не раскрывается публично.

Поэтому сравниваем не скрытую внутреннюю архитектуру, а **то, что мы можем реально построить и продемонстрировать**.

| Возможность                                  | Legal AI Assistant   |
| -------------------------------------------- | -------------------- |
| Юридический вопрос → структурированный ответ | Да                   |
| Внутренний юридический корпус                | Да                   |
| Внешние официальные источники                | Да                   |
| Динамическое подключение внешнего поиска     | Да                   |
| Международный поиск по запросу юриста        | Да                   |
| Поиск на языке оригинала                     | Да                   |
| Hybrid lexical + semantic retrieval          | Да                   |
| CrossEncoder reranking                       | Да                   |
| Version-aware retrieval                      | Да                   |
| Provenance                                   | Да                   |
| Exact quote extraction                       | Да                   |
| Claim → Evidence mapping                     | Да                   |
| Citation verification                        | Да                   |
| URL verification                             | Да                   |
| Version verification                         | Да                   |
| Source freshness monitoring                  | Да                   |
| Golden Dataset                               | Да                   |
| Reproducible evaluation                      | Да                   |
| Автоматическая подготовка документов         | Да                   |
| Private corpus                               | Да, в roadmap        |
| Production-scale infrastructure              | После финансирования |

---

# 42. Где находится наша практическая дифференциация

Мы не пытаемся конкурировать количеством документов с многолетними правовыми информационными системами.

Наша проектная дифференциация:

```text
                    LEGAL QUESTION
                          │
                          ▼
                 RESEARCH ORCHESTRATOR
                          │
       ┌──────────────────┼──────────────────┐
       ▼                  ▼                  ▼
   EXTERNAL            INTERNAL            GLOBAL
   SOURCES              CORPUS             PRACTICE
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ▼
                   HYBRID RETRIEVAL
                          │
                          ▼
                      RERANKING
                          │
                          ▼
                    EXACT EVIDENCE
                          │
                          ▼
                 CLAIM → EVIDENCE
                          │
                          ▼
                     VERIFICATION
                          │
                          ▼
                    LEGAL ANSWER
                          │
                          ▼
                  DOCUMENT DRAFT
```

То есть проект объединяет:

**Multi-Agent Research + Multi-Source Retrieval + Evidence Grounding + Verification + Legal Reasoning + Document Drafting.**

---

# 43. Что действительно можно доказать на MVP

Мы не должны пытаться доказать:

> «Legal AI Assistant лучше КонсультантПлюс».

Вместо этого проект должен доказать воспроизводимыми экспериментами:

### Retrieval

```text
BM25
vs
E5
vs
Hybrid
vs
Hybrid + CrossEncoder
```

### Evidence

```text
LLM citation
vs
Exact Evidence Builder
```

### Versioning

```text
Current version retrieval
vs
Historical version retrieval
```

### Generation

```text
LLM without evidence
vs
Evidence-grounded generation
```

### Verification

```text
Generated answer
vs
Verified answer
```

Именно такие эксперименты можно разместить в GitHub.

---

# 44. Ключевая гипотеза проекта

> **Юридический AI должен быть не просто генератором текста, а системой исследования, в которой каждый существенный вывод можно проследить до конкретного проверяемого источника.**

Полный pipeline:

```text
QUESTION
   ↓
UNDERSTAND
   ↓
PLAN
   ↓
SEARCH
   ↓
RETRIEVE
   ↓
RERANK
   ↓
EXTRACT EVIDENCE
   ↓
REASON
   ↓
VERIFY
   ↓
ANSWER
   ↓
DRAFT DOCUMENT
```

---

# 45. Итог

Legal AI Assistant — это не попытка заменить существующие правовые информационные системы.

Это проект **evidence-grounded multi-agent legal research system**, который может объединить:

```text
External Sources
       +
Internal Legal Knowledge
       +
International Practice
       +
Hybrid Retrieval
       +
Reranking
       +
Version Management
       +
Provenance
       +
Evidence Extraction
       +
Claim Verification
       +
Legal Reasoning
       +
Document Drafting
```

Главный принцип:

> **Система должна не просто найти ответ, а показать, почему этот ответ был сформирован, на каком документе он основан, какая редакция использована и где находится точный фрагмент первоисточника.**

А масштабирование проекта после финансирования происходит не через изменение основной концепции, а через увеличение:

* количества источников;
* объёма корпуса;
* вычислительных ресурсов;
* количества языков;
* качества моделей;
* глубины экспертной оценки;
* уровня безопасности;
* числа корпоративных интеграций.

**Итоговая архитектурная формула проекта:**

```text
Multi-Agent Legal Research
        +
Multi-Source Retrieval
        +
Version-Aware Knowledge
        +
Evidence Grounding
        +
Verification
        +
Legal Reasoning
        +
Document Drafting
```

> **Legal AI Assistant — от юридического вопроса к проверяемому исследованию и готовому рабочему документу.**
