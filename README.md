# Neural 3D Reconstruction in the Wild
### [Project Page](https://zju3dv.github.io/neuralrecon-w) | [Paper](https://arxiv.org/pdf/2205.12955)
<br/>

> Neural 3D Reconstruction in the Wild  
> [Jiaming Sun](https://jiamingsun.ml), [Xi Chen](https://github.com/Burningdust21), [Qianqian Wang](https://www.cs.cornell.edu/~qqw/), [Zhengqi Li](https://zhengqili.github.io/), [Hadar Averbuch-Elor](https://www.cs.cornell.edu/~hadarelor/), [Xiaowei Zhou](https://xzhou.me), [Noah Snavely](https://www.cs.cornell.edu/~snavely/)  
> SIGGRAPH 2022 (Conference Proceedings)

![demo_vid](assets/neuconw-github-teaser.gif)

## TODO List
- [x] Training and inference code.
- [x] Pipeline to reproduce the evaluation results on the proposed Hritage-Recon dataset.
- [x] Config for reconstructing generic indoor scenes.

## Installation

```shell
conda env create -f environment.yaml
conda activate neuconw
scripts/download_sem_model.sh
```

## Reproduce reconstruction results on Heritage-Recon
### Data setup

Download the [Heritage-Recon](https://drive.google.com/drive/folders/1ch-RRnC2CrYSeKpbldSwZu5ifKQHS_CU?usp=sharing) dataset and put it under `data`. You can also use gdown to download it in command line:

```
mkdir data && cd data
gdown --id 1ch-RRnC2CrYSeKpbldSwZu5ifKQHS_CU
```

### Training
To train scenes in our Heritage-Recon dataset: 

```bash
scripts/train.sh $EXP_NAME config/train_${SCENE_NAME}.yaml $NUM_GPU $NUM_NODE
```

Subsutitude `SCENE_NAME` with the scene you want to train. Please refer to our paper for training time.

### Evaluating

First, extracting mesh from a checkpoint you want to evaluate:

```bash
scripts/sdf_extract.sh $EXP_NAME config/train_${SCENE_NAME}.yaml $CKPT_PATH 10
```

The reconstructed meshes will be saved to `PROJECT_PATH/results`.

Then run the evaluation pipline:

```bash
scipts/eval_pipeline.sh $SCENE_NAME $MESH_PATH
```

Evaluation results will be saved in the same folder as the evaluated mesh.


## Reconstructing custom data

### Ddata preparation

#### Auto generation

We take any COLMAP workspace as input, a script is provided for autolmatically convert a colmap workspace into our data format:

```bash
scripts/preprocess_data.sh
```

More instructions can be found in `scripts/preprocess_data.sh`

#### Mannuly select

However, if you wish to select a better bounding box mannuly, do the following steps.

#### 1. Generate semantic maps

Generate semantic maps:

```bash
python tools/prepare_data/prepare_semantic_maps.py --root_dir $WORKSAPCE_PATH --gpu 0
```

#### 2. Create scene metadata file

Create a file `config.yaml` into worksapce to write metadata. The target scene needs to be normalized into a unit sphere, which require manual selection. One simple way is to use SFM key-points points from COLMAP to determine the origin and radius. Also a bounding box is required, which can be set to `[origin-raidus, origin+radius]`, or only the region you're interested in.

```yaml
{
    name: brandenburg_gate, # scene name
    origin: [ 0.568699, -0.0935532, 6.28958 ], 
    radius: 4.6,
    eval_bbx: [[-14.95992661, -1.97035599, -16.59869957],[48.60944366, 30.66258621, 12.81980324]],
    voxel_size: 0.25,
    min_track_length: 10,
    # The following configuration is only used in evaluation, can be ignored for your own scene
    sfm2gt: [[1, 0, 0, 0],
            [ 0, 1, 0, 0],
            [ 0, 0, 1, 0],
            [ 0, 0, 0, 1]],
}
```

#### 3. Generate cache

run following command with specified `WORKSAPCE_PATH`:

```bash
scripts/data_generation.sh $WORKSAPCE_PATH
```

After completing above steps, whether automaticaly or manully, the COLMAP workspace should be looking like this;

```bash
└── brandenburg_gate
  └── brandenburg_gate.tsv
  ├── cache_sgs
    └── splits
        ├── rays1_meta_info.json
        ├── rgbs1_meta_info.json
        ├── split_0
            ├── rays1.h5
            └── rgbs1.h5
        ├── split_1
        ├──.....
  ├── config.yaml
  ├── dense
    └── sparse
        ├── cameras.bin
        ├── images.bin
        ├── points3D.bin
  └── semantic_maps
      ├── 99119670_397881696.jpg
      ├── 99128562_6434086647.jpg
      ├── 99250931_9123849334.jpg
      ├── 99388860_2887395078.jpg
      ├──.....
```

### Training

Change `DATASET.ROOT_DIR` to COLMAP workspace path in `config/train.yaml`, and run:

```bash
scripts/train.sh $EXP_NAME config/train.yaml $NUM_GPU $NUM_NODE
```

Additionally, `NEUCONW.SDF_CONFIG.inside_outside` should be set to `True` if training an indoor scene.

### Extracting mesh

```bash
scripts/sdf_extract.sh $EXP_NAME config/train.yaml $CKPT_PATH $EVAL_LEVEL
```

The reconstructed meshes will be saved to `PROJECT_PATH/results`.

## Citation

If you find this code useful for your research, please use the following BibTeX entry.

```bibtex
@inproceedings{sun2022neuconw,
  title={Neural {3D} Reconstruction in the Wild},
  author={Sun, Jiaming and Chen, Xi and Wang, Qianqian and Li, Zhengqi and Averbuch-Elor, Hadar and Zhou, Xiaowei and Snavely, Noah},
  booktitle={SIGGRAPH Conference Proceedings},
  year={2022}
}
```

## Acknowledgement
Part of our code is borrowed from [nerf_pl](https://github.com/kwea123/nerf_pl) and [NeuS](https://github.com/Totoro97/NeuS), thanks to their authors for the great works.