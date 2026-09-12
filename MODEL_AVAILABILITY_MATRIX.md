# Amazon Bedrock Model Availability Matrix: Complete Engine & Japan Geo Survey

**Target Regions:** Tokyo (`ap-northeast-1`) & Osaka (`ap-northeast-3`)  
**Data Residency Policy:** 100% In-Country Processing Guarantee (Strict Data Sovereignty)  
**Engines Covered:** Amazon Bedrock Mantle (`bedrock-mantle`) & Amazon Bedrock Runtime (`bedrock-runtime`)  
**Last Updated:** September 2026  

---

## 1. Executive Summary & Routing Philosophy

For an internal engineering development gateway, Amazon Bedrock operates across two distinct data planes:
1. **Bedrock Mantle (`bedrock-mantle.<region>.api.aws`):**  
   Next-gen distributed inference engine offering **native OpenAI (Responses, Chat Completions) & Anthropic (Messages) wire protocols**, token-governed queues (no RPM throttle limits; governed by input/output TPM quotas), and stateful conversation support. **Primary engine for interactive reasoning and high-throughput agent loops.**  
   *Regional Constraint:* Mantle is deployed in Tokyo (`ap-northeast-1`), but is **NOT available in Osaka (`ap-northeast-3`)**.
2. **Bedrock Runtime (`bedrock-runtime.<region>.amazonaws.com`):**  
   Classic Bedrock data plane offering the **Converse API**, `InvokeModel`, `/openai/v1` compatibility routes, and media endpoints. **Required engine for vector embeddings (RAG), media tools, and flagship coding agents (`Claude Sonnet 4.5 / 4.6`, `DeepSeek-R1`, `Llama 3.3`) that are not hosted on Mantle.**

### Japan Geo Residency Rules
* **✅ Compliant (Japan Geo):** The model runs **In-Region Tokyo (`ap-northeast-1`)**, **In-Region Osaka (`ap-northeast-3`)**, or via a **Japan Cross-Region profile (`jp.` prefix)**. When using `jp.` profiles, requests load-balance strictly between Tokyo and Osaka with a 100% guarantee that prompts and completions never leave Japan.
* **⚠️ Limited (APAC):** Available only in APAC regional profiles (`apac.` prefix, e.g. Nova Pro, Claude Sonnet 4), which route dynamically across Asia-Pacific (Sydney, Singapore, Tokyo). Requires explicit compliance waiver.
* **❌ Non-Compliant (Overseas):** Available only in US (`us.` / `us-east-1`, `us-west-2`), EU (`eu.`), or Global (`global.`) profiles. Prompts egress Japanese sovereign borders; strictly blocked without a Tier 3 SecOps waiver.

---

## 2. Complete Foundation Model & Engineering Survey

Below is the verified survey of Foundation Models across both Bedrock execution engines and Japan Geo residency status:

| Provider | Model Name | Bedrock Mantle | Bedrock Runtime | Japan Geo Status | In-Country Profile / Endpoint Format | Modality & Engineering Use |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **AI21 Labs** | Jamba 1.5 Large | ❌ No | ✅ Yes | ❌ Non-Compliant | `ai21.jamba-1-5-large-v1:0` (US) | Long-context SSM-Transformer text |
|  | Jamba 1.5 Mini | ❌ No | ✅ Yes | ❌ Non-Compliant | `ai21.jamba-1-5-mini-v1:0` (US) | Fast low-cost SSM-Transformer |
| **Amazon** | Amazon Nova Multimodal Embeddings | ❌ No | ✅ Yes | ❌ Non-Compliant | `amazon.nova-2-multimodal-embeddings-v1:0` (US) | Multimodal vector embeddings for RAG |
|  | Nova 2 Lite | ❌ No | ✅ Yes | **✅ Compliant** | `jp.amazon.nova-2-lite-v1:0` | Ultra-fast text/code utility |
|  | Nova 2 Sonic | ❌ No | ✅ Yes | **✅ Compliant** | `amazon.nova-2-sonic-v1:0` (`ap-northeast-1`) | Low-latency speech-to-speech interaction |
|  | Nova Canvas | ❌ No | ✅ Yes | ❌ Non-Compliant | `amazon.nova-canvas-v1:0` (US) | Enterprise image generation & editing |
|  | Nova Lite | ❌ No | ✅ Yes | **✅ Compliant** | `amazon.nova-lite-v1:0` (`ap-northeast-1`) | Cost-effective text and code analysis |
|  | Nova Micro | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.amazon.nova-micro-v1:0` | Ultra-low latency text summarization |
|  | Nova Premier | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.amazon.nova-premier-v1:0` | High-complexity multi-step reasoning |
|  | Nova Pro | ❌ No | ✅ Yes | ⚠️ Limited | `apac.amazon.nova-pro-v1:0` (APAC) | Multimodal code inspection & analysis |
|  | Nova Reel | ❌ No | ✅ Yes | ❌ Non-Compliant | `amazon.nova-reel-v1:0` (US) | Video generation (Egresses Japan) |
|  | Nova Sonic | ❌ No | ✅ Yes | **✅ Compliant** | `amazon.nova-sonic-v1:0` (`ap-northeast-1`) | Speech generation and transcription |
|  | Titan Embeddings G1 - Text | ❌ No | ✅ Yes | **✅ Compliant** | `amazon.titan-embed-text-v1` (`ap-northeast-1`) | Classic text embeddings |
|  | Titan Embeddings G1 - Text v2 | ❌ No | ✅ Yes | ❌ Non-Compliant | `amazon.titan-embed-g1-text-02` (US) | Classic text embeddings |
|  | Titan Image Generator G1 v2 | ❌ No | ✅ Yes | ❌ Non-Compliant | `amazon.titan-image-generator-v2:0` (US) | Image generation & inpainting |
|  | Titan Multimodal Embeddings G1 | ❌ No | ✅ Yes | ❌ Non-Compliant | `amazon.titan-embed-image-v1` (US) | Text & image hybrid vector search |
|  | Titan Text Embeddings V2 | ❌ No | ✅ Yes | **✅ Compliant** | `amazon.titan-embed-text-v2:0` (`ap-northeast-1`, `ap-northeast-3`) | **Primary AWS codebase RAG embeddings** |
| **Anthropic** | Claude 3 Haiku | ❌ No | ✅ Yes | **✅ Compliant** | `anthropic.claude-3-haiku-20240307-v1:0` (`ap-northeast-1`) | Legacy lightweight task execution |
|  | Claude 3.5 Haiku | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.anthropic.claude-3-5-haiku-20241022-v1:0` | Sub-second inline code completion |
|  | Claude Fable 5 | ✅ Yes | ✅ Yes | ❌ Non-Compliant | `us.anthropic.claude-fable-5` | Creative writing & dialogue generation |
|  | Claude Fable 5.1 | ✅ Yes | ✅ Yes | ❌ Non-Compliant | `us.anthropic.claude-fable-5-1` | Experimental creative/prose model |
|  | Claude Haiku 4.5 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-haiku-4-5-20251001-v1:0` | **Ultra-fast coding linter & test runner** |
|  | Claude Mythos 5 | ✅ Yes | ❌ No | ❌ Non-Compliant | `anthropic.claude-mythos-5` (US) | Frontier autonomous agent preview |
|  | Claude Mythos 5.1 | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.anthropic.claude-mythos-5-1` | Next-gen autonomous agent reasoning |
|  | Claude Mythos Preview | ✅ Yes | ❌ No | ❌ Non-Compliant | `anthropic.claude-mythos-preview` (US) | Early preview reasoning engine |
|  | Claude Opus 4.1 | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.anthropic.claude-opus-4-1-20250805-v1:0` | Deep technical writing & documentation |
|  | Claude Opus 4.5 | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.anthropic.claude-opus-4-5-20251101-v1:0` | Complex code synthesis & algorithmic math |
|  | Claude Opus 4.6 | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.anthropic.claude-opus-4-6-v1` | Deep reasoning and formal verification |
|  | Claude Opus 4.7 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-opus-4-7` | Complex multi-repository refactoring |
|  | Claude Opus 4.8 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-opus-4-8` | Advanced architecture analysis & synthesis |
|  | Claude Opus 5 | ✅ Yes | ✅ Yes | ❌ Non-Compliant | `us.anthropic.claude-opus-5` | Maximum frontier intelligence & math |
|  | Claude Sonnet 4 | ❌ No | ✅ Yes | ⚠️ Limited | `apac.anthropic.claude-sonnet-4-20250514-v1:0` (APAC) | Reliable coding and logic reasoning |
|  | Claude Sonnet 4.5 | ❌ No | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-sonnet-4-5-20250929-v1:0` | **Current standard IDE agent (Cursor/Cline)** |
|  | Claude Sonnet 4.6 | ❌ No | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-sonnet-4-6` | High-performance full-stack coding |
|  | Claude Sonnet 5 | ✅ Yes | ✅ Yes | ❌ Non-Compliant | `us.anthropic.claude-sonnet-5` | **Next-gen primary coding & dev agent** |
| **Cohere** | Command R | ❌ No | ✅ Yes | ❌ Non-Compliant | `cohere.command-r-v1:0` (US) | Multilingual document processing |
|  | Command R\+ | ❌ No | ✅ Yes | ❌ Non-Compliant | `cohere.command-r-plus-v1:0` (US) | Multilingual document processing |
|  | Embed English | ❌ No | ✅ Yes | **✅ Compliant** | `cohere.embed-english-v3` (`ap-northeast-1`) | English-only vector indexing |
|  | Embed Multilingual | ❌ No | ✅ Yes | **✅ Compliant** | `cohere.embed-multilingual-v3` (`ap-northeast-1`) | **Gold-standard Japanese & Codebase RAG** |
|  | Embed v4 | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.cohere.embed-v4:0` | High-dimensional dense vector embeddings |
|  | Rerank 3.5 | ❌ No | ✅ Yes | **✅ Compliant** | `cohere.rerank-v3-5:0` (`ap-northeast-1`) | **Search re-ranking for engineering RAG** |
| **DeepSeek** | DeepSeek V3.2 | ✅ Yes | ✅ Yes | **✅ Compliant** | `deepseek.v3.2` (`ap-northeast-1`) | High-efficiency general-purpose coding |
|  | DeepSeek-R1 | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.deepseek.r1-v1:0` | **Open-weight deep reasoning & math synthesis** |
|  | DeepSeek-V3.1 | ✅ Yes | ✅ Yes | **✅ Compliant** | `deepseek.v3-v1:0` (`ap-northeast-1`) | Cost-effective open-weight assistant |
| **Google** | Gemma 3 12B IT | ✅ Yes | ✅ Yes | **✅ Compliant** | `google.gemma-3-12b-it` (`ap-northeast-1`) | Instruction-tuned local development |
|  | Gemma 3 27B PT | ✅ Yes | ✅ Yes | **✅ Compliant** | `google.gemma-3-27b-it` (`ap-northeast-1`) | Pre-trained open research foundation |
|  | Gemma 3 4B IT | ✅ Yes | ✅ Yes | **✅ Compliant** | `google.gemma-3-4b-it` (`ap-northeast-1`) | Micro test runner & unit test scaffolding |
|  | Gemma 4 26B-A4B | ✅ Yes | ❌ No | ❌ Non-Compliant | `google.gemma-4-26b-a4b` (US) | MoE open model for developer testing |
|  | Gemma 4 31B | ✅ Yes | ❌ No | ❌ Non-Compliant | `google.gemma-4-31b` (US) | High-capacity open weights for research |
|  | Gemma 4 E2B | ✅ Yes | ❌ No | ❌ Non-Compliant | `google.gemma-4-e2b` (US) | Edge-optimized lightweight testing |
| **Meta** | Llama 3 70B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `meta.llama3-70b-instruct-v1:0` (US) | Legacy open-weight benchmark baseline |
|  | Llama 3 8B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `meta.llama3-8b-instruct-v1:0` (US) | Legacy lightweight assistant |
|  | Llama 3.1 405B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.meta.llama3-1-405b-instruct-v1:0` | Massive open model (Egresses Japan) |
|  | Llama 3.1 70B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.meta.llama3-1-70b-instruct-v1:0` | Proven enterprise open coding foundation |
|  | Llama 3.1 8B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.meta.llama3-1-8b-instruct-v1:0` | Cost-effective rapid test runner |
|  | Llama 3.2 11B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.meta.llama3-2-11b-instruct-v1:0` | Vision & text multimodal open assistant |
|  | Llama 3.2 1B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.meta.llama3-2-1b-instruct-v1:0` | Instant edge completion & simple linting |
|  | Llama 3.2 3B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.meta.llama3-2-3b-instruct-v1:0` | Lightweight text parsing & commit tools |
|  | Llama 3.2 90B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.meta.llama3-2-90b-instruct-v1:0` | Multimodal code & diagram architecture |
|  | Llama 3.3 70B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.meta.llama3-3-70b-instruct-v1:0` | **Top open-weights coding model** |
|  | Llama 4 Maverick 17B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.meta.llama4-maverick-17b-instruct-v1:0` | **Next-gen autonomous open coding agent** |
|  | Llama 4 Scout 17B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.meta.llama4-scout-17b-instruct-v1:0` | Next-gen reasoning & search assistant |
| **MiniMax** | MiniMax M2 | ✅ Yes | ✅ Yes | **✅ Compliant** | `minimax.minimax-m2` (`ap-northeast-1`) | Baseline long-context reasoning |
|  | MiniMax M2.1 | ✅ Yes | ✅ Yes | **✅ Compliant** | `minimax.minimax-m2.1` (`ap-northeast-1`) | Dialogue & code translation |
|  | MiniMax M2.5 | ✅ Yes | ✅ Yes | **✅ Compliant** | `minimax.minimax-m2.5` (`ap-northeast-1`) | Long-context Chinese/English synthesis |
| **Mistral AI** | Devstral 2 123B | ✅ Yes | ✅ Yes | **✅ Compliant** | `mistral.devstral-2-123b` (`ap-northeast-1`) | **Specialized software engineering LLM** |
|  | Magistral Small 2509 | ✅ Yes | ✅ Yes | **✅ Compliant** | `mistral.magistral-small-2509` (`ap-northeast-1`) | Lightweight agent orchestration |
|  | Ministral 14B 3.0 | ✅ Yes | ✅ Yes | **✅ Compliant** | `mistral.ministral-3-14b-instruct` (`ap-northeast-1`) | High-density developer assistant |
|  | Ministral 3 8B | ✅ Yes | ✅ Yes | **✅ Compliant** | `mistral.ministral-3-8b-instruct` (`ap-northeast-1`) | Edge-friendly dev assistant |
|  | Ministral 3B | ✅ Yes | ✅ Yes | **✅ Compliant** | `mistral.ministral-3-3b-instruct` (`ap-northeast-1`) | Ultra-fast inline code suggester |
|  | Mistral 7B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `mistral.mistral-7b-instruct-v0:2` (US) | Classic lightweight code generation |
|  | Mistral Large | ❌ No | ✅ Yes | ❌ Non-Compliant | `mistral.mistral-large-2402-v1:0` (US) | Complex multilingual logic & reasoning |
|  | Mistral Large 3 | ✅ Yes | ✅ Yes | **✅ Compliant** | `mistral.mistral-large-3-675b-instruct` (`ap-northeast-1`) | High-precision coding & multilingual math |
|  | Mistral Small | ❌ No | ✅ Yes | ❌ Non-Compliant | `mistral.mistral-small-2402-v1:0` (US) | Fast structured JSON output |
|  | Mixtral 8x7B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `mistral.mixtral-8x7b-instruct-v0:1` (US) | Sparse MoE benchmark standard |
|  | Pixtral Large | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.mistral.pixtral-large-2502-v1:0` | Multimodal visual code & diagram parser |
|  | Voxtral Mini 3B 2507 | ✅ Yes | ✅ Yes | **✅ Compliant** | `mistral.voxtral-mini-3b-2507` (`ap-northeast-1`) | Voice and audio instruction following |
|  | Voxtral Small 24B 2507 | ✅ Yes | ✅ Yes | **✅ Compliant** | `mistral.voxtral-small-24b-2507` (`ap-northeast-1`) | High-fidelity audio & speech intelligence |
| **Moonshot AI** | Kimi K2 Thinking | ✅ Yes | ✅ Yes | **✅ Compliant** | `moonshot.kimi-k2-thinking` (`ap-northeast-1`) | Deep reasoning & long-document audit |
|  | Kimi K2.5 | ✅ Yes | ✅ Yes | **✅ Compliant** | `moonshotai.kimi-k2.5` (`ap-northeast-1`) | Ultra-long context (2M+ tokens) |
| **NVIDIA** | NVIDIA Nemotron 3 Super 120B | ✅ Yes | ✅ Yes | **✅ Compliant** | `nvidia.nemotron-super-3-120b` (`ap-northeast-1`) | Massive enterprise LLM (Egresses Japan) |
|  | NVIDIA Nemotron Nano 12B v2 VL BF16 | ✅ Yes | ✅ Yes | **✅ Compliant** | `nvidia.nemotron-nano-12b-v2` (`ap-northeast-1`) | Multimodal edge inspection |
|  | NVIDIA Nemotron Nano 9B v2 | ✅ Yes | ✅ Yes | **✅ Compliant** | `nvidia.nemotron-nano-9b-v2` (`ap-northeast-1`) | Lightweight synthetic data generator |
|  | Nemotron Nano 3 30B | ✅ Yes | ✅ Yes | **✅ Compliant** | `nvidia.nemotron-nano-3-30b` (`ap-northeast-1`) | Efficient tool-calling & agent routing |
| **OpenAI** | Daybreak Blue: GPT-5.6 Sol | ✅ Yes | ❌ No | ❌ Non-Compliant | `openai.gpt-daybreak-blue-5.6-sol` (US) | Frontier general intelligence (Egresses Japan) |
|  | Daybreak Red: GPT-5.6 Cyber | ✅ Yes | ❌ No | ❌ Non-Compliant | `openai.gpt-5.6-cyber` (US) | Cybersecurity & code audit (Egresses Japan) |
|  | GPT OSS Safeguard 120B | ✅ Yes | ✅ Yes | **✅ Compliant** | `openai.gpt-oss-safeguard-120b` (`ap-northeast-1`) | Open-weight guardrails & safety evaluation |
|  | GPT OSS Safeguard 20B | ✅ Yes | ✅ Yes | **✅ Compliant** | `openai.gpt-oss-safeguard-20b` (`ap-northeast-1`) | Lightweight safety classifier |
|  | GPT-5.4 | ✅ Yes | ❌ No | ❌ Non-Compliant | `openai.gpt-5.4` (US) | Baseline enterprise GPT on Bedrock |
|  | GPT-5.5 | ✅ Yes | ❌ No | ❌ Non-Compliant | `openai.gpt-5.5` (US) | High-accuracy coding & reasoning (Egresses Japan) |
|  | GPT-5.6 Luna | ✅ Yes | ✅ Yes | ❌ Non-Compliant | `us.openai.gpt-5.6-luna` | Fast reasoning tier (Egresses Japan) |
|  | GPT-5.6 Sol | ✅ Yes | ✅ Yes | ❌ Non-Compliant | `us.openai.gpt-5.6-sol` | Frontier general intelligence (Egresses Japan) |
|  | GPT-5.6 Terra | ✅ Yes | ✅ Yes | ❌ Non-Compliant | `us.openai.gpt-5.6-terra` | Grounded scientific reasoning (Egresses Japan) |
|  | GPT-6 Astra | ✅ Yes | ✅ Yes | ❌ Non-Compliant | `us.openai.gpt-6-astra` | Next-gen omni reasoning (Egresses Japan) |
|  | gpt-oss-120b | ✅ Yes | ✅ Yes | **✅ Compliant** | `openai.gpt-oss-120b-1:0` (`ap-northeast-1`) | Foundation model |
|  | gpt-oss-20b | ✅ Yes | ✅ Yes | **✅ Compliant** | `openai.gpt-oss-20b-1:0` (`ap-northeast-1`) | Foundation model |
| **Qwen** | Qwen3 235B A22B 2507 | ✅ Yes | ✅ Yes | **✅ Compliant** | `qwen.qwen3-235b-a22b-2507-v1:0` (`ap-northeast-1`) | High-parameter open MoE model |
|  | Qwen3 32B | ✅ Yes | ✅ Yes | **✅ Compliant** | `qwen.qwen3-32b-v1:0` (`ap-northeast-1`) | Mid-sized coding and reasoning |
|  | Qwen3 Coder 480B A35B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `qwen.qwen3-coder-480b-a35b-v1:0` (`ap-northeast-1`) | **Top open-weights repository coding agent** |
|  | Qwen3 Coder Next | ✅ Yes | ✅ Yes | ❌ Non-Compliant | `qwen.qwen3-coder-next` (US) | Next-gen code refactoring & synthesis |
|  | Qwen3 Next 80B A3B | ✅ Yes | ✅ Yes | **✅ Compliant** | `qwen.qwen3-next-80b-a3b` (`ap-northeast-1`) | Fast sparse reasoning engine |
|  | Qwen3 VL 235B A22B | ✅ Yes | ✅ Yes | **✅ Compliant** | `qwen.qwen3-vl-235b-a22b` (`ap-northeast-1`) | Multimodal architecture & UI design review |
|  | Qwen3-Coder-30B-A3B-Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `qwen.qwen3-coder-30b-a3b-v1:0` (`ap-northeast-1`) | Lightweight fast coding assistant |
| **Stability AI** | Stable Image Suite (13 tools) | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-*` (US) | Image generation, upscaling, inpainting & editing suite (US-only) |
| **TwelveLabs** | Marengo & Pegasus Suite (3 models) | ❌ No | ✅ Yes | ❌ Non-Compliant | `twelvelabs.*` (US) | Video embeddings & QA comprehension (US-only; egresses Japan) |
| **Writer** | Palmyra Vision 7B | ✅ Yes | ✅ Yes | **✅ Compliant** | `writer.palmyra-vision-7b` (`ap-northeast-1`) | Document & PDF vision parsing |
|  | Palmyra X4 | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.writer.palmyra-x4-v1:0` | Enterprise document generation |
|  | Palmyra X5 | ❌ No | ✅ Yes | ❌ Non-Compliant | `us.writer.palmyra-x5-v1:0` | Complex enterprise writing & formatting |
| **xAI** | Grok 4.3 | ✅ Yes | ❌ No | ❌ Non-Compliant | `xai.grok-4.3` (US) | Real-time reasoning & code synthesis |
|  | Grok 4.6 | ✅ Yes | ✅ Yes | ❌ Non-Compliant | `us.xai.grok-4.6` | Frontier agentic reasoning & math |
| **Z.AI** | GLM 4.7 | ✅ Yes | ✅ Yes | **✅ Compliant** | `zai.glm-4.7` (`ap-northeast-1`) | Multilingual dialogue & reasoning |
|  | GLM 4.7 Flash | ✅ Yes | ✅ Yes | **✅ Compliant** | `zai.glm-4.7-flash` (`ap-northeast-1`) | Ultra-fast lightweight response |
|  | GLM 5 | ✅ Yes | ✅ Yes | **✅ Compliant** | `zai.glm-5` (`ap-northeast-1`) | Next-gen general intelligence |

---

## 3. Key Findings for Gateway Configuration

1. **Flagship Coding Agent Runtime Constraint (Critical Architecture Notice):**
   * **`Claude Sonnet 4.5` and `Claude Sonnet 4.6` are RUNTIME-EXCLUSIVE (`bedrock-runtime`).** Neither model is supported on `bedrock-mantle`.
   * The gateway **must route all developer IDE coding traffic (Cursor, Cline, VS Code) targeting Sonnet 4.5/4.6 to `bedrock-runtime`** using the Converse API or Anthropic-compatible runtime wrapper.
   * `Claude Haiku 4.5` and `Claude Opus 4.7 / 4.8` are supported on **both** `bedrock-mantle` and `bedrock-runtime`.

2. **Amazon Models are 100% Runtime-Exclusive:**
   * In AWS's official endpoint availability specification (`models-endpoint-availability`), **every single Amazon-branded model** (including `Nova 2 Lite`, `Nova Lite`, `Nova Pro`, `Nova Premier`, `Nova Micro`, `Nova Canvas`, `Nova Reel`, `Nova Sonic`, and all `Titan` models) is supported **exclusively on `bedrock-runtime`** (`bedrock-mantle: ❌ No`).
   * No Amazon Nova or Titan models may be routed to `bedrock-mantle`.

3. **True Japan Cross-Region (`jp.`) Profile Scope:**
   * Exactly **6 models** in the entire Bedrock catalog utilize the `jp.` cross-region inference profile (Tokyo <-> Osaka load balancing):
     * `jp.anthropic.claude-sonnet-4-5-20250929-v1:0` (Runtime only)
     * `jp.anthropic.claude-sonnet-4-6` (Runtime only)
     * `jp.anthropic.claude-haiku-4-5-20251001-v1:0` (Mantle & Runtime)
     * `jp.anthropic.claude-opus-4-7` (Mantle & Runtime)
     * `jp.anthropic.claude-opus-4-8` (Mantle & Runtime)
     * `jp.amazon.nova-2-lite-v1:0` (Runtime only)
   * All other 45 Japan-compliant models (Qwen3, Devstral, Gemma 3, DeepSeek V3, Ministral, GPT OSS, Nova Lite) run **strictly In-Region in Tokyo (`ap-northeast-1`)** and MUST be invoked using standard model IDs without any `jp.` prefix.

4. **Osaka Regional & Mantle Constraints:**
   * **`bedrock-mantle` is NOT available in Osaka (`ap-northeast-3`).** It is deployed exclusively in Tokyo (`bedrock-mantle.ap-northeast-1.api.aws`). Gateway proxy configurations must never attempt Mantle calls to Osaka.
   * In-region Osaka hosts only a single model (`amazon.titan-embed-text-v2:0`). All other Osaka compute is accessible strictly via `jp.` cross-region failover.

5. **Mantle-Exclusive Models (Runtime: ❌ No):**
   * Certain frontier and open models run **exclusively on `bedrock-mantle`** and cannot be invoked via `bedrock-runtime`:
     * `Google Gemma 4` (31B, 26B-A4B, E2B)
     * `Claude Mythos 5` and `Claude Mythos Preview`
     * `OpenAI Daybreak Red: GPT-5.6 Cyber` and `Daybreak Blue: GPT-5.6 Sol`
     * `OpenAI GPT-5.4` and `GPT-5.5`
     * `xAI Grok 4.3`

6. **Data Sovereignty Exclusions (Strict Non-Compliance):**
   * `DeepSeek-R1` on Bedrock is hosted **exclusively in the US** (`us.deepseek.r1-v1:0`) and is **Runtime-only**. It cannot be used for strict Japan Geo workloads.
   * **All Meta Llama models** (Llama 3.3 70B, 3.2, 3.1, 4) reside strictly on US/Global endpoints and egress Japan. All Llama models on Bedrock are **Runtime-only**.
   * `Claude Sonnet 5`, `Claude Opus 5`, and `Claude 3.5 Haiku` currently lack Japan Geo profiles and egress Japan.

7. **Expanded In-Region Tokyo Open-Weight & Reasoning Catalog:**
   * `MiniMax M2.5 / M2.1`, `Moonshot Kimi K2.5 / K2 Thinking`, `Z.AI GLM 4.7 / 5`, `Mistral Devstral 2 123B`, and `Qwen3 Coder 480B` are all deployed **directly In-Region in Tokyo (`ap-northeast-1`)** on both `bedrock-mantle` and `bedrock-runtime`.
   * This enables sub-millisecond local network hops and full in-country data sovereignty without overseas routing.

8. **Embeddings & Vector Search Architecture:**
   * `amazon.titan-embed-text-v2:0` (Tokyo & Osaka in-region) and `cohere.embed-multilingual-v3` (Tokyo in-region) are **Runtime-exclusive**.
   * `cohere.embed-english-v3` is also available In-Region in Tokyo (`ap-northeast-1`).
   * The gateway `/v1/embeddings` endpoint must route 100% of vectorization calls to `bedrock-runtime`.

9. **OpenAI & xAI Dual-Plane Availability:**
   * OpenAI (`GPT-6 Astra`, `GPT-5.6 Sol / Terra / Luna`) and xAI (`Grok 4.6`) are available on both `bedrock-mantle` (US-West-2) and `bedrock-runtime` (via `/openai/v1`). Both providers egress Japan and require Tier 3 waivers.
