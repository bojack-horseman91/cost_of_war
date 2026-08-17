# Estimating the Number of Impacted Infrastructure

This repository contains the Jupyter Notebooks used for processing, augmenting, and analyzing satellite imagery and text reports of kinetic events (airstrikes and missile attacks) tracked via ArcGIS and LiveUAMap. The pipeline utilizes Large Vision-Language Models (LVLMs), Large Language Models (LLMs), and the Segment Anything Model (SAM) to assess building damage, crater depths, blast radii, and weapon types.

## 📌 Important Notes

1. **Local TIFF Images:** The original `.tiff` files are provided in the dataset. You can modify the Step 2 notebooks to load these local files directly instead of downloading images from google or ESRI.
2. **Directory Naming:** References to `extracted_map_layers` within the notebooks refer specifically to the ArcGIS data and image files.

---

## 📂 Project Structure & Notebooks

### 1. Weapon Classification & Radius Calculation (Step 1)
These notebooks process textual and tabular data (such as event titles and summaries) to classify the munitions used in strikes and calculate corresponding blast/crater radii. This serves as the foundational first step before generating any images.

*   **`ARCGIS_WeaponClassfication_part.ipynb`**
    Processes the ArcGIS event datasets (`VIEW_V4_MDS_IranCrisisEvents2026_layer_46.csv` and `layer_48.csv`) to classify weapon types used in specific strike events and calculates the associated blast radius for each event.
*   **`LiveUAMap_LLM_weapon_classification.ipynb`**
    Passes the `event_full_title` text reports from the LiveUAMap dataset through an LLM to automatically categorize weapon/event types based on open-source intelligence reports, calculating expected radii for generated blast zones.

### 2. Data Preparation & Image Generation (Step 2)
Using the calculated radii and classified weapon data from Step 1, these notebooks handle extraction, segmentation, and preparation of satellite imagery for model inference.

*   **`SAM_SEGMENTATION_AND_AUGMENTED_IMAGE_GENERATION.ipynb`**
    Utilizes SAMGeo to perform zero-shot image segmentation on Google and ESRI satellite images. Generates outlined building masks, crater segmentations based on calculated radii, and builds visual comparison grids for analysis.
    > **Note:** File paths in this notebook are configured for LiveUAMap by default and must be modified when running for ArcGIS data.
*   **`Image_Creation_for_base_configs(A).ipynb`**
    Prepares the dataset for **Config A** by overlaying or embedding text and location labels inside visual blast zones (red zones) on satellite images to provide spatial context.
*   **`IMAGE_CREATION_RAW_CONFIG_B.ipynb`**
    Prepares the dataset for **Config B** by processing raw, unaugmented baseline satellite images (without text labels) to serve as a control group for model evaluation.

### 3. LVLM Inference Configurations (Step 3)
These notebooks feed prepared images and prompts into Large Vision-Language Models to estimate building damage (total, partial, and complete counts) and analyze blast zones across four distinct configurations:

*   **`LOCATION_LABEL(CONFIG_A)_LVLM_INFERENECE.ipynb`**
    **Config A:** Runs inference on images with embedded text/location labels inside the blast (red) zone. *(Note: Primarily applied to LiveUAMap data, as this did not improve performance for ArcGIS).*
*   **`RAW_BASELINE(CONFIG_B)_LVLM_INFERENECE.ipynb`**
    **Config B:** Runs baseline inference using raw, unaugmented satellite images to establish standard model performance without visual aids.
*   **`SEG_AUGMENTED(CONFIG_c)_LVLM_INFERENECE.ipynb`**
    **Config C:** Runs inference on SAM-segmented images, relying on visual outlines of destroyed and partially damaged buildings to guide model counting and damage assessment.
*   **`DEPTH_AUGMENTED(CONFIG_D)_LVLM_INFERENECE.ipynb`**
    **Config D:** Runs inference on 4-grid panel images that include depth-estimated visual mappings alongside SAM segmentations, providing spatial depth context for crater analysis.

### 4. Evaluation (Step 4)
*   **`evaluation_of_war_project.ipynb`**
    The final evaluation script that takes output predictions from all four LVLM configurations and LLM weapon classifications/radii and compares them against ground truth datasets (`ground_truth_for_ArcGIS.xlsx` and `ground_truth_for_LiveUAMap.xlsx`). Calculates error rates, accuracy metrics for building damage counts, and overall model performance.

---

## 🚀 Suggested Execution Order

If reproducing this pipeline from scratch, run the notebooks in the exact order below:

1. **Phase 1: Textual Weapon Classification & Radius Calculation** (`ARCGIS_WeaponClassfication_part.ipynb` and `LiveUAMap_LLM_weapon_classification.ipynb`)
2. **Phase 2: Image Generation & Masking** (`SAM_SEGMENTATION_AND_AUGMENTED_IMAGE_GENERATION.ipynb`, `Image_Creation_for_base_configs(A).ipynb`, `IMAGE_CREATION_RAW_CONFIG_B.ipynb`)
3. **Phase 3: LVLM Inference** (Configs A, B, C, and D notebooks)
4. **Phase 4: Evaluation** (`evaluation_of_war_project.ipynb`)
