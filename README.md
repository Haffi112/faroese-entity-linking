# Faroese Entity Linking Gold Standard

A manually annotated entity linking dataset for Faroese, the first of its kind. The dataset contains 1,010 examples with 1,813 entity linking predictions, each independently reviewed by two Faroese-speaking annotators.

This dataset accompanies the paper:

> **Entity Linking for Faroese Using Large Language Models with Web Search**
> Annika Simonsen, Iben Nyholm Debess, and Hafsteinn Einarsson.
> *RESOURCEFUL 2026 Workshop.*

## Dataset Description

The dataset provides entity linking annotations for Faroese text from the [Sosialurin](https://www.sosialurin.fo/) news corpus. Entity mentions were identified using an existing Faroese NER model ([ScandiBERT-NER](https://huggingface.co/vesteinn/ScandiBERT-NER)) and linked to Wikipedia pages using GPT-5 with integrated web search through a three-tier fallback strategy (Faroese Wikipedia, English Wikipedia, any Wikipedia).

Two annotators independently evaluated each predicted entity link and classified it as **correct**, **incorrect**, or **uncertain**.

### Statistics

| Metric | Value |
|--------|-------|
| Examples | 1,010 |
| Entity predictions | 1,813 |
| Entity types | Person, Location, Organization, Miscellaneous, Date, Money, Time, Percent |
| Inter-annotator agreement | 89.9% (Cohen's Kappa = 0.584; 0.706 excluding uncertain) |
| Both annotators agree correct | 1,353 predictions |
| Both annotators agree incorrect | 126 predictions |

### Entity Type Distribution (in the 1,010-example subset)

| Type | Count |
|------|-------|
| Organization | 562 |
| Location | 530 |
| Person | 485 |
| Date | 108 |
| Miscellaneous | 59 |
| Money | 43 |
| Percent | 14 |
| Time | 12 |

## File Format

The dataset is provided as a JSONL file (`faroese_entity_linking_gold.jsonl`), with one JSON object per line.

### Schema

```json
{
  "example_id": "faroese_3063",
  "text": "Tað verður at kalla ikki gjørt í Suðuroy .",
  "entities": [
    {
      "name": "Suðuroy",
      "text": "Suðuroy",
      "type": "Location"
    }
  ],
  "predictions": [
    {
      "entity": "Suðuroy",
      "link": "Suðuroy >> fo",
      "annotator_1": "correct",
      "annotator_2": "correct"
    }
  ]
}
```

### Fields

| Field | Description |
|-------|-------------|
| `example_id` | Unique identifier for the example |
| `text` | The original Faroese text (from Sosialurin news articles) |
| `entities` | List of named entities identified by the NER model, each with `name`, `text` (surface form), and `type` |
| `predictions` | List of entity linking predictions from GPT-5 |
| `predictions[].entity` | Entity name |
| `predictions[].link` | Wikipedia link in format `"PageTitle >> lang_code"` (e.g., `"Tórshavn >> fo"`) or empty string `""` if no Wikipedia page was found |
| `predictions[].annotator_1` | First annotator's judgment: `"correct"`, `"incorrect"`, or `"uncertain"` |
| `predictions[].annotator_2` | Second annotator's judgment: `"correct"`, `"incorrect"`, or `"uncertain"` |

### Link Format

Wikipedia links follow the format `"PageTitle >> lang_code"` where:
- `PageTitle` is the Wikipedia article title
- `lang_code` is the two-letter Wikipedia language code (e.g., `fo` for Faroese, `en` for English, `de` for German)

Examples:
- `"Tórshavn >> fo"` — Faroese Wikipedia article for Tórshavn
- `"Faroe Islands >> en"` — English Wikipedia article for the Faroe Islands
- `""` — No appropriate Wikipedia page exists

### Annotation Labels

| Label | Definition |
|-------|------------|
| `correct` | The Wikipedia link accurately identifies the entity in context. For empty links, this means no appropriate Wikipedia page exists. |
| `incorrect` | The link is wrong, irrelevant, or the entity should have been linked but was not. |
| `uncertain` | The annotator could not confidently determine correctness. |

## Usage

### Loading the Dataset

```python
import json

with open("faroese_entity_linking_gold.jsonl") as f:
    dataset = [json.loads(line) for line in f]

# Filter to examples where both annotators agreed
agreed_correct = [
    (ex, pred)
    for ex in dataset
    for pred in ex["predictions"]
    if pred["annotator_1"] == "correct" and pred["annotator_2"] == "correct"
]

print(f"Gold standard (both agree correct): {len(agreed_correct)} predictions")
```

### Computing Metrics

```python
from collections import Counter

labels = Counter()
for ex in dataset:
    for pred in ex["predictions"]:
        a1, a2 = pred["annotator_1"], pred["annotator_2"]
        labels[(a1, a2)] += 1

for (a1, a2), count in labels.most_common():
    print(f"  Annotator 1={a1}, Annotator 2={a2}: {count}")
```

## Source Data

The source texts come from the Faroese NER dataset by [Snæbjarnarson et al. (2023)](https://aclanthology.org/2023.nodalida-1.61/), which contains named entity annotations for Faroese news articles from [Sosialurin](https://www.sosialurin.fo/).

Entity mentions were linked to Wikipedia using GPT-5 (OpenAI) via the OpenRouter API with integrated web search, following a three-tier fallback strategy:
1. Faroese Wikipedia (fo.wikipedia.org)
2. English Wikipedia (en.wikipedia.org)
3. Any Wikipedia language edition

## Citation

If you use this dataset, please cite:

```bibtex
@inproceedings{simonsen2026faroese,
  title={Entity Linking for Faroese Using Large Language Models with Web Search},
  author={Simonsen, Annika and Debess, Iben Nyholm and Einarsson, Hafsteinn},
  booktitle={Proceedings of the RESOURCEFUL 2026 Workshop},
  year={2026}
}
```

## License

This dataset is released under the [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) license. The source texts are from the Sosialurin newspaper; please respect the original data licensing terms.
