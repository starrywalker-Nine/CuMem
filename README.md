# CuMem

**A Tool-Grounded Autobiographical Memory Agent for Dietary Decision Support in Wilson Disease**

CuMem studies how a dietary-support agent can ask for missing information, track changing patient-specific records, and provide more reliable personalized recommendations across repeated interactions.

This repository currently presents **experimental results and selected worked examples** associated with the manuscript above. Detailed methods, implementation, prompts, and the full reproducibility package will be added in a later release.

## Highlights

- **87.6% conditional copper-value accuracy**, compared with 70.5% for RAG.
- **99.6% required-clarification recall** on information-insufficient queries.
- **53.3% longitudinal state-task accuracy**, compared with 29.6% for RAG, with improvements across all seven evaluated backbones.
- **98.6% clinician-rated safe-output coverage**, compared with 75.2% for RAG, in simulated recipe-recommendation cases.

These findings describe performance in clinician-authored simulated scenarios. Clinical effectiveness has not been evaluated.

## Experimental results

We compare CuMem with chain-of-thought prompting (**CoT**) and retrieval-augmented generation (**RAG**) across seven language-model backbones. The results below are reported in the associated manuscript; this release summarizes them without disclosing the internal implementation.

| Experiment | Endpoint ↑ | CoT | RAG | CuMem | CuMem − RAG, with 95% CI |
| :--- | :--- | ---: | ---: | ---: | :--- |
| 1 · Short dietary queries | Conditional copper-value accuracy | 16.4% | 70.5% | **87.6%** | +17.1 pp [5.4, 28.4] |
| 1 · Short dietary queries | Required-clarification recall | 97.3% | 97.2% | **99.6%** | +2.4 pp [0.4, 5.0] |
| 2 · Longitudinal state tracking | Gold-answer task accuracy | 22.9% | 29.6% | **53.3%** | +23.6 pp [17.2, 29.9] |
| 3 · Recipe recommendations | Comprehensive quality, 1–5 | 2.652 | 2.940 | **3.408** | +0.468 [−0.170, 1.079] |
| 3 · Recipe recommendations | Clinician-rated safe-output coverage | 71.0% | 75.2% | **98.6%** | +23.3 pp [13.6, 34.5] |
| 3 · Recipe recommendations | Copper-control score, 1–5 | 2.067 | 2.367 | **2.969** | +0.602 [0.302, 0.933] |

**Reading the table.** Higher is better; pp means percentage points. Differences and confidence intervals are reported from paired analyses in the manuscript. Differences use unrounded estimates, so they may differ slightly from subtraction of the displayed values. The comprehensive-quality interval includes zero: its higher point estimate does not establish a clear improvement over RAG. The intervals for clarification recall, state-task accuracy, safe-output coverage, and copper-control score exclude zero.

### Experiment 1: answering when possible, asking when necessary

The suite contains **300 single-turn cases** evaluated with an empty memory state: **38 directly answerable cases** and **262 cases requiring clarification**. Thus, 87.3% of this constructed suite requires additional information before a reliable copper determination is possible.

CuMem combines higher conditional copper-value accuracy with 99.6% recall for required clarification. The two endpoints measure different behaviors: numerical correctness when an answer can be evaluated, and recognition of missing information. **87.6% is a conditional endpoint, not accuracy over all 300 cases.** It should be interpreted alongside numerical answer coverage and unnecessary refusal, whose full reporting remains part of the forthcoming evaluation package.

### Experiment 2: maintaining information across interactions

This suite uses **60 clinician-authored simulated trajectories**. Of **478 issued probes**, **467 are scored** against gold answers; 11 designated not applicable are excluded. CuMem improves overall accuracy by **23.6 percentage points over RAG**, with a positive difference under every evaluated backbone.

The task-family breakdown shows where the gains occur and where work remains:

| Probe family | Scored probes | CoT | RAG | CuMem |
| :--- | ---: | ---: | ---: | ---: |
| Update confirmation | 108 | 15.6% | 28.2% | **77.8%** |
| Highest copper-source query | 60 | 25.0% | 38.1% | **78.3%** |
| Cumulative-intake query | 105 | 1.1% | 2.6% | **25.7%** |
| Budget query | 59 | 87.2% | 90.3% | **100.0%** |
| Treatment-rule violation | 60 | 31.7% | 40.5% | **48.6%** |
| Deletion confirmation | 60 | 0.2% | **2.1%** | 0.0% |
| Patient-state violation | 15 | 21.9% | **23.8%** | 18.1% |

The largest gains are in update confirmation and highest-source identification. Cumulative-intake accuracy improves but remains limited. CuMem refuses all deletion-confirmation probes in this evaluation, receiving zero gold-answer accuracy on that family. A refusal receives no correctness credit when the probe requires a determinate answer.

### Experiment 3: personalized recipe recommendations

The suite crosses **60 open-ended cases × 3 systems × 7 backbones**, producing **1,260 recommendations** for clinician assessment. These are repeated evaluations of 60 cases, rather than 1,260 independent patient scenarios.

**Safe-output coverage** is the proportion of recommendations that clinicians do **not** flag as potentially harmful in the simulated setting. CuMem reaches 98.6%, compared with 75.2% for RAG. Copper-control quality also improves, while the comprehensive-quality difference remains uncertain under its reported confidence interval.

## Worked examples: what useful clarification looks like

The examples below use selected Chinese case inputs and their clarification annotations. **Quoted assistant wording is an illustrative rewrite of the expected behavior, not a verbatim CuMem run or an additional experimental result.** Case identifiers link each example to the underlying case material; full run transcripts are reserved for a later release.

### 1. Ask about the uncertain item, preserve the known portion

**Case E1-B-003 · Before-meal query**

> **User:** 一把花生；50 克即食燕麦。  
> *A handful of peanuts; 50 g of instant oats.*
>
> **Illustrative clarification:** 燕麦的份量已明确。花生仁实际有多少克？“一把”的重量差异较大，请称量或补充克重，再计算这餐的铜量。

**Annotated requirement:** Request the peanut-kernel weight instead of assigning a fixed weight to “a handful.”

**Why this is useful:** The question targets the missing quantity and keeps the already specified 50 g of oats. It avoids presenting an unsupported numerical total.

### 2. Clarify a mixed dish and its accompaniments

**Case E1-B-004 · Takeaway meal**

> **User:** 一份黄焖鸡米饭；1 个脐橙。  
> *One serving of braised chicken with rice; one navel orange.*
>
> **Illustrative clarification:** 这份黄焖鸡米饭有哪些主要配料，各吃了多少？能提供商家的配料和份量信息吗？也请补充用油种类和用量，以及脐橙去皮后的可食重量。信息补齐后再计算铜量。

**Annotated requirement:** Obtain the vendor recipe or clarify components individually; request the edible orange weight. The case material also requires oil type and amount to be recorded separately.

**Why this is useful:** “One serving” does not establish ingredient quantities. The follow-up identifies the information needed for a calculation rather than substituting a generic takeaway estimate.

### 3. Clarify the consumed portion before recording intake

**Case E1-A-002 · After-meal record**

> **User:** 半盘香菇炒肉；200 克原味酸奶。  
> *Half a plate of mushrooms stir-fried with meat; 200 g of plain yogurt.*
>
> **Illustrative clarification:** 酸奶按你提供的 200 克记录。香菇炒肉中实际吃掉的香菇有多少克？“半盘”无法确定香菇份量；请补充实际食用量，再确认这道菜的铜摄入记录。

**Annotated requirement:** Request the mushroom weight and clarify the actual consumed amount.

**Why this is useful:** The follow-up separates a known portion from an uncertain component. An unresolved dish should not silently become a confirmed copper total in the longitudinal record.

### 4. Make portion uncertainty explicit during restaurant planning

**Case E3-051 · Restaurant recommendation**

**Simulated context:** Stable management phase; prefers malatang; limited spending budget; travelling and unable to cook.

> **User:** 今晚在麻辣烫店用餐，请给出选料方案。  
> *I will eat at a malatang restaurant tonight; please suggest a selection plan.*
>
> **Illustrative clarification:** 你准备选哪些配料，每种大约多少克或多少份？店家能否提供份量信息？目前外食份量还不明确，需要先确认配料和实际份量，再给出具体的选料方案。

**Annotated requirement:** When restaurant portions cannot be determined, state the uncertainty and request ingredient or portion information.

**Why this is useful:** A recommendation can sound personalized while still missing information needed to assess it. The manuscript describes an evaluated response to this case that omitted mandatory clarification and was flagged as potentially harmful. This example illustrates the missing clarification, rather than claiming that a particular CuMem transcript passed the case.

## What the current evidence supports

The evaluation supports **system-level improvements over the tested CoT and RAG baselines** in numerical grounding, required clarification, longitudinal state-task accuracy, and clinician-rated recommendation safety.

Several boundaries remain: overall state-task accuracy is 53.3%; two probe families do not improve over RAG; comprehensive recipe quality has an interval crossing zero; and component-specific contributions have not been isolated. Comparisons with additional memory agents and prospective evaluations with real patients remain future work.

## Release status

| Material | Current status |
| :--- | :--- |
| Aggregate experimental results and endpoint definitions | Included in this README |
| Task-family results and selected clarification examples | Included in this README |
| Original model outputs, run-level scores, and complete evaluation assets | Planned for a later release |
| Detailed method, architecture, prompts, tools, and source code | Planned for a later release |
| Installation and reproduction instructions | To accompany the implementation release |
| Public paper link and formal citation | To be added when available |

This is a results-focused research release. The examples are simulated and are intended to explain evaluation behavior, not to provide individual dietary advice.

## Associated manuscript

**CuMem: A Tool-Grounded Autobiographical Memory Agent for Dietary Decision Support in Wilson Disease**

A public manuscript link and citation metadata will be added here when available. This README does not imply a publication or acceptance status.
