
# Image Outpainting Pipeline

## Overview
This repository contains a robust image outpainting pipeline leveraging **ControlNet Inpainting** on **Triton Inference Server**. The process involves expanding an image by filling in missing areas while preserving the original context and style. The pipeline ensures high-quality results by performing **pre-processing**, **model inference**, and **post-processing** steps.

---

## Processing Stages
### 1. Pre-processing
Before passing an image to the model, the system performs several validation and transformation steps to ensure compatibility and optimal results.

✅ **Image Loading & Validation:**
   - Load input image from a file or URL.
   - Extract essential metadata, including size and position.
   - Validate input parameters such as **canvas size**, **image size**, and **location**.

✅ **Ethical Compliance Check:**
   - The prompt is validated using `verify_generate_query(prompt)` to ensure adherence to ethical guidelines.

✅ **Image Resizing & Adaptation:**
   - Adjust image proportions to fit the required aspect ratio (`resize_canvas`).
   - If needed, downscale the image to ensure it does not exceed `MAX_NUM_PIXELS_FOR_IMAGE_EXPANSION_TRITON_MODEL`.
   - Generate a **binary mask** indicating areas for expansion (`mask_np`).

✅ **Seed Generation:**
   - If no seed is provided, a **random seed** is generated to ensure reproducibility.

---

### 2. Model Inference (Outpainting Process)
Once the image is prepared, it is passed through the **ControlNet Inpainting model** to generate the expanded regions.

✅ **Execution of ControlNet Inpainting:**
   - The model is run using the **Triton Inference Server**.
   - The `infer` function receives the **input image**, **mask**, **prompt**, and **conditioning parameters**.
   - The model outputs a new image where missing areas are intelligently filled.

---

### 3. Post-processing
After the model generates the expanded image, additional refinements are applied to ensure smooth blending and final output quality.

✅ **Seamless Blending (Poisson Blending):**
   - `poisson_blend` is used to **blend edges** between the original and generated regions for a seamless transition.

✅ **Cropping & Resizing:**
   - The expanded image is cropped back to match the **original canvas dimensions**.
   - If transparency was present in the original image, the alpha channel is restored.

✅ **Content Moderation:**
   - The output is checked for inappropriate content using `is_appropriate(image_result)`.
   - If flagged, the image is blocked and not returned.

✅ **Saving & Uploading the Final Image:**
   - The final image is uploaded to **AWS S3** using `s3_upload_image`.
   - Metadata such as the **seed**, **prompt**, and **status** is logged for tracking.

✅ **Logging & API Integration:**
   - The result is sent to `post_image_generation` for tracking and further API usage.

---

## Summary
| Stage        | Description |
|-------------|------------|
| **Pre-processing** | Validate inputs, resize the image, generate masks, and ensure compliance. |
| **Model Processing** | Execute ControlNet Inpainting on Triton Inference Server. |
| **Post-processing** | Blend edges, crop, validate content, and upload results to S3. |


