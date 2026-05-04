# Methodology Breakdown

> Technical companion to the CoBICET 2023 paper. This document expands on the experimental design, prompt structure, and analytical framework so that other researchers can reproduce or extend the work.

---

## 1. Research question

> *How does GPT-4's output quality differ between English and Portuguese under an identical structured prompt, and what mechanism explains the difference?*

The hypothesis going in: outputs in non-English languages would show measurable degradation in content depth and linguistic complexity, traceable to the model's English-dominant pre-training distribution.

---

## 2. Experimental setup

| Parameter | Value | Rationale |
|---|---|---|
| **Model** | GPT-4 (LLM `code-davinci-003`) via ChatGPT public product | Frontier model at time of study; widely accessible |
| **Date of data collection** | April 2023 | Stable account state; pre-major-update window |
| **Account geolocation** | United States | Controls for any region-based account state |
| **Languages tested** | English (US) and Portuguese (BR) | Native and high-resource non-native pairing |
| **Prompt class** | In-Context Learning (ICL) | Forces the model to use prompt-side context, not just pre-training |
| **Temperature / sampling** | Default ChatGPT settings | Reproducibility within product defaults |

---

## 3. Prompt design — the Prompt-EDU framework

The prompt was structured in three explicit components, following the Prompt-EDU methodology (Ricieri et al., 2023). This is *not* a generic prompt — every part has a function.

### 3.1 Component: Context
> *"Create a hypothetical dialogue between Paulo Freire, the educator, and Mark Zuckerberg, CEO of Meta. This dialogue should use the characteristics of each of these two figures to construct the arguments. The theme of the dialogue should be: how will learning be modified by the integration of artificial intelligence technologies in schools and universities?"*

**Function:** establishes the task domain (education × technology) and the personas the model must instantiate.

### 3.2 Component: Purpose (intended generalization)
> *"In this hypothetical dialogue, each will follow this sequence of participation: 1 — one will ask the other 'what is your point of view on Artificial Intelligence in the educational formation of the citizen of the future?'"*

**Function:** scopes the generalization the model should make — a specific, answerable question rather than open-ended generation.

### 3.3 Component: Command key
> *"The response should be constructed with the personality and professional characteristics that each of them practices/practiced."*

**Function:** the explicit constraint that forces the model to retrieve and apply persona-specific knowledge — not just generic dialogue.

---

## 4. Why these characters

The selection of Paulo Freire and Mark Zuckerberg was a deliberately controlled stress test:

| Control axis | Selection rationale |
|---|---|
| **Pre-training cutoff coverage** | Both were globally famous well before September 2021 (the GPT-4 training cutoff at the time), ensuring the model has rich representations of both. |
| **Cultural-data balance** | One Brazilian, one American — controls for cultural-corpus imbalance as a confound. |
| **Cross-domain generalization** | Education vs. technology — forces the model to bridge two distant domains, which is exactly what ICL is designed to enable. |
| **Real-world recognizability** | Names that any human reader can validate against — enables qualitative evaluation by domain non-experts. |

---

## 5. Experimental conditions

```
                    ┌────────────────────────┐
                    │   Identical ICL prompt  │
                    │   (3 components above)  │
                    └────────────────────────┘
                                │
                ┌───────────────┴───────────────┐
                ▼                               ▼
        ┌──────────────┐               ┌──────────────┐
        │  Run in PT   │               │  Run in EN   │
        │   (Portuguese)│               │   (English)  │
        └──────────────┘               └──────────────┘
                │                               │
                ▼                               ▼
        ┌──────────────┐               ┌──────────────┐
        │  PT output   │               │  EN output   │
        └──────────────┘               └──────────────┘
                │                               │
        Ask GPT-4 to translate          Ask GPT-4 to translate
                ▼                               ▼
        ┌──────────────┐               ┌──────────────┐
        │ PT→EN trans. │               │ EN→PT trans. │
        └──────────────┘               └──────────────┘

             Re-analyze all four artifacts across
              content / context / language axes.
```

The cross-translation step is what isolates *language bias* from *prompt-design bias* and *account-state confounds*. If the gap were due to the prompt itself, translating would surface the same gap in the translated copies. It didn't — the gap was traceable to the language of generation, not the language of analysis.

---

## 6. Analytical framework — three dimensions

### 6.1 Content
*Did the model add, preserve, or lose substance after translation?*
- Count of distinct ideas raised
- Specificity of references (named works, concepts, examples)
- Argumentative depth
- **Finding:** English originals significantly more detailed; translation pulls them closer to the Portuguese original's depth.

### 6.2 Context
*Did the dialogue stay within the prompted parameters?*
- Did each persona stay in character?
- Did the conversation address the assigned question?
- Did the back-and-forth structure persist?
- **Finding:** Both languages preserved context robustly. The ICL framing successfully constrained the model's output across the language switch.

### 6.3 Language
*Vocabulary diversity and structural complexity.*
- Sentence length distribution
- Vocabulary breadth
- Use of domain-specific terminology
- **Finding:** English outputs used richer, more diverse vocabulary; Portuguese outputs leaned on common-register terms with less domain-specific terminology.

---

## 7. The mechanism: why the gap exists

From the paper (Discussion section):

> *"Since the GPT-3 version, OpenAI Inc. showed that the LLM processing of the GPT family neural network is always done in the English language. Therefore, what the AI does when receiving commands in other languages is to insert a double translation in the processing: one translation in the language→English direction for the input command, and another translation in the English→language direction for the output of results."*

This double-translation pipeline is **not user-controllable**. The model treats every non-English prompt as a non-native communication, which:
1. Adds two translation steps that carry their own information loss,
2. Triggers a more "literal" interpretation pattern (similar to automatic translators),
3. Disrupts the attention mechanism's ability to weight contextual cues that are linguistically idiomatic to the source language.

---

## 8. Connection to formal NLP bias literature

Hovy & Prabhumoye (2021) identified five sources of bias in NLP:

1. **Data** — what's in the training corpus (heavily English).
2. **Annotation processes** — how training data was labeled.
3. **Input representations** — how prompts are tokenized and embedded.
4. **Models** — architectural choices that may amplify imbalances.
5. **Task design** — how the task itself can encode assumptions.

This study primarily surfaces sources **(1)** and **(3)**, with implications for **(4)**.

Liang et al. (2023, arXiv:2304.02819) further documented that GPT-based detectors are biased against non-native English writers — a complementary finding from the *detection* side that aligns with our finding from the *generation* side.

---

## 9. Limitations & extensions

The original paper is a qualitative case study. Natural extensions:

- **Quantitative metrics:** Add type-token ratio, BLEU/BERTScore against reference, Flesch-Kincaid complexity scores.
- **Statistical robustness:** Multiple seeds per condition; significance testing on metric distributions.
- **Language coverage:** Expand beyond Portuguese to Spanish, French, Mandarin, Arabic, Hindi — test whether the gap is monotonic with training-corpus representation.
- **Newer models:** Replicate on GPT-4o, Claude 4.x, Gemini 2.x, Llama 4, etc., and check whether mitigation strategies (multilingual fine-tuning, larger non-English corpora) close the gap.
- **Prompt-design sweep:** Vary ICL prompt complexity to see whether richer scaffolding compensates more in non-English than in English.

---

## 10. Reproducing the experiment

To reproduce:
1. Use any frontier LLM with public API access.
2. Translate the three-component prompt above into your two target languages.
3. Issue both prompts to a fresh, geolocation-controlled account.
4. Apply the cross-translation step.
5. Score outputs on the three dimensions in §6.

Total experiment time: ~30 min for the model interactions, several hours for the qualitative analysis.

---

## Author

**Fabiano Rodrigues de Souza, PhD** — first author and methodology designer.
Contact: mrdesouzaphd@gmail.com
