**How do AI Images Work by Welch Labs**
https://youtu.be/iv-5mZ\_9CPY
---

#### 

* **How does AI Image generation work?**
* 

**A Naive understanding: Randomly generated noise is fed through a Transformer (MLM) by various repeating steps "unblurring" the Image. ("Anti-Diffusion")**



###### **CLIP - How It Works**

###### 

**CLIP is fundamentally a bridge between images and text – it learns to understand what text descriptions match what images.**



* Two Encoders: CLIP has an Image Encoder and a Text Encoder that transform images and text into vectors (numerical representations)
* Shared Vector Space: Both image vectors and text vectors exist in the same high-dimensional space (512 dimensions). They're positioned as points in this space.
* Learning Through Training Data: CLIP is trained on millions of image-text pairs from the internet. During training, it learns to position matching images and text close together through iterative adjustment.
* Similarity = Proximity: The closer two vectors are in the space, the more similar their meaning. Distance is measured using cosine similarity.
* Through training on many examples, directional relationships emerge.
  E.g. The Delta of the vectors **photo of a man** **and photo of a man wearing a hat** gives you "a direction" and if you were to "travel" in that direction (the direction is the Delta Vector of the two images) you would either "fly" through the position of the hat vector or be close to it
* This way text and Images are trained to form relationships.





###### **Image Generation - How it Works**



* Firstly a Model is being fed Training Data: You take an Image and incrementally add Noise to it until it results in pure noise.
* The old way to generate images would be to give the Diffusion model the task: Given noise at step N, predict what the image should be at step N-1. This resulted in low quality results.
* In new DDPM Models(Denoising Diffusion Probabilistic Models) random noise in incrementally added throughout the diffusion process to enhance results. Also, the transformer is being asked to not predict what the image should be at step N-1, but at Step 0. 
  	In practice this looks like this:
  	In an example the DDPM Diffusor is asked to generate the image in 20 Steps
  		Input: Pure noise

&nbsp;			Model predicts: Step 0 (what the final image should be)

&nbsp;			Blend: 5% predicted image + 95% noise

&nbsp;			Output: Slightly denoised image



&nbsp;				**Step 2:**

&nbsp;				Input: Slightly denoised image from Step 1

&nbsp;				Model predicts: Step 0 again (refined prediction)

&nbsp;				Blend: 10% predicted image + 90% noise

&nbsp;				Output: More denoised image



&nbsp;					**Step 3:**

&nbsp;					Input: More denoised from Step 2

&nbsp;					Model predicts: Step 0 again

&nbsp;					Blend: 15% predicted + 85% noise

&nbsp;					Output: Even cleaner



&nbsp;						... repeat 20 times total



&nbsp;						**Final Step** (Step 20):

&nbsp;						Input: Nearly clean image

&nbsp;						Model predicts: Step 0

&nbsp;						Blend: 100% predicted image + 0% noise

&nbsp;						Output: Final clean image



• Without conditioning this result in unpreditable random images. The CLIP vector would act as a steering mechanism during denoising.

