# Brain Tumor Segmentation — Colab Demo

This is my M.Tech project demo for Explainable Brain Tumor Segmentation using MRI images. I built this as a prototype to show the full workflow — upload an MRI, detect if there's a tumor, highlight the region, and explain where the model looked. Everything runs inside Google Colab, no setup needed on your end.

---

## What this does

You upload a brain MRI image and the notebook gives you three things:

1. A segmentation overlay showing where the tumor was detected (red region on the MRI)
2. A heatmap showing which parts of the brain the algorithm paid most attention to
3. A diagnostic report at the bottom with coverage percentage, confidence score, and location

If no tumor is found, it tells you that clearly — no fake red blobs on a healthy scan. That was actually a bug in earlier versions that I fixed.

---

## How to run it

Open the notebook in Google Colab and run the three cells in order.

**Step 1** — installs the packages (pillow, numpy, matplotlib, scipy, scikit-image)

**Step 2** — a file upload button appears. Pick your MRI image. You can re-run this cell anytime to switch to a different image without restarting anything.

**Step 3** — runs everything. The result pops up right inside the notebook. It also saves to `/content/brain_tumor_result.png` so you can right-click and save it.

That's it. No API keys, no ngrok, no Cloudflare, nothing extra.

---

## How the detection works

I didn't want to use a simple threshold (like "flag anything above the 88th percentile brightness") because that will always find something suspicious even on a perfectly healthy scan. So instead I run three separate checks and only confirm a tumor if at least two of them agree.

**Check 1 — Bright spot detection**
Uses Otsu's method to find the optimal intensity cutoff for that specific image, then flags pixels that are significantly above even that threshold. Small noise blobs are cleaned out so artefacts don't trigger it.

**Check 2 — Hemisphere symmetry**
A healthy brain is roughly symmetric left to right. If one side is noticeably brighter or more variable than the other, that's flagged. I compare both mean intensity and standard deviation between the two halves.

**Check 3 — Local variance**
Tumors tend to have heterogeneous texture — a mix of necrotic core, enhancing rim, and edema creates sharp local intensity changes. I scan the brain in patches and flag areas with unusually high variance.

If 2 or 3 checks flag something, the system reports TUMOR DETECTED. If only 1 or 0 flag, it reports NO TUMOR DETECTED. The confidence score reflects how many checks agreed.

---

## The heatmap

The explainability heatmap is shown for every scan, not just ones with tumors. It's coloured blue to red — blue means the algorithm didn't pay much attention to that region, red means it did. When a tumor is found, the heatmap will have a strong red patch right over the detected area.

This is meant to simulate what Grad-CAM does in a real trained neural network — showing you not just what the model decided, but where it was looking when it made that decision. In the final version with the trained Attention U-Net, this will be a proper Grad-CAM heatmap computed from actual network gradients.

---

## Sample output

Here's what the output looks like on a tumor-positive scan:

![Demo result showing tumor detected on left side of brain with red segmentation overlay and Grad-CAM heatmap](result_sample.png)

The system detected the tumor on the left hemisphere with 88% confidence. All three checks flagged it — bright lesion at 8.76%, hemisphere asymmetry at 10.42%, local variance at 4.19%. Tumor coverage was 17.52% of brain area.

---

## Files in this repo

```
Brain_Tumor_Final_v7.ipynb   ← the notebook, run this in Colab
Brain_Tumor_Report_v2.docx   ← full project report
README.md                    ← you're reading this
```

---

## What's next

This prototype uses image processing rules. The actual final system I'm building will use a trained Attention U-Net on the BraTS 2020 dataset — 369 multi-modal MRI cases with expert annotations. The network takes four MRI sequences as input (T1, T1ce, T2, FLAIR) and outputs segmentation masks for three tumor sub-regions. Real Grad-CAM will replace the current attention proxy.

Evaluation will be done using Dice score, IoU, and Hausdorff distance on the BraTS validation set.

---

## Known issues

- Works on 2D single images only, not 3D MRI volumes
- The heatmap is simulated, not actual Grad-CAM from a neural network
- Thresholds were tuned manually and might not work perfectly on every scanner type or tumor subtype
- No ground truth comparison yet since this is a prototype

---

## References

- Ronneberger et al. (2015) — U-Net: Convolutional Networks for Biomedical Image Segmentation
- Oktay et al. (2018) — Attention U-Net: Learning Where to Look for the Pancreas
- Selvaraju et al. (2017) — Grad-CAM: Visual Explanations from Deep Networks
- Menze et al. (2015) — The Multimodal Brain Tumor Image Segmentation Benchmark (BRATS)
- Otsu (1979) — A Threshold Selection Method from Gray-Level Histograms

---

Academic prototype — not for clinical use.
