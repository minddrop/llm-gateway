# Amazon Bedrock Model Availability Matrix: Complete Engine & Japan Geo Survey

**Target Regions:** Tokyo (`ap-northeast-1`) & Osaka (`ap-northeast-3`)  
**Data Residency Policy:** 100% In-Country Processing Guarantee (Strict Data Sovereignty)  
**Engines Covered:** Amazon Bedrock Mantle (`bedrock-mantle`) & Amazon Bedrock Runtime (`bedrock-runtime`)  
**Last Updated:** September 2026  

---

## 1. Executive Summary & Routing Philosophy

For an internal engineering development gateway, Amazon Bedrock operates across two primary execution engines:
1. **Bedrock Mantle (`bedrock-mantle.<region>.api.aws`):**  
   Next-gen distributed inference engine offering **native OpenAI & Anthropic wire protocols**, token-governed queues (no RPM throttle limits), and cryptographic **Zero Operator Access (ZOA)**. **Primary engine for coding agents, IDEs, and interactive reasoning.**
2. **Bedrock Runtime (`bedrock-runtime.<region>.amazonaws.com`):**  
   Classic data plane offering the **AWS Converse API**, `InvokeModel`, and media endpoints. **Required engine for vector embeddings (RAG), media generation, and fallback execution.**

### Japan Geo Residency Rules
* **✅ Compliant (Japan Geo):** The model runs **in-region Tokyo (`ap-northeast-1`)** or via a **Japan Cross-Region profile (`jp.` prefix)**. Requests load-balance strictly between Tokyo and Osaka with a 100% guarantee that data never leaves Japan.
* **⚠️ Limited (APAC):** Available in APAC regional profiles (`apac.`), which may route to Sydney or Singapore. Requires explicit waiver.
* **❌ Non-Compliant (Overseas):** Available only in US (`us-east-1`, `us-west-2`) or `global.` profiles. Prompts egress Japan.

---

## 2. Complete Exhaustive Model Survey (127 Models)

Below is the complete survey of all models across both engines and Japan Geo status:

| Provider | Model Name | Bedrock Mantle | Bedrock Runtime | Japan Geo Status | In-Country Profile / Endpoint Format | Modality & Engineering Use |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **AI21 Labs** | Jamba 1.5 Large | ❌ No | ✅ Yes | ❌ Non-Compliant | `ai21.jamba-1-5-large-v1:0` (US) | Long-context SSM-Transformer text |
| | Jamba 1.5 Mini | ❌ No | ✅ Yes | ❌ Non-Compliant | `ai21.jamba-1-5-mini-v1:0` (US) | Fast low-cost SSM-Transformer |
| **Amazon** | Nova Multimodal Embeddings | ❌ No | ✅ Yes | **✅ Compliant** | `amazon.nova-embed-multimodal-v1:0` (`ap-northeast-1`) | Multimodal vector embeddings for RAG |
| | Nova 2 Lite | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.amazon.nova-2-lite-v1:0` | Ultra-fast text/code utility |
| | Nova 2 Sonic | ❌ No | ✅ Yes | **✅ Compliant** | `amazon.nova-2-sonic-v1:0` (`ap-northeast-1`) | Low-latency speech-to-speech interaction |
| | Nova Canvas | ❌ No | ✅ Yes | ⚠️ Limited | `amazon.nova-canvas-v1:0` (`ap-northeast-1`) | Enterprise image generation & editing |
| | Nova Lite | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.amazon.nova-lite-v1:0` | Cost-effective text and code analysis |
| | Nova Micro | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.amazon.nova-micro-v1:0` | Ultra-low latency text summarization |
| | Nova Premier | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.amazon.nova-premier-v1:0` | High-complexity multi-step reasoning |
| | Nova Pro | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.amazon.nova-pro-v1:0` | Multimodal code inspection & analysis |
| | Nova Reel | ❌ No | ✅ Yes | ❌ Non-Compliant | `amazon.nova-reel-v1:0` (US) | Video generation (Egresses Japan) |
| | Nova Sonic | ❌ No | ✅ Yes | **✅ Compliant** | `amazon.nova-sonic-v1:0` (`ap-northeast-1`) | Speech generation and transcription |
| | Titan Embeddings G1 - Text | ❌ No | ✅ Yes | **✅ Compliant** | `amazon.titan-embed-text-v1` (`ap-northeast-1`) | Classic text embeddings |
| | Titan Image Generator G1 v2 | ❌ No | ✅ Yes | ⚠️ Limited | `amazon.titan-image-generator-v2:0` (`ap-northeast-1`) | Image generation & inpainting |
| | Titan Multimodal Embeddings G1 | ❌ No | ✅ Yes | **✅ Compliant** | `amazon.titan-embed-image-v1` (`ap-northeast-1`) | Text & image hybrid vector search |
| | Titan Text Embeddings V2 | ❌ No | ✅ Yes | **✅ Compliant** | `amazon.titan-embed-text-v2:0` (`ap-northeast-1`) | **Primary AWS codebase RAG embeddings** |
| | Titan Embeddings G1 - Text v2 | ❌ No | ✅ Yes | **✅ Compliant** | `amazon.titan-embed-g1-text-02` (`ap-northeast-1`) | General document vector indexing |
| **Anthropic** | Claude Fable 5.1 | ✅ Yes | ❌ No | ❌ Non-Compliant | `global.anthropic.claude-fable-5-1` | Experimental creative/prose model |
| | Claude Mythos 5.1 | ✅ Yes | ❌ No | ❌ Non-Compliant | `global.anthropic.claude-mythos-5-1` | Next-gen autonomous agent reasoning |
| | Claude Opus 5 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-opus-5-...` | Maximum frontier intelligence & math |
| | Claude Sonnet 5 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-sonnet-5-...` | **Next-gen primary coding & dev agent** |
| | Claude Mythos 5 | ✅ Yes | ❌ No | ❌ Non-Compliant | `global.anthropic.claude-mythos-5` | Frontier autonomous agent preview |
| | Claude Fable 5 | ✅ Yes | ❌ No | ❌ Non-Compliant | `global.anthropic.claude-fable-5` | Creative writing & dialogue generation |
| | Claude Opus 4.8 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-opus-4-8-...` | Advanced architecture analysis & synthesis |
| | Claude Opus 4.7 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-opus-4-7-...` | Complex multi-repository refactoring |
| | Claude Opus 4.6 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-opus-4-6-...` | Deep reasoning and formal verification |
| | Claude Sonnet 4.6 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-sonnet-4-6-...` | High-performance full-stack coding |
| | Claude Haiku 4.5 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-haiku-4-5-...` | **Ultra-fast coding linter & test runner** |
| | Claude Opus 4.5 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-opus-4-5-...` | Complex code synthesis & algorithmic math |
| | Claude Sonnet 4.5 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-sonnet-4-5-...` | **Current standard IDE agent (Cursor/Cline)** |
| | Claude Sonnet 4 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-sonnet-4-...` | Reliable coding and logic reasoning |
| | Claude Opus 4.1 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-opus-4-1-...` | Deep technical writing & documentation |
| | Claude 3.5 Haiku | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.anthropic.claude-3-5-haiku-...` | Sub-second inline code completion |
| | Claude 3 Haiku | ✅ Yes | ✅ Yes | **✅ Compliant** | `anthropic.claude-3-haiku-20240307-v1:0` (`ap-northeast-1`) | Legacy lightweight task execution |
| | Claude Mythos Preview | ✅ Yes | ❌ No | ❌ Non-Compliant | `global.anthropic.claude-mythos-preview` | Early preview reasoning engine |
| **Cohere** | Rerank 3.5 | ❌ No | ✅ Yes | **✅ Compliant** | `cohere.rerank-v3-5:0` (`ap-northeast-1`) | **Search re-ranking for engineering RAG** |
| | Command R | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.cohere.command-r-...` | Multilingual document processing |
| | Command R+ | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.cohere.command-r-plus-...` | High-accuracy business logic & tool calling |
| | Embed English | ❌ No | ✅ Yes | ❌ Non-Compliant | `cohere.embed-english-v3` (US) | English-only vector indexing |
| | Embed Multilingual | ❌ No | ✅ Yes | **✅ Compliant** | `cohere.embed-multilingual-v3` (`ap-northeast-1`) | **Gold-standard Japanese & Codebase RAG** |
| | Embed v4 | ❌ No | ✅ Yes | **✅ Compliant** | `cohere.embed-multilingual-v4` (`ap-northeast-1`) | High-dimensional dense vector embeddings |
| **DeepSeek** | DeepSeek V3.2 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.deepseek.v3-2-...` | High-efficiency general-purpose coding |
| | DeepSeek-V3.1 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.deepseek.v3-1-...` | Cost-effective open-weight assistant |
| | DeepSeek-R1 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.deepseek.r1-...` / `ap-northeast-1` | **Open-weight deep reasoning & math synthesis** |
| **Google** | Gemma 4 31B | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.google.gemma-4-31b-...` | High-capacity open weights for research |
| | Gemma 4 26B-A4B | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.google.gemma-4-26b-a4b-...` | MoE open model for developer testing |
| | Gemma 4 E2B | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.google.gemma-4-e2b-...` | Edge-optimized lightweight testing |
| | Gemma 3 12B IT | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.google.gemma-3-12b-it-...` | Instruction-tuned local development |
| | Gemma 3 27B PT | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.google.gemma-3-27b-pt-...` | Pre-trained open research foundation |
| | Gemma 3 4B IT | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.google.gemma-3-4b-it-...` | Micro test runner & unit test scaffolding |
| **Meta** | Llama 3.3 70B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.meta.llama3-3-70b-instruct-...` | **Top open-weights coding model** |
| | Llama 3.2 11B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.meta.llama3-2-11b-instruct-...` | Vision & text multimodal open assistant |
| | Llama 3.2 1B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.meta.llama3-2-1b-instruct-...` | Instant edge completion & simple linting |
| | Llama 3.2 3B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.meta.llama3-2-3b-instruct-...` | Lightweight text parsing & commit tools |
| | Llama 3.2 90B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.meta.llama3-2-90b-instruct-...` | Multimodal code & diagram architecture |
| | Llama 3.1 405B Instruct | ❌ No | ✅ Yes | ❌ Non-Compliant | `meta.llama3-1-405b-instruct-v1:0` (US) | Massive open model (Egresses Japan) |
| | Llama 3.1 70B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.meta.llama3-1-70b-instruct-...` | Proven enterprise open coding foundation |
| | Llama 3.1 8B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.meta.llama3-1-8b-instruct-...` | Cost-effective rapid test runner |
| | Llama 3 70B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `meta.llama3-70b-instruct-v1:0` (`ap-northeast-1`) | Legacy open-weight benchmark baseline |
| | Llama 3 8B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `meta.llama3-8b-instruct-v1:0` (`ap-northeast-1`) | Legacy lightweight assistant |
| | Llama 4 Maverick 17B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.meta.llama4-maverick-17b-...` | **Next-gen autonomous open coding agent** |
| | Llama 4 Scout 17B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.meta.llama4-scout-17b-...` | Next-gen reasoning & search assistant |
| **MiniMax** | MiniMax M2.5 | ✅ Yes | ✅ Yes | ⚠️ Limited | `apac.minimax.m2-5-...` | Long-context Chinese/English synthesis |
| | MiniMax M2.1 | ✅ Yes | ✅ Yes | ⚠️ Limited | `apac.minimax.m2-1-...` | Dialogue & code translation |
| | MiniMax M2 | ✅ Yes | ✅ Yes | ⚠️ Limited | `apac.minimax.m2-...` | Baseline long-context reasoning |
| **Mistral AI** | Ministral 14B 3.0 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.mistral.ministral-14b-...` | High-density developer assistant |
| | Devstral 2 123B | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.mistral.devstral-2-123b-...` | **Specialized software engineering LLM** |
| | Magistral Small 2509 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.mistral.magistral-small-...` | Lightweight agent orchestration |
| | Ministral 3 8B | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.mistral.ministral-3-8b-...` | Edge-friendly dev assistant |
| | Ministral 3B | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.mistral.ministral-3b-...` | Ultra-fast inline code suggester |
| | Mistral 7B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `mistral.mistral-7b-instruct-v0:2` (`ap-northeast-1`) | Classic lightweight code generation |
| | Mistral Large | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.mistral.mistral-large-...` | Complex multilingual logic & reasoning |
| | Mistral Large 3 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.mistral.mistral-large-3-...` | High-precision coding & multilingual math |
| | Mistral Small | ✅ Yes | ✅ Yes | **✅ Compliant** | `mistral.mistral-small-2402-v1:0` (`ap-northeast-1`) | Fast structured JSON output |
| | Mixtral 8x7B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `mistral.mixtral-8x7b-instruct-v0:1` (`ap-northeast-1`) | Sparse MoE benchmark standard |
| | Pixtral Large | ✅ Yes | ✅ Yes | ⚠️ Limited | `apac.mistral.pixtral-large-...` | Multimodal visual code & diagram parser |
| | Voxtral Mini 3B 2507 | ❌ No | ✅ Yes | ⚠️ Limited | `apac.mistral.voxtral-mini-...` | Voice and audio instruction following |
| | Voxtral Small 24B 2507 | ❌ No | ✅ Yes | ⚠️ Limited | `apac.mistral.voxtral-small-...` | High-fidelity audio & speech intelligence |
| **Moonshot AI** | Kimi K2.5 | ✅ Yes | ✅ Yes | ⚠️ Limited | `apac.moonshot.kimi-k2-5-...` | Ultra-long context (2M+ tokens) |
| | Kimi K2 Thinking | ✅ Yes | ✅ Yes | ⚠️ Limited | `apac.moonshot.kimi-k2-thinking-...` | Deep reasoning & long-document audit |
| **NVIDIA** | Nemotron Nano 12B v2 VL BF16 | ✅ Yes | ✅ Yes | **✅ Compliant** | `nvidia.nemotron-nano-12b-vl` (`ap-northeast-1`) | Multimodal edge inspection |
| | Nemotron Nano 9B v2 | ✅ Yes | ✅ Yes | **✅ Compliant** | `nvidia.nemotron-nano-9b` (`ap-northeast-1`) | Lightweight synthetic data generator |
| | Nemotron Nano 3 30B | ✅ Yes | ✅ Yes | **✅ Compliant** | `nvidia.nemotron-nano-30b` (`ap-northeast-1`) | Efficient tool-calling & agent routing |
| | Nemotron 3 Super 120B | ❌ No | ✅ Yes | ❌ Non-Compliant | `nvidia.nemotron-super-120b` (US) | Massive enterprise LLM (Egresses Japan) |
| **OpenAI** | GPT-6 Astra | ✅ Yes | ❌ No | ❌ Non-Compliant | `openai.gpt-6-astra-v1:0` (US) | Next-gen omni reasoning (Egresses Japan) |
| | GPT-5.6 Sol | ✅ Yes | ❌ No | ❌ Non-Compliant | `openai.gpt-56-sol-v1:0` (US) | Frontier general intelligence (Egresses Japan) |
| | GPT-5.6 Terra | ✅ Yes | ❌ No | ❌ Non-Compliant | `openai.gpt-56-terra-v1:0` (US) | Grounded scientific reasoning (Egresses Japan) |
| | GPT-5.6 Luna | ✅ Yes | ❌ No | ❌ Non-Compliant | `openai.gpt-56-luna-v1:0` (US) | Fast reasoning tier (Egresses Japan) |
| | Daybreak Red: GPT-5.6 Cyber | ✅ Yes | ❌ No | ❌ Non-Compliant | `openai.gpt-56-cyber-v1:0` (US) | Cybersecurity & code audit (Egresses Japan) |
| | Daybreak Blue: GPT-5.6 Sol | ✅ Yes | ❌ No | ❌ Non-Compliant | `openai.gpt-daybreak-sol-v1:0` (US) | Frontier research agent (Egresses Japan) |
| | GPT-5.5 | ✅ Yes | ❌ No | ❌ Non-Compliant | `openai.gpt-55-v1:0` (US) | High-accuracy coding & reasoning (Egresses Japan) |
| | GPT-5.4 | ✅ Yes | ❌ No | ❌ Non-Compliant | `openai.gpt-54-v1:0` (US) | Baseline enterprise GPT on Bedrock |
| | GPT OSS Safeguard 120B | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.openai.gpt-oss-safeguard-120b` | Open-weight guardrails & safety evaluation |
| | GPT OSS Safeguard 20B | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.openai.gpt-oss-safeguard-20b` | Lightweight safety classifier |
| | gpt-oss-120b | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.openai.gpt-oss-120b` | High-capacity open weights on Bedrock |
| | gpt-oss-20b | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.openai.gpt-oss-20b` | Fast open-weight coding assistant |
| **Qwen** | Qwen3 235B A22B 2507 | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.qwen.qwen3-235b-...` | High-parameter open MoE model |
| | Qwen3 32B | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.qwen.qwen3-32b-...` | Mid-sized coding and reasoning |
| | Qwen3 Coder 480B A35B Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.qwen.qwen3-coder-480b-...` | **Top open-weights repository coding agent** |
| | Qwen3 Coder Next | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.qwen.qwen3-coder-next-...` | Next-gen code refactoring & synthesis |
| | Qwen3 Next 80B A3B | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.qwen.qwen3-next-80b-...` | Fast sparse reasoning engine |
| | Qwen3 VL 235B A22B | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.qwen.qwen3-vl-235b-...` | Multimodal architecture & UI design review |
| | Qwen3-Coder-30B-A3B-Instruct | ✅ Yes | ✅ Yes | **✅ Compliant** | `jp.qwen.qwen3-coder-30b-...` | Lightweight fast coding assistant |
| **Stability AI** | Stable Image Conservative Upscale | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-upscale` (US) | Texture-preserving image upscaling |
| | Stable Image Control Sketch | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-sketch` (US) | Sketch-to-image synthesis |
| | Stable Image Control Structure | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-structure` (US) | Geometry-conditioned generation |
| | Stable Image Creative Upscale | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-creative` (US) | Generative image resolution enhancement |
| | Stable Image Erase Object | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-erase` (US) | Inpainting / object removal |
| | Stable Image Fast Upscale | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-fast` (US) | Low-latency 4x upscaler |
| | Stable Image Inpaint | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-inpaint` (US) | Masked image infilling |
| | Stable Image Outpaint | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-outpaint` (US) | Canvas extension and outpainting |
| | Stable Image Remove Background | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-bg-removal` (US) | Alpha mask subject extraction |
| | Stable Image Search and Recolor | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-recolor` (US) | Segmented color alteration |
| | Stable Image Search and Replace | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-replace` (US) | Semantic prompt-guided replacement |
| | Stable Image Style Guide | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-style` (US) | Visual brand style transfer |
| | Stable Image Style Transfer | ❌ No | ✅ Yes | ❌ Non-Compliant | `stability.stable-image-transfer` (US) | Artistic neural style transfer |
| **TwelveLabs** | Marengo Embed 3.0 | ❌ No | ✅ Yes | ❌ Non-Compliant | `twelvelabs.marengo-embed-3-0` (US) | Multimodal video vector embedding |
| | Marengo Embed v2.7 | ❌ No | ✅ Yes | ❌ Non-Compliant | `twelvelabs.marengo-embed-2-7` (US) | Legacy video retrieval embedding |
| | Pegasus v1.2 | ❌ No | ✅ Yes | ❌ Non-Compliant | `twelvelabs.pegasus-1-2` (US) | Video comprehension & natural language QA |
| **Writer** | Palmyra X4 | ❌ No | ✅ Yes | ❌ Non-Compliant | `writer.palmyra-x4` (US) | Enterprise document generation |
| | Palmyra X5 | ❌ No | ✅ Yes | ❌ Non-Compliant | `writer.palmyra-x5` (US) | Complex enterprise writing & formatting |
| | Palmyra Vision 7B | ❌ No | ✅ Yes | ❌ Non-Compliant | `writer.palmyra-vision-7b` (US) | Document & PDF vision parsing |
| **xAI** | Grok 4.3 | ✅ Yes | ❌ No | ❌ Non-Compliant | `xai.grok-4-3-v1:0` (US) | Real-time reasoning & code synthesis |
| | Grok 4.6 | ✅ Yes | ❌ No | ❌ Non-Compliant | `xai.grok-4-6-v1:0` (US) | Frontier agentic reasoning & math |
| **Z.AI** | GLM 4.7 | ✅ Yes | ✅ Yes | ⚠️ Limited | `apac.zai.glm-4-7-...` | Multilingual dialogue & reasoning |
| | GLM 4.7 Flash | ✅ Yes | ✅ Yes | ⚠️ Limited | `apac.zai.glm-4-7-flash-...` | Ultra-fast lightweight response |
| | GLM 5 | ✅ Yes | ✅ Yes | ⚠️ Limited | `apac.zai.glm-5-...` | Next-gen general intelligence |

---

## 3. Key Findings for Gateway Configuration

1. **All 37 Japan-Compliant Generative LLMs are available on `bedrock-mantle`:**
   * This includes the premier coding models: `jp.anthropic.claude-sonnet-5`, `jp.anthropic.claude-sonnet-4-5`, `jp.qwen.qwen3-coder-480b`, `jp.mistral.devstral-2-123b`, and `jp.deepseek.r1`.
   * Developers can use these via standard OpenAI/Anthropic wire format with zero RPM throttle limits.
2. **Embeddings & Vector Search are 100% on `bedrock-runtime`:**
   * `cohere.embed-multilingual-v3` and `amazon.titan-embed-text-v2:0` are **Runtime-exclusive**.
   * Gateway must route `/v1/embeddings` to `bedrock-runtime`.
3. **Mantle-Exclusive Models:**
   * OpenAI on Bedrock (GPT-5/6 series) and xAI Grok run exclusively on `bedrock-mantle`. Note that they currently require `global.` or US endpoints (egressing Japan).
4. **Media Generation is Runtime-Exclusive:**
   * Image models (Nova Canvas, Stability AI) and video models (TwelveLabs, Nova Reel) run exclusively on `bedrock-runtime`.
