---
title: A Possible Way to Remove Watermark perfectly
date: 2020-11-08 19:41:34
tags: 
	- Machine Learning
	- Deep Learning
	- Project
categories: 
	- Machine Learning
	- Project
id: WatermarkRemoval
---
Although the traditional method of image watermark removal is efficient, it is more damaging to the details. It may takes a few seconds for some watermarks to be removed using repair stamp, and a few watermarks may be failed to be removed even given  one or two hours. 

Some images that are not very rich in detail can be filled with adjacent pixels through Photoshop and other image processing software to cover up the watermark part, which can achieve near-perfect results.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://s1.ax1x.com/2020/11/09/BHcTpQ.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">From left to right, Original image (with watermark), PS processing (heavy loss of detail), Deep de-watermarking (full details).</div>
</center>

PS is no longer perfect for some extremely detailed and complex images. Traditional PS watermark removal methods can no longer meet the demand for detailed and complex watermarks.

Nowadays, using AI technology to remove watermarks, we can achieve almost perfection.

<!-- more -->

> With the continuous development of artificial intelligence technology, deep learning its application in the filed of image processing is becoming more and more widespread. At ICML 2018, researchers from institutions such as NVIDIA and MIT showcased an image degradation technology, Noise2Noise, which automatically removes watermarks, blurs and other noises from images, almost perfectly restores them, and the rendering time is milliseconds. Reference: [Noise2Noise: Learning Image Restoration without Clean Data](https://arxiv.org/abs/1803.04189)

Third party reproduction project: [yu4u/noise2noise](https://link.zhihu.com/?target=https%3A//github.com/yu4u/noise2noise) . This can be used to remove subtitles and image noise, but the author did not add a watermark removal feature. I've modified this python script to remove the watermark already. **But it only works on computers with NVIDIA series graphics card.**

# Preparations

## Download Script

[https://github.com/shimmerjordan/n2n-dewatermark](https://github.com/shimmerjordan/n2n-dewatermark)

## Build environment

First go to the NVIDIA website and download the latest version of the graphics driver for your computer's graphics card. (studio version - for designer PS modeling and drawing, GRD version - for playing games)

[NVIDIA Driver Download](https://www.nvidia.cn/Download/index.aspx?lang=cn)

Many people are using older versions of video card drivers. So the cuda version is too low and the tf frame runs up reporting errors.

After installation, restart your computer, right-click on the desktop and click NVIDIA Control Panel -> System Information -> Components.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/17/DZYscn.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">System Information</div>
</center>
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/17/DZtpjI.jpg" width="70%" height="70%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">CUDA version</div>
</center>
See if the CUDA version is higher than 9.0, otherwise it will report an error. If it is still below 9.0, use a software program such as Software Manager to uninstall any software on the computer that starts with NVIDIA, and then re-download the latest version of the graphics card driver from the address above. Then we need to download and install the CUDA through official website: [https://developer.nvidia.com/cuda-toolkit-archive](https://developer.nvidia.com/cuda-toolkit-archive). Besides, we should go to the official website and select the corresponding version of cuDNN to download.
[https://developer.nvidia.com/cudnn](https://developer.nvidia.com/cudnn).

>**1、CUDA**
>
>CUDA (Compute Unified Device Architecture) is a computing platform from graphics card manufacturer NVIDIA. CUDA is a general-purpose parallel computing architecture introduced by NVIDIA, which enables GPUs to solve complex computing problems.
>
>**2、CUDNN**
>
>NVIDIA cuDNN is a GPU acceleration library for deep neural networks. With an emphasis on performance, ease of use, and low memory overhead, NVIDIA cuDNN can be integrated into higher-level machine learning frameworks such as Google's Tensorflow and UC Berkeley's popular caffe software. The simple **insert design** allows developers to focus on designing and implementing neural network models rather than simply tuning performance, while also enabling high-performance modern parallel computing on the GPU.
>
>  **3、Relationship between CUDA & CUDNN**
>
>CUDA is seen as a workbench, equipped with many tools, such as a hammer, screwdriver, etc. cuDNN is a deep learning GPU accelerated library based on CUDA, with which deep learning calculations can be done on the GPU. It's the equivalent of a working tool, like it's a wrench. But CUDA, the workbench, didn't come with a wrench when it was bought. To run a deep neural network on CUDA, you have to install cuDNN, just like you have to buy the wrench back if you want to twist a nut. This is what allows the GPU to do deep neural network work and work much faster than the CPU.
>
>**4、CUDNN will have no impact on CUDA**
>
>From the official installation guide, we can see that as long as the cuDNN file is copied to the corresponding folder of CUDA, that is, the so-called plug-in design, the cuDNN database is added to CUDA, cuDNN is the extended computing library of CUDA, will not cause other effects on CUDA.
>
>As you can see, the existing files in CUDA and cuDNN do not have the same files, and after copying the files of cuDNN, the files in CUDA will not be overwritten, and other files in CUDA will not be affected.

Then build the python run container. For convenience, go straight to the highly integrated IDE, Anaconda. The tensorflow-gpu version needs to match the cuda/cudnn version, otherwise the script will run and report an error.

We open Anaconda Prompt and enter the following command into it:

````
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
````

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/18/DmAjU0.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Add channels</div>
</center>


Then press enter twice and copy this line again:

````
conda install tensorflow-gpu
````

After the installation we can close the Anaconda Prompt. Here I have created and activated a *DeepLearning* environment in the *env* under *Anaconda*, where all operations take place.

## Prepare data set

Download the coco2017 dataset at [http://images.cocodataset.org/zips/val2017.zip](http://images.cocodataset.org/zips/val2017.zip)

This zip file can decompress 5,000 images, of which 4,200 are used for training. The remaining 800 images are used for testing.

Open the n2n-watermark-remove-master directory that we have just extracted to the desktop, enter the dataset inside, then enter the train, use the file manager that comes with windows to randomly select 800 of the pictures, right-click and move them to the test directory. The remaining 4200 pictures (ctrl+A) are all selected and moved to the train directory.

## Acquisition/Production of watermarks

This step is very important, because if you want the computer to remove the watermark, you have to teach the computer to distinguish the watermark, and the watermark can only be removed if the computer learns to distinguish which parts of a watermarked image are watermarked and which parts are not. The key to this step is to find the original watermark image. Usually logos are watermarked and the logo image can be found on their website, (if it is on a white background it needs to be keyed).
Of course you can also use a cleverer approach, assuming that all the images on a website have a uniform style of watermark, you just need to go to the website and upload an image with a solid background (50% neutral grey recommended), let the system add a watermark to this image, and then calculate the difference through image subtraction, which is also the watermark image.

## Train models

Use the command via Anaconda Prompt to access the project directory of n2n-watermark-remove. Then use the following command to install the dependencies:

````
pip install -r requirements.txt
````

It may be necessary to wait a while for this to be installed, after which the training operation can be carried out.

### Train with watermark

````
python train.py --image_dir dataset/train --test_dir dataset/test --image_size 128 --batch_size 8 --lr 0.001 --source_noise_model text,0,50 --target_noise_model text,0,50 --val_noise_model text,25,25 --loss mae --output_path text_noise
````

The training time is determined by the graphics card. They generally range from a few dozen hours to several hundred hours. If possible, Kaggle and Google Colab can be used.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/20/DQorBF.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">The Beginning of Training</div>
</center>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/20/DQoghR.jpg">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">The Process of Training</div>
</center>

We can see the ETA, loss and the PSNR during the process(Some index such as batch_size was initialized in the command above).

During the training process, each iteration will generate a weights.xxxxxx-xxxx.hdf5 model file. The hdf5 file is not always generated, and there are times when it is normal that it is not. 

The number at the beginning represents the number of laps. The higher the number, the better the de-watermarking effect. The default for this script is 100 laps. The script runs by default for 100 laps. Normally the window is closed after about 50 laps and the resulting model is de-watermarked.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/20/DQRNM4.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">De-watermark model</div>
</center>

**Further, the model can be trained on the noise of the image and achieve a noise reduction effect.**

### Train with Gaussian noise

````python
# train model using (noise, noise) pairs (noise2noise)
python3 train.py --image_dir dataset/291 --test_dir dataset/Set14 --image_size 128 --batch_size 8 --lr 0.001 --output_path gaussian

# train model using (noise, clean) paris (standard training)
python3 train.py --image_dir dataset/291 --test_dir dataset/Set14 --image_size 128 --batch_size 8 --lr 0.001 --target_noise_model clean --output_path clean
````

### Train with text insertion

```python
# train model using (noise, noise) pairs (noise2noise)
python3 train.py --image_dir dataset/291 --test_dir dataset/Set14 --image_size 128 --batch_size 8 --lr 0.001 --source_noise_model text,0,50 --target_noise_model text,0,50 --val_noise_model text,25,25 --loss mae --output_path text_noise

# train model using (noise, clean) paris (standard training)
python3 train.py --image_dir dataset/291 --test_dir dataset/Set14 --image_size 128 --batch_size 8 --lr 0.001 --source_noise_model text,0,50 --target_noise_model clean --val_noise_model text,25,25 --loss mae --output_path text_clean
```

### Train with random-valued impulse noise

```python
# train model using (noise, noise) pairs (noise2noise)
python3 train.py --image_dir dataset/291 --test_dir dataset/Set14 --image_size 128 --batch_size 8 --lr 0.001 --source_noise_model impulse,0,95 --target_noise_model impulse,0,95 --val_noise_model impulse,70,70 --loss l0 --output_path impulse_noise

# train model using (noise, clean) paris (standard training)
python3 train.py --image_dir dataset/291 --test_dir dataset/Set14 --image_size 128 --batch_size 8 --lr 0.001 --source_noise_model impulse,0,95 --target_noise_model clean --val_noise_model impulse,70,70 --loss l0 --output_path impulse_clean
```

**Model architectures**

With `--model unet`, UNet model can be trained instead of SRResNet.

**Resume training**

With `--weight path/to/weight/file`, training can be resumed with trained weights.

The detailed options are:

```
optional arguments:
  -h, --help            show this help message and exit
  --image_dir IMAGE_DIR
                        test image dir (default: None)
  --model MODEL         model architecture ('srresnet' or 'unet') (default:
                        srresnet)
  --weight_file WEIGHT_FILE
                        trained weight file (default: None)
  --test_noise_model TEST_NOISE_MODEL
                        noise model for test images (default: gaussian,25,25)
  --output_dir OUTPUT_DIR
                        if set, save resulting images otherwise show result
                        using imshow (default: None)
```

This script adds noise using `test_noise_model` to each image in `image_dir` and performs denoising. If you want to perform denoising to already noisy images, use `--test_noise_model clean`.

### Noise Models

Using `source_noise_model`, `target_noise_model`, and `val_noise_model` arguments, arbitrary noise models can be set for source images, target images, and validatoin images respectively. Default values are taken from the experiment in [1].

- Gaussian noise
  - gaussian,min_stddev,max_stddev (e.g. gaussian,0,50)
- Clean target
  - clean
- Text insertion
  - text,min_occupancy,max_occupancy (e.g. text,0,50)
- Random-valued impulse noise
  - impulse,min_occupancy,max_occupancy (e.g. impulse,0,50)

You can see how these noise models work by:

```
python3 noise_model.py --noise_model text,0,95
```

### Results

#### Plot training history

```
python3 plot_history.py --input1 gaussian --input2 clean
```

#### Gaussian noise

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/20/DQy9zj.png" width="60%" height="60%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Gaussian noise</div>
</center>
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/20/DQcFaV.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Denoising result by clean target model (left to right: original, degraded image, denoised image)</div>
</center>

From the above result, I confirm that we can train denoising model using noisy targets but it is not comparable to the model trained using clean targets. If UNet is used, the result becomes 29.67 (noisy targets) vs. 30.14 (clean targets).

#### Text insertion

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/20/DQyDmt.png" width="60%" height="60%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Text insertion</div>
</center>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/20/DQRpKH.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Denoising result by clean target model</div>
</center>



#### Random-valued impulse noise

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/20/DQyxn1.png" width="60%" height="60%">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;"> Random-valued impulse noise</div>
</center>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/20/DQRZRS.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Denoising result by clean target model</div>
</center>

#### Check denoising result

```
python3 test_model.py --weight_file [trained_model_path] --image_dir dataset/Set14
```

## De-watermarking

Once you have the model, you can use it to remove the watermark. We use the following command:

````
python test_model.py --weight_file watermark-model-file-names.hdf5  --image_dir inputdir --output_dir outputdir
````

Replace the *watermark-model-file-names.hdf5* with the actual file name, put the watermarked image in the *inputdir* and execute the command. After removing the watermark the picture will lie quietly in the *outputdir* directory.

**Watermarking effect:** Here is the result of 9 hours of training on the 1050ti, which may be a little unclean, but in theory a basic level of usability can be reached after 20 hours of training (original image on the left, watermarking image on the right). 

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/20/DQozDS.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">de-watermark sample</div>
</center>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/20/DQTCNj.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">de-watermark sample</div>
</center>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@main/blog/2020/11/20/DQTk3q.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">de-watermark sample</div>
</center>

# References

[1] J. Lehtinen, J. Munkberg, J. Hasselgren, S. Laine, T. Karras, M. Aittala, T. Aila, "Noise2Noise: Learning Image Restoration without Clean Data," in Proc. of ICML, 2018.

[2] J. Kim, J. K. Lee, and K. M. Lee, "Accurate Image Super-Resolution Using Very Deep Convolutional Networks," in Proc. of CVPR, 2016.

[3] X.-J. Mao, C. Shen, and Y.-B. Yang, "Image Restoration Using Convolutional Auto-Encoders with Symmetric Skip Connections," in Proc. of NIPS, 2016.

[4] C. Ledig, et al., "Photo-Realistic Single Image Super-Resolution Using a Generative Adversarial Network," in Proc. of CVPR, 2017.

[5] O. Ronneberger, P. Fischer, and T. Brox, "U-Net: Convolutional Networks for Biomedical Image Segmentation," in MICCAI, 2015.