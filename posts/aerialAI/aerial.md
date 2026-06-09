# Aerial AI

## Project
Aerial AI is a dual-task segmentation project for aerial imagery:
- Semantic segmentation: buildings, roads, water
- Instance segmentation: solar panels
- Web inference UI with Streamlit in app.py

## Datasets
- Aerial Segmentation (Kaggle): RGB aerial images + masks for semantic classes
- Solar Plants Brazil: TIFF imagery + binary masks for solar panel instances
- Indian Demo: sample aerial TIFF images for testing/inference

## Models
- Semantic model: SegFormer (`nvidia/segformer-b1-finetuned-cityscapes-1024-1024`)
- Instance model: Mask2Former (`facebook/mask2former-swin-base-coco-instance`)
- Fine-tuned outputs are stored under:
  - output/semantic/best_model
  - output/instance/best_model

## Architecture (Brief)
1. Data prep/download scripts -> organized train/val folders
2. Training scripts:
   - train_semantic.py (SegFormer)
   - train_instance.py (Mask2Former)
3. Inference/UI:
   - app.py loads best checkpoints and runs selected task
   - overlay + basic stats shown in Streamlit

## App Screenshot
![Aerial AI Streamlit inference interface showing segmentation overlays](aerial.png)

## Demo Video
<video controls width="100%" poster="aerial.png">
   <source src="aerial.mp4" type="video/mp4">
   Your browser does not support embedded videos. Download it here:
   <a href="aerial.mp4">aerial.mp4</a>
</video>

## Train (Brief)
```bash
# Train both models (Windows)
train_all.bat

# Or train individually
python train_semantic.py --train_image_dir ./data/aerial_segmentation/train/images --train_mask_dir ./data/aerial_segmentation/train/masks --val_image_dir ./data/aerial_segmentation/val/images --val_mask_dir ./data/aerial_segmentation/val/masks
python train_instance.py --train_image_dir ./data/solar_panels/train/images --train_mask_dir ./data/solar_panels/train/masks --val_image_dir ./data/solar_panels/val/images --val_mask_dir ./data/solar_panels/val/masks
```

## Run (Brief)
```bash
# Launch web app
streamlit run app.py
```

Source: [aerial_AI](https://github.com/Ajeets6/aerial_AI)
