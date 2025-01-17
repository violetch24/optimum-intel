<!---
Copyright 2020 The HuggingFace Team. All rights reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Stable Diffusion

The script [`run_diffusion.py`](https://github.com/huggingface/optimum-intel/blob/main/examples/neural_compressor/text-to-image/run_diffusion.py)
allows us to apply different post-training quantization approaches (such as dynamic and static) using the [Intel Neural Compressor ](https://github.com/intel/neural-compressor) library for text to image tasks.

The following example applies post-training static quantization on a stable diffusion [model](https://huggingface.co/lambdalabs/sd-pokemon-diffusers)fine-tuned on Pokémon. In this example, we only quantize the unet model which is the performance critical in the diffusion pipeline, the diffusion pipeline has four models: safety_checker, text_encoder, unet, vae.

### Ground Truth Images
In this case, we used FID metric to evaluate the model, so we should download [training datasets](https://huggingface.co/datasets/lambdalabs/pokemon-blip-captions) and choose one image to a directory(like "base_images").
>**Note:** In this case we used picture: [Ground_Truth_Image](https://datasets-server.huggingface.co/assets/lambdalabs/pokemon-blip-captions/--/lambdalabs--pokemon-blip-captions/train/14/image/image.jpg).

## Quantization
```bash
python run_diffusion.py \
    --model_name_or_path lambdalabs/sd-pokemon-diffusers \
    --apply_quantization \
    --quantization_approach static \
    --tolerance_criterion 0.02 \
    --verify_loading \
    --output_dir /tmp/diffusion_output \
    --base_images base_images \   # The path of the ground truth pictures
    --input_text "a drawing of a gray and black dragon" \
    --calib_text "a drawing of a green pokemon with red eyes" # The prompt to calibrate for static quantization
```

In order to apply dynamic or static, `quantization_approach` must be set to respectively `dynamic` or `static`.

The configuration file containing all the information related to the model quantization can be 
specified using respectively `quantization_config`. If not specified, the default
[quantization](https://github.com/huggingface/optimum-intel/blob/main/examples/neural_compressor/text-to-image/quantization.yml),
configuration files will be used.

The flag `--verify_loading` can be passed along to verify that the resulting quantized model can be loaded correctly.

The ground truth image:

<div align="left">
<img src=images/ground_truth.png width=40%/>
</div>

The image generated by original model(FID with ground truth: 333):

<div align="left">
<img src=images/fp32.png width=40%/>
</div>

The image generated by quantized UNet(FID with ground truth: 246):

<div align="left">
<img src=images/int8.png width=40%/>
</div>

## Performance
Original model
```bash
python run_diffusion.py \
    --model_name_or_path sd-pokemon-diffusers \
    --output_dir /tmp/diffusion_output \
    --base_images base_images \   # The path of the ground truth pictures
    --input_text "a drawing of a gray and black dragon" \
    --benchmark
```
The model of quantized UNet
```bash
python run_diffusion.py \
    --model_name_or_path sd-pokemon-diffusers \
    --output_dir /tmp/diffusion_output \
    --base_images base_images \   # The path of the ground truth pictures
    --input_text "a drawing of a gray and black dragon" \
    --benchmark \
    --int8
```
>**Note:** You will see the inference performance speedup with Intel DL Boost (VNNI) on Intel(R) Xeon(R) hardware. You may also refer to [Performance Tuning Guide](https://intel.github.io/intel-extension-for-pytorch/cpu/latest/tutorials/performance_tuning/tuning_guide.html) for more optimizations.
