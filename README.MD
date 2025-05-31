- [Abstract](#abstract)
- [✨ Citation](#-citation)
- [Project structure](#project-structure)
- [Installation](#installation)
- [Running](#running)
  - [Inference](#inference)
  - [Evaluate](#evaluate)
- [License](#license)

<p align="center">
  <h1 align="center">SwiftBrush v2: Make Your One-step Diffusion Model Better Than Its Teacher</h1>
  <h3 align="center">ECCV 2024</h3>
  <p align="center">
    <a href="https://trung-dt.com/"><strong>Trung Dao</strong></a>
    &nbsp;&nbsp;
    <a href="https://thuanz123.github.io"><strong>Thuan Hoang Nguyen</strong></a>
    &nbsp;&nbsp;
    <a href="https://luvata.github.io/"><strong>Thanh Le</strong></a>
    &nbsp;&nbsp;
    <a href="https://www.vinai.io"><strong>Duc Vu</strong></a>
  </p>
  <p align="center">
    <a href="https://www.khoinguyen.org"><strong>Khoi Nguyen</strong></a>
    &nbsp;&nbsp;
    <a href="https://sites.google.com/view/cuongpham/home"><strong>Cuong Pham</strong></a>
    &nbsp;&nbsp;
    <a href="https://scholar.google.com/citations?user=FYZ5ODQAAAAJ&hl=en"><strong>Anh Tran</strong></a>
  </p>
  <br>

<div align="center">
        <img src="./assets/banner_v2.png", width="500">
  </div>

<p align="center">
    <a href="http://arxiv.org/abs/2408.14176"><img alt='arXiv' src="https://img.shields.io/badge/arXiv-2312.07409-b31b1b.svg"></a>
    <a href="https://swiftbrushv2.github.io"><img alt='page' src="https://img.shields.io/badge/Project-Website-pink"></a>
    <a href="https://vinairesearch.github.io/SwiftBrush/"><img alt='page' src="https://img.shields.io/badge/SwiftBrushV1-Website-purple"></a>
  </p>
  <br>
</p>

## Abstract

> &#160;   In this paper, we aim to enhance the performance of SwiftBrush, a prominent one-step text-to-image diffusion model, to be competitive with its multi-step Stable Diffusion counterpart. Initially, we explore the quality-diversity trade-off between SwiftBrush and SD Turbo: the former excels in image diversity, while the latter excels in image quality. This observation motivates our proposed modifications in the training methodology, including better weight initialization and efficient LoRA training. Moreover, our introduction of a novel clamped CLIP loss enhances image-text alignment and results in improved image quality. Remarkably, by combining the weights of models trained with efficient LoRA and full training, we achieve a new state-of-the-art one-step diffusion model, achieving an FID of 8.14 and surpassing all GAN-based and multi-step Stable Diffusion models.

## ✨ Citation

Please CITE [our paper](http://arxiv.org/abs/2408.14176) whenever this repository is used to help produce published results or incorporated into other software:

```bib
@inproceedings{dao2024swiftbrushv2,
  title={SwiftBrush v2: Make Your One-step Diffusion Model Better Than Its Teacher},
  author={Trung Dao and Thuan Hoang Nguyen and Thanh Le and Duc Vu and Khoi Nguyen and Cuong Pham and Anh Tran},
  booktitle={Proceedings of the European Conference on Computer Vision (ECCV)},
  year={2024}
}
```

## Project structure

- We provide the following files and folders:
  - `tools`: utils code such as: text embedding extraction, resizing
  - `eval`: evaluation code
  - `dataset.json`: The MS-COCO2014 prompts used for evaluation.
- We also provide the checkpoint at this [link](https://drive.google.com/drive/folders/1eUVwTrkOVWT2gCJ4TiWlZmCV2sODuvQD?usp=drive_link).
- NOTE: While our codebase uses the 3-Clause BSD License, our model is derived from SD-Turbo and therefore must comply with SD-Turbo's [original license](https://huggingface.co/webml/models/blob/5085cddea6b3138215675660ec708f215cb28dd4/sd-turbo/LICENSE.txt)

## Installation

- First create a torch-cuda available environment:
  ```bash
  conda create -n sbv2 python=3.10
  conda activate sbv2
  conda install pytorch==2.2.2 torchvision==0.17.2 torchaudio==2.2.2 pytorch-cuda=11.8 -c pytorch -c nvidia
  ```
- Install the remaining dependencies:
  ```bash
  pip install -r requirements.txt
  ```
- For evaluating `recall`, please create another environment:
  ```bash
  conda create -n sbv2_recall python=3.10
  conda activate sbv2_recall
  pip install -r requirements_eval_recall.txt
  ```

## Running

### Inference

1. Infer a normal prompt or a txt file of prompts by using `infer_by_prompt.py`

```bash
    python infer_by_prompt.py <prompt> <ckpt_path>
```

1. Infer the COCO2014 prompts using the json that we delivered using the corresponding `infer.py` script.

- Easier to run but longer: `python infer.py <ckpt_path> --caption-path=dataset.json`
- Faster route:
  - Generate the embeddings: `python tools/prepare.py dataset.json`
  - Infer using the embeddings: `python infer.py <ckpt_path> --caption-path=dataset.json --precomputed-text-path-path=<generated_embeds_path>`

### Evaluate

- Following GigaGAN paper, we evaluate the model with the following flow:
  1. Center-crop the COCO2014 to 256x256 images
  ```bash
    python tools/resize.py <coco2014_path> <coco2014_resized_path> --nsamples=-1
  ```
  2. Resize the inferred folder:
  ```bash
  python tools/resize.py main <infer_folder> <infer_resized_folder>
  ```
  3. Evaluate:
  - **FID**:
    ```bash
    python eval/fid.py <infer_resized_folder> --ref-dir=<coco2014_resized_path> --no-crop
    ```
  - **CLIP Score**:
    ```bash
    python eval/clip_score.py <infer_resized_folder>  --batch-size=1024 --prompt_path=<coco2014_prompt_path>
    ```
  - **Precision/Recall**:
    ```bash
    python eval/recall.py <coco2014_resized_path> <infer_resized_folder>
    ```
  - One can use `scripts/eval.sh` to automate step 4, but remember to change the paths
- For HPSv2 evaluation metrics:
  - Infer using the corresponding `infer_hps.py` script:
    ```bash
    python infer_hps.py <ckpt_path>
    ```
  - Get the final score with:
    ```bash
    python eval/hps.py <hps_infer_folder>
    ```

## License

```
Copyright (c) 2024 VinAI
Licensed under the 3-Clause BSD License.
You may obtain a copy of the License at
    https://opensource.org/license/bsd-3-clause
```
