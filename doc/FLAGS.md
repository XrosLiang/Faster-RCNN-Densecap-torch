# Flags
This file documents all available flags for all scripts.

## Common flags
There are some common flags that are shared by many Lua scripts:

- `-checkpoint`: Path to the checkpoint containing the model to use
- `-image_size`: Before being processed by the model, images will be resized so their max side length is equal to this;
  larger values will make the model run more slowly.
- `-test_rpn_nms_thresh`: Threshold for non-maximum suppression among region proposals in the region proposal network; should
   be a number between 0 and 1.
- `-test_final_nms_thresh`: Threshold for non-maximum suppression among final output boxes; should be a number between 0 and 1.
  Using a larger value will cause the model to produce more detections, but they will have more overlap with each other.
- `-num_proposals`: The number of region proposals that should be run through the recognition network and language model.
  Using a larger number will tend to give better results, but will also make the model run slower.
- `-gpu`: The (zero-indexed) index of the GPU to use. Setting this to -1 will run in CPU-only mode.
- `-use_cudnn`: Setting this to 1 will enable the use of the [cuDNN](https://developer.nvidia.com/cudnn) library when
  running in GPU mode. Setting this to 0 will fall back to the default torch implementation of cuDNN layers.
- `-anchor_type`: The type of anchors used. Could choose from densecap, voc and coco (voc and coco is that in the faster-rcnn code)

## preprocess2.py
The script `preprocess2.py` is used to preprocess a dataset of images, regions from Pascal VOC, and convert the entire
dataset (except raw images) to an HDF5 file and a JSON file to be read by Lua scripts.

During preprocessing we remove images with no region annotations.

The following flags are available:

**Input data**:
- `--data_dir`: Directory of the VOC data, could exactly be what you downloaded from VOC website. You can also merge the images and annotations to another directory which has the same structure of the normal VOC directory.

**Output data**:
- `--json_output`: Path where the output JSON file should be written; this file contains the mapping from
  class label indices to strings.
- `--h5_output`: Path where the output HDF5 file should be written; this file contains all regions, class labels and etc.

**Options**:
- `--train_split`: Image ids of training data. (Default is the public training splitin the dataset)
- `--val_split`: like above
- `--test_split`: like above

** You can also use preprocess.py which will contain images in the dataset, in that way, you need to use 'DataLoader.lua' instead of 'DataLoader_new.lua' **

## run_model.lua
The script `run_model.lua` is used to run a trained model. It can take as input a single image, a directory of images, or
an HDF5 file produced by `preprocess.py`. It can output JSON files to be viewed by the HTML visualizer, or image files with
a prespecified number of detections baked in.

The following flags are available:

**Model options**:
- `-checkpoint`: [See above](#common-flags)
- `-image_size`: [See above](#common-flags)
- `-rpn_nms_thresh`: [See above](#common-flags)
- `-final_nms_thresh`: [See above](#common-flags)
- `-num_proposals`: [See above](#common-flags)
- `-gpu`: [See above](#common-flags)
- `-use_cudnn`: [See above](#common-flags)

**Input options**:
At least one of the following must be provided:
- `-input_image`: Path to a single image on which to run the model
- `-input_dir`: Path to a directory of images on which to run the model
- `-input_split`: One of `train`, `val`, or `test` indicating a split of the Visual Genome data on which to run
  - `-splits_json`: Path to a JSON file containing splits of the Visual Genome data; only used if `-input_split`
     is provided.
  - `-vg_img_root_dir`: Path to a directory containing all of the Visual Genome images; only used if `-input_split`
    is provided.
    
**Output options**:
The script can produce two types of outputs. By default it will write images and JSON files to the `vis/data`
directory, which can be visualized with the HTML viewer. By providing the `-output_dir` flag the script will also
produce images with a certain number of detections and captions baked in.
- `-max_images`: The maximum number of images to process
- `-output_dir`: If provided, the directory to which images with "baked in" detections should be written
  - '-num_to_draw`: The number of detections to draw in output images; only used if `-output_dir` is provided
  - `-text_size`: The font size to use for captions in output images; only used if `-output_dir` is provided
  - `-box_width`: Width (in pixels) of boxes in output images; only used if `-output_dir` is provided.
- `-output_vis`: If 1 (default) then output JSON files and images for the HTML viewer
  `-output_vis_dir`: Directory to which images and JSON files (for the HTML viewer) should be written;
  default is `vis/data`.

## train.lua
The script `train.lua` uses the HDF5 and JSON files produced by `preprocess2.py` to train models. This script periodically checks accuracy on the validation set, and saves checkpoints containing trained models.

In most cases the only settings you will need to change are `-learning_rate`, `-checkpoint_start_from` to start
training from a checkpoint rather than training from scratch, `-finetune_cnn_after` to enable finetuning of the CNN,
and `-checkpoint_path` to change the location of saved checkpoints.

~~
## evaluate_model.lua
The script `evaluate_model.lua` is used to quantitatively evaluate a trained model using our mean average precision metric;
a model is run on images in a preprocessed dataset that has been created by `preprocess.py`.
This script assumes that METEOR has been downloaded by running `scripts/setup_eval.sh`, and requires both Python and Java.
~~

The following flags are available:

**Model options**:
- `-checkpoint`: [See above](#common-flags)
- `-rpn_nms_thresh`: [See above](#common-flags)
- `-final_nms_thresh`: [See above](#common-flags)
- `-num_proposals`: [See above](#common-flags)
- `-gpu`: [See above](#common-flags)
- `-use_cudnn`: [See above](#common-flags)

**Other options**:
- `-data_h5`: Path to an HDF5 file created by `preprocess.py`.
- `-data_json`: Path to a JSON file created by `preprocess.py`.
- `-split`: Which split to use; either `train`, `val`, or `test`.
- `-max_images`: How many images to use for evaluation; set to -1 to use the entire split.
