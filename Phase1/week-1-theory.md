# Week 1-2: Diffusion Models & AI Image Generation Fundamentals

**Resource:** [How do AI Images Work - Welch Labs](https://youtu.be/iv-5mZ_9CPY)

---

## CLIP - How It Works

CLIP is fundamentally a **bridge between images and text** – it learns to understand what text descriptions match what images.

### Core Mechanics

- **Two Encoders:** CLIP has an Image Encoder and a Text Encoder that transform images and text into vectors (numerical representations)
- **Shared Vector Space:** Both image vectors and text vectors exist in the same high-dimensional space (512 dimensions). They're positioned as points in this space.
- **Learning Through Training Data:** CLIP is trained on millions of image-text pairs from the internet. During training, it learns to position matching images and text close together through iterative adjustment.
- **Similarity = Proximity:** The closer two vectors are in the space, the more similar their meaning. Distance is measured using cosine similarity.

### Semantic Relationships

Through training on many examples, directional relationships emerge:
- The delta (difference vector) between "photo of man" and "photo of man wearing hat" gives you a direction
- If you "travel" in that direction, you move close to where the "hat" vector is positioned
- This way, text and images are trained to form semantic relationships

### Why It Matters for Image Generation

The main use of CLIP in image generation is **conditioning** – the text embedding vector steers the diffusion process in the direction we want.

---

## Image Generation - How It Works

### Training Process

1. Take an image and incrementally add noise to it until it results in pure noise
2. Without conditioning, this results in unpredictable random images
3. With CLIP vector conditioning, the diffusion process is steered toward desired outputs

---

## Diffusion Algorithms

### DDPM Models (Denoising Diffusion Probabilistic Models)

**The Old Way:**
- Give the diffusion model the task: "Given noise at step N, predict what the image should be at step N-1"
- Result: Low quality

**The New Way (DDPM):**
- Ask the transformer to predict Step 0 (the final image) at each step
- Result: More efficient, less random learning, refined final steps

**How It Works in Practice:**

In a 20-step example with a linear schedule:

```
Step 1:
  Input: Pure noise
  Model predicts: Step 0 (final image)
  Blend: 5% predicted image + 95% noise
  Add random noise back
  Output: Slightly denoised image

Step 2:
  Input: Slightly denoised image from Step 1
  Model predicts: Step 0 (refined prediction)
  Blend: 10% predicted image + 90% noise
  Add random noise back
  Output: More denoised image

Step 3:
  Input: More denoised from Step 2
  Model predicts: Step 0
  Blend: 15% predicted + 85% noise
  Add random noise back
  Output: Even cleaner

... repeat 20 times total

Final Step (Step 20):
  Input: Nearly clean image
  Model predicts: Step 0
  Blend: 100% predicted image + 0% noise
  Output: Final clean image
```

**Why Random Noise is Added Each Step:**
- Without noise: Model converges to the median/average of training data (generic)
- With noise: Random perturbations force the model to explore different solutions
- Result: More diverse, less generic outputs

**Trade-off:** DDPM requires significant compute power

---

### DDIM Models (Denoising Diffusion Implicit Models)

**Key Difference:** Does NOT add random noise at each step

**Advantages:**
- Much faster generation
- Works with fewer steps (10-50 vs 100+)
- Deterministic (same seed = same output)

**Limitation:**
- Without conditioning, generates generic median images
- In practice, DDIM is always used WITH conditioning (text prompts via CLIP)
- The prompt steers the denoising away from the median and produces images matching the description

**Why This Works:**
- Only source of randomness is the initial noise seed
- Different seed = different starting point = different final image
- Same seed + same prompt = identical output (reproducible)

---

### Other Sampling Algorithms

- **DPM++:** Generally best quality/speed balance (most popular modern choice)
- **Euler:** Fast, decent quality, good for style transfer
- **Heun:** Slower, slightly better quality
- **Others:** Niche use cases

**Practical Advice:**
- Don't overthink sampler choice yet – start with DPM++
- Bigger quality gains come from: better models, better prompts, more steps, guidance tuning
- Sampler choice is ~5% of the equation

---

## Guidance & Classifier-Free Guidance (CFG)

**Guidance Scale:** Controls how strongly the prompt influences image generation

### How It Works

The model makes **two predictions at each step:**

1. **Unconditional prediction:** "Denoise this noise (no prompt)"
2. **Conditional prediction:** "Denoise this noise toward [CLIP prompt vector]"

Then blends them:

```
Final prediction = Unconditional + (Guidance Scale) × (Conditional - Unconditional)
```

### Guidance Scale Values

- **1-3 (Low):** Creative, but mostly ignores prompt
- **7-12 (Medium):** Good balance (typical use, recommended starting point)
- **15+ (High):** Overly literal, can produce artifacts and unnatural results

### Guidance Scale by Model Type

**Large, General Models (FLUX, SDXL):**
- Can handle high guidance (12-20)
- Rich semantic space handles strong prompts well

**Specialized Models (Fine-tuned, LoRAs):**
- Work better with medium guidance (5-10)
- High guidance can cause artifacts (overfits to training style)
- Smaller semantic space = strong guidance amplifies model bias

### Best Practice

- Start at guidance scale 7.5 (safe default)
- Adjust based on results
- No hard rules – experiment per model

### Why CFG is Used

- Makes the prompt actually matter
- No need for separate classifier model
- Simple but effective mechanism

---

## Noise Schedules

Different diffusion models use different noise schedules – meaning the **percentage of predicted image blended in at each step varies**.

### Schedule Types

- **Linear:** Increases evenly (5% → 10% → 15% → 20%)
- **Quadratic:** Increases slower at first, faster later
- **Cosine:** Smooth, curved progression (often used in modern models)
- **Exponential:** Increases exponentially

### Why It Matters

Different schedules affect:
- Image quality
- Speed vs quality tradeoff
- Detail preservation

The noise schedule is usually **hardcoded into the model** – you don't typically choose it.

---

## Negative Prompts (2026 Status)

As of 2026, negative prompts are changing:

- **Older models (Stable Diffusion, SDXL):** Still use and support negative prompts effectively
- **Newer models (FLUX, Nano Banana Pro):** Less reliant or don't support them
- **Trend:** Moving toward natural language ("no X" in positive prompt instead of separate negative prompt)

### Still Useful For

- Quality issues: "blurry, pixelated, low resolution, grainy, distorted"
- Preventing artifacts: "noise, compression artifacts, jpeg artifacts, glitches"

### Best Practice

- For modern models: Include unwanted elements in positive prompt with "no" (e.g., "Business man... No text or typography")
- For older models: Still use separate negative prompts (50-150 words is optimal)

---

## Key Takeaways

✅ CLIP encodes image-text relationships in a shared vector space  
✅ Diffusion works by iteratively denoising noise toward a target image  
✅ DDPM adds noise each step to prevent averaging to the median  
✅ DDIM is faster by not adding noise, but relies on CLIP conditioning  
✅ Guidance Scale controls how strongly prompts influence generation (7.5 is typical)  
✅ Noise schedules vary by model but are built-in, not user-controlled  
✅ Samplers (DPM++, Euler, etc.) are variations on core algorithms – DPM++ is safest choice  

---

## Next Steps

- Finish the Welch Labs video if there's more content
- Install ComfyUI locally
- Generate first images with different parameters to see concepts in action
