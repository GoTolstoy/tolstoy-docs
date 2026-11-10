---
name: tolstoy
description: Use when building AI agents that create product images and videos
  for e-commerce brands. Reach for this skill when you need to generate on-model
  photos, lifestyle scenes, product shots, or video content with brand context,
  product catalog integration, and creative direction. Use for agents connected
  via MCP (ChatGPT, Claude, Cursor) or CLI for terminal/script-based workflows.
metadata:
  mintlify-proj: tolstoy
  version: "1.0"
---

# Tolstoy Skill

## Product summary

Tolstoy is an AI content creation platform for e-commerce brands. It provides agents with brand context (logos, colors, fonts), product catalog access, a media library, and tools to generate images, videos, music, and speech. Connect via the Tolstoy MCP server (`https://apilb.gotolstoy.com/mcp/v1/mcp`) for ChatGPT, Claude, Cursor, or Codex, or use the CLI (`@tolstoy-ai/cli`) for terminal and script workflows. All generations spend workspace credits and save to the media library. See [Tolstoy for AI agents](https://developers.gotolstoy.com/agents/overview) for the primary docs.

## When to use

- **Image generation**: Create product photos, on-model try-on shots, lifestyle scenes, detail shots, or mood boards grounded in product and brand context.
- **Video generation**: Render product videos, lifestyle clips, ads, or animated sequences from text, keyframes, or reference images.
- **Media analysis**: Extract pacing, motion, audio properties, or visual details from video/audio files to inform next steps.
- **Brand context lookup**: Read logos, colors, fonts, size guides, and brand profile before generating.
- **Product catalog search**: Find exact products and variants with images to use as references.
- **Media library management**: Search past generated or uploaded media, save fashion models, organize playlists.
- **Planning and research**: Use the Tolstoy Agent to write briefs, plan campaigns, research brand and market.

## Quick reference

### MCP connection setup

| Client | Steps |
| - | - |
| **Claude** | Settings → Connectors → Add custom connector. Name: `Tolstoy`. URL: `https://apilb.gotolstoy.com/mcp/v1/mcp`. Click Connect, sign in. |
| **ChatGPT** | Tolstoy platform → Settings → API, MCP & CLI → MCP → Open Tolstoy in ChatGPT. Sign in and approve. |
| **Cursor** | Add to `~/.cursor/mcp.json`: `{"mcpServers": {"tolstoy": {"url": "https://apilb.gotolstoy.com/mcp/v1/mcp"}}}` |
| **CLI** | `npm install -g @tolstoy-ai/cli` then `tolstoy auth login <api-key>` (create key in Tolstoy platform Settings → API, MCP & CLI). |

### Core MCP tools (28 total)

| Tool | Purpose | Claude | Credits |
| - | - | - | - |
| `get_brand_context` | Read brand profile (logos, colors, fonts, size guide) | Yes | No |
| `search_products` | Find products/variants in catalog with images | Yes | No |
| `search_media` | Search workspace library (images, videos, drafts) | Yes | No |
| `search_fashion_models` | List saved models with reference images | Yes | No |
| `search_templates` | Find content templates (proven recipes) | Yes | No |
| `generate_image` | Create image from brief + references | Allowed only | Yes |
| `generate_video` | Create video from text/image/keyframes/references | Allowed only | Yes |
| `generate_audio` / `text_to_speech` | Create music, sound effects, or speech | Allowed only | Yes |
| `remove_background` | Segment subject onto transparent background | Allowed only | Yes |
| `analyze_media` | Answer questions about video/audio (pacing, motion, tone) | Yes | Yes |
| `save_fashion_model` | Persist generated images as reusable model | Yes | No |
| `run_agent` | Send message to Tolstoy Agent for planning/research | Allowed only | Yes |
| `get_agent_chat` | Read Tolstoy Agent response | Yes | No |
| `list_skills` / `get_skill` | Load creative skills before generation | Yes | No |
| `get_usage` | Check credit balance and period usage | Yes | No |

### CLI command structure

```bash
tolstoy <service> <command> [flags]
tolstoy products search-products --json
tolstoy brand get-brand-context --app-url example.com
tolstoy ai-widgets list-projects
tolstoy models get-models
```

Key services: `products`, `brand`, `media`, `models`, `ai-widgets`, `billing`, `folders`, `meta-ads`.

### Image generation modes

| Mode | Use when | References | Example |
| - | - | - | - |
| `text` | Pure text-to-image, no product/brand reference | null | Abstract background, mood board |
| `image` | Product, model, or style reference exists | Required | On-model try-on, product in scene, lifestyle |

**Rule**: When ANY product photo, brand image, model photo, or style reference exists in context, use `mode='image'`. Even "create a lifestyle scene around this product" requires the product image as reference.

### Video generation modes

| Mode | Use when | Input | Models |
| - | - | - | - |
| `text` | Prompt only, no source media | prompt | Most models |
| `image` | Animate one image | imageUrl | Kling, Seedance, Gemini, Flux, Runway |
| `keyframe` | Interpolate between first and last frame | firstFrameUrl, lastFrameUrl | Kling, Seedance, Gemini, Flux, Luma, Minimax |
| `reference` | Compose multiple images/audio/elements | referenceImages, referenceAudioUrls, elements | Seedance 2.5, Kling V4, Gemini Omni, WAN 3.0 |

## Decision guidance

### When to use `generate_image` vs `generate_video`

| Scenario | Use | Why |
| - | - | - |
| Show product on model in one pose | `generate_image` | Faster, cheaper, precise product detail |
| Show product being worn/used in motion | `generate_video` | Motion conveys fit, action, lifestyle |
| Create a still lifestyle scene | `generate_image` | Composition and detail matter more than time |
| Show product transformation or sequence | `generate_video` | Multiple frames tell the story |

### When to use `text` vs `image` mode for images

| Scenario | Mode | Why |
| - | - | - |
| Product photo exists, user wants it in a new scene | `image` | Reference grounds the product identity and color |
| No product photo, user wants abstract background | `text` | No reference to preserve |
| Product photo exists but user wants "completely different" framing | Regenerate from source | Don't chain edits; rebuild from original images |
| User wants to remove a structural element (seam, floor line) | Regenerate from source | `image` mode cannot reliably remove structure |

### When to use Tolstoy Agent vs direct generation

| Task | Use | Why |
| - | - | - |
| Generate one product image now | `generate_image` | Direct, immediate, costs one generation |
| Plan a content campaign, write briefs, research brand | `run_agent` | Agent plans, writes, researches; you execute |
| Analyze competitor video for pacing/structure | `analyze_media` | Multimodal model extracts temporal details |
| Quick one-off person without reusable library | `generate_image` mode=text | Faster than saving a model |
| Recurring model for multiple products | `save_fashion_model` then `generate_image` | Reuse identity across shoots |

## Workflow

### Typical image generation task

1. **Understand the request**: What product, what scene, what brand context matters?
2. **Check brand and catalog**: Call `get_brand_context` and `search_products` to find exact product ID, images, brand colors/fonts.
3. **Find or request references**: If product photo exists, use `mode='image'`. If model/style reference needed, search `search_fashion_models` or ask user to provide.
4. **Load creative skill**: Call `list_skills`, find matching skill (e.g., "on-model photoshoot"), call `get_skill` to load guidance.
5. **Generate**: Call `generate_image` with mode, prompt (following skill guidance), references, product ID, aspect ratio, resolution.
6. **Check result**: Review generated image. If small delta needed (color, mood, light), regenerate with updated prompt. If structural change needed (layout, framing), regenerate from original sources.
7. **Save and tag**: Image auto-saves to library. Call `save_fashion_model` if the person should be reusable.

### Typical video generation task

1. **Understand the request**: What motion, what duration, what style?
2. **Pick mode**: Text (prompt only), image (animate one still), keyframe (first→last interpolation), or reference (multi-image composition).
3. **Load creative skill**: Call `list_skills`, find matching skill (e.g., "product video ad"), call `get_skill`.
4. **Select model**: Check model capabilities in reference docs. Match mode to model (not every model runs every mode).
5. **Prepare references**: If mode=image/keyframe/reference, gather URLs. For Seedance, structure audio in channels: (music), <sfx>, {dialogue}, 【subtitles】.
6. **Generate**: Call `generate_video` with mode, prompt, references, duration, aspect ratio, model.
7. **Analyze if needed**: Call `analyze_media` to extract pacing, motion, audio quality, visual beats.
8. **Iterate**: Regenerate with updated prompt or different model. Do not chain edits; rebuild from original sources.

### Planning and research workflow

1. **Ask Tolstoy Agent**: Call `run_agent` with your question (campaign plan, brief, market research, brand analysis).
2. **Wait for response**: Agent returns when turn starts. Call `get_agent_chat` to read the answer.
3. **Execute**: Use the brief/plan to guide `generate_image` or `generate_video` calls.

## Common gotchas

- **Chaining edits compounds artifacts**: Every generation reconstructs the frame. Repeated edits on the same image carry forward rendering artifacts in texture, color, detail, and identity. Regenerate from original source photos instead.
- **Reference mode requires matching model**: Not every model runs every mode. `kling_v3_*` is text/image/keyframe only (no reference mode). `kling_v4`, `seedance_2.5`, `gemini_omni_flash`, `alibaba_wan_3_0` support reference mode. Check model capabilities before calling.
- **Small details don't always carry**: Logos, fine print, subtle colors, tiny product details may simplify in output. Reference images are visual evidence, not pixel contracts. Use words to reinforce what pixels cannot carry.
- **Aspect ratio mismatch on video**: Seedance 2.x in image mode returns a clip matching the input image's shape, not the requested aspectRatio. Resize the anchor image before generating if shapes differ.
- **Audio-driven models require audio URL**: `veed_fabric_1_0` and `bytedance_omnihuman_v2` require `audioUrl` in image mode. Other models reject it.
- **Seedance audio quality degrades past ~10s**: Cap audio-critical clips at 10s regardless of tier ceiling.
- **Intimate apparel framing**: When references show underwear, lingerie, swimwear, or sleepwear, apply commercial framing on the first attempt.
- **Identity anchor matters**: The first reference image is read as the person. A person in a product photo is context, never a face source. Pair like with like.
- **Conversational references fail**: Prompts must be self-contained. Do not reference prior turns ("like we discussed") or conversation context. Repeat the full direction in each prompt.
- **Claude lists tools only for allowed workspaces**: Media generation tools (`generate_image`, `generate_video`, `run_agent`) appear only for workspaces with generation access. Ask support@gotolstoy.com for access.
- **Tool list is cached**: If a tool in the reference is missing in your client, refresh or reconnect the Tolstoy connector.
- **Credits are spent immediately**: Generations and Agent turns spend credits when called, not when results return. Check `get_usage` before expensive operations.

## Verification checklist

Before submitting generated media:

- [ ] Brand context loaded: logos, colors, fonts match brand profile.
- [ ] Product identity preserved: product shape, color, labels visible and accurate (if reference image used).
- [ ] Person identity consistent: face, skin tone, body, age match source model (if model reference used).
- [ ] Prompt self-contained: no conversation refs, no prior-turn context.
- [ ] Mode matches references: `mode='image'` when references exist; `mode='text'` only when no references.
- [ ] Model supports mode: checked model capabilities for text/image/keyframe/reference.
- [ ] Aspect ratio correct: matches deliverable requirement (or null to use model default).
- [ ] Resolution appropriate: 2K default; 4K only for hero/print-quality.
- [ ] No chained edits: if multiple corrections needed, regenerated from original sources, not from prior output.
- [ ] Credit balance sufficient: checked `get_usage` before generation.
- [ ] Media saved to library: generation auto-saves; verify in `search_media` if needed.
- [ ] Reusable model saved: if person should be reused, called `save_fashion_model`.

## Resources

- **Full page navigation**: [https://developers.gotolstoy.com/llms.txt](https://developers.gotolstoy.com/llms.txt)
- **Connect your agent**: [https://developers.gotolstoy.com/agents/connect](https://developers.gotolstoy.com/agents/connect)
- **All MCP tools reference**: [https://developers.gotolstoy.com/agents/reference/tools](https://developers.gotolstoy.com/agents/reference/tools)
- **Create media (image/video/audio)**: [https://developers.gotolstoy.com/agents/reference/create-media](https://developers.gotolstoy.com/agents/reference/create-media)

---

> For additional documentation and navigation, see: https://developers.gotolstoy.com/llms.txt