# ⚡ AI Circuit Design Assistant

> An AI-powered engineering assistant that turns a natural-language circuit description — in **Arabic or English** — into a component list, a step-by-step wiring guide, and a rendered 2D schematic. Built with Gemini and `schemdraw`, running entirely in Google Colab.

<p align="center">
  <img src="./circuit_017d71e7.svg" alt="Generated LM7805 voltage regulator circuit with LED indicator" width="600"/>
</p>

<p align="center"><em>↑ Live output for the prompt: "12V to 5V Voltage Regulator using LM7805 with LED Indicator"</em></p>

---

## 🚀 Try It Now

**[Open in Google Colab →](https://colab.research.google.com/drive/1MQvJLSVaN_1NwpSWyYx2gNddkIEPVz0j?usp=sharing)**

No setup required — just add your own [Gemini API key](https://aistudio.google.com/apikey) via Colab's Secrets Manager and run the cells top to bottom.

---

## 💡 The Problem

Beginners and even experienced engineers lose time translating a circuit idea into a correct schematic — picking the right components, values, and connections, especially for standard building blocks like voltage regulators, sensor interfaces, or LED indicators. Existing AI tools can describe a circuit in text, but none reliably *draw* one.

## ✅ The Solution

This assistant closes that gap by having the LLM generate structured, **executable** `schemdraw` code — not an image, not a description of an image, but real vector schematic code that gets validated and rendered into an actual SVG diagram.

---

## 🧠 How It Works

```
User Prompt (AR/EN)
       │
       ▼
┌─────────────────────┐        ┌──────────────────────┐
│  Explanation Call    │        │     Code Call         │
│  (Gemini 2.5 Flash)  │        │  (Gemini 2.5 Flash)   │
│  → Markdown sections │        │  → schemdraw code     │
└─────────────────────┘        └──────────┬────────────┘
                                            ▼
                                 ┌─────────────────────┐
                                 │  Safety Validator     │
                                 │  (blocks os/subprocess│
                                 │   /eval/open etc.)    │
                                 └──────────┬────────────┘
                                            ▼
                                 ┌─────────────────────┐
                                 │ Isolated Subprocess   │
                                 │ Renderer (schemdraw)  │
                                 └──────────┬────────────┘
                                            ▼
                              ✅ Success → SVG displayed
                              ❌ Failure → error fed back to
                                 Gemini for a self-healing retry
                                 (up to 3 attempts)
```

**Key design decision:** the explanation and the schematic code are generated in **two separate API calls**, not one. Combining them into a single response caused the model to truncate the code mid-generation once the explanation grew long — splitting them removed that failure mode entirely.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| LLM | Google Gemini 2.5 Flash |
| Schematic rendering | Python `schemdraw` |
| Execution environment | Google Colab |
| Code isolation | `subprocess` (never `exec()` in-kernel) |
| Secrets management | Colab `userdata` Secrets Manager |

---

## ✨ Features

- 🌐 **Bilingual** — accepts and explains circuits in Arabic or English
- 📋 Structured output: component table, wiring steps, technical specs, and safety notes
- 🖼️ Real rendered SVG schematic, not a text description
- 🔁 **Self-healing generation** — if the generated code fails, the actual error is sent back to Gemini for a targeted fix (not a blind retry)
- 🔒 **Sandboxed execution** — generated code runs in an isolated subprocess with a forbidden-pattern filter (blocks `os.`, `subprocess`, `eval(`, `open(`, etc.) before it's ever executed
- 🔑 API key stored via Colab Secrets — never hardcoded or committed


---

## ⚠️ Lessons Learned / Known Constraints

- LLMs reliably hallucinate `schemdraw` methods that don't exist (`.connect()`, `.wire()`, `.pin()`) unless the prompt explicitly whitelists real API elements and includes a worked example — the system prompt in `prompt_engine.py` documents this in detail.
- `schemdraw`'s direction methods (`.up()/.down()/.left()/.right()`) take no numeric length argument — a common model mistake that's now explicitly forbidden in the prompt.
- Splitting explanation and code generation into separate calls avoids token-budget truncation on longer bilingual responses.

## 🔮 Future Work

- Export generated schematics as Netlists compatible with Proteus / Altium
- Persistent conversation history for iterative edits ("add a second LED")
- Component datasheet lookups for automatic value validation

---

## 👤 Author

**Mark Medhat & Goerg Michael** 
