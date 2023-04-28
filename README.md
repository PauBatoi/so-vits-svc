# SoftVC VITS Singing Voice Conversion

## Notice
* This fork has some modifications to make it work better on Windows and with smaller multi-speaker datasets.
* Some modifications have also been made to pitch inference for better performance.
* There is one gui using PySide6 `inference_gui.py` and one gui using PyQt5 currently a work in progress `inference_gui2.py`
	* Inference GUI 2 features experimental TalkNet integration, in-program recording, as well as other features like timestretching with rubberband. Instructions can be found below under `Inference GUI 2` header.
	* Also check out GothicAnon's GUI, written in tkinter. See [here](https://docs.google.com/document/d/1PDkSrKKiHzzpUTKzBldZeKngvjeBUjyTtGCOv2GWwa0/edit#heading=h.l2lv04nvagvx), under So-Vits-SVC for more info.

+ The current branch is the 32kHz version, which requires less vram during inferencing, as well as faster inferencing speeds, and datasets for said branch take up less disk space. Thus the 32 kHz branch is recommended for use.
+ If you want to train 48 kHz variant models, switch to the [main branch](https://github.com/innnky/so-vits-svc/tree/main). NOTE: effusiveperiscope does not maintain a 48khz branch.
## Colab notebook scripts

[Colab training notebook (EN)](https://colab.research.google.com/drive/1ABKgdAWp2UGfgavyxI_Nv7ytdL5h6J6f?usp=sharing)

Note that the following notebooks are not maintained by me.

[Colab training notebook (CN)](https://colab.research.google.com/drive/1rCUOOVG7-XQlVZuWRAj5IpGrMM8t07pE?usp=sharing)

[Colab inference notebook](https://colab.research.google.com/drive/1BaU09Xkhe6MEHxCC9URDYZ2ERajfJ_Fa#scrollTo=oFr2MWaQfR6X)

# Inference
Instructions for CLI inference can be found in the original repository or [here](https://docs.google.com/document/d/1y1pfS0LCrwbbvxdn3ZksH25BKaf0LaO13uYppxIQnac/edit#heading=h.ce7da4l3o6jf). Note that CLI inference is very clunky and generally not recommended for production. Instructions for inference using various GUIs are available below.

## Docker
An anon has provided docker files with instructions [here.](https://files.catbox.moe/6ct61q.zip)

## Required downloads
+ Download softVC hubert model:[hubert-soft-0d54a1f4.pt](https://github.com/bshall/hubert/releases/download/v0.1/hubert-soft-0d54a1f4.pt)
  + Place under `hubert`.
+ Download pretrained models [G_0.pth](https://huggingface.co/datasets/HazySkies/SV3/resolve/main/G_0.pth) and [D_0.pth](https://huggingface.co/datasets/HazySkies/SV3/resolve/main/D_0.pth)
  + Place under `logs/32k`.
  + Pretrained models are required, because from experiments, training from scratch can be rather unpredictable to say the least, and training with a pretrained model can greatly improve training speeds.
  + The pretrained model includes云灏, 即霜, 辉宇·星AI, 派蒙, and 绫地宁宁, covering the common ranges of both male and female voices, and so it can be seen as a rather universal pretrained model.
  + The pretrained model exludes the `optimizer speaker_embedding` section, rendering it only usable for pretraining and incapable of inferencing with.

The following shell commands can be used for downloading with bash.
```shell
# For simple downloading.
# hubert
wget -P hubert/ https://github.com/bshall/hubert/releases/download/v0.1/hubert-soft-0d54a1f4.pt
# G&D pretrained models
wget -P logs/32k/ https://huggingface.co/datasets/HazySkies/SV3/resolve/main/G_0.pth
wget -P logs/32k/ https://huggingface.co/datasets/HazySkies/SV3/resolve/main/D_0.pth

```

## Inference GUI 2
For Inference GUI 2, you need to `pip install PyQt5`. Additional features may be available based on other dependencies:
* OPTIONAL - You PROBABLY DO NOT NEED THIS: For timestretching support, you need to install BOTH [the rubberband standalone program](https://breakfastquay.com/rubberband/), ensuring the rubberband executable is on your PATH, and the python module `pip install pyrubberband`. __Note that installing pyrubberband installs PySoundFile which needs to be uninstalled, and SoundFile will need to be reinstalled.__
* OPTIONAL - For TalkNet support, you need to `pip install requests` and also install this [ControllableTalkNet fork](https://github.com/effusiveperiscope/ControllableTalkNet).

### Basic Usage 

Models should be placed in separate folders within a folder called `models`, in the same directory as `inference_gui2.py` by default. Specifically, the file structure should be:
```
so-vits-svc-eff\
	models\
		TwilightSparkle
			G_*****.pth
			D_*****.pth
			config.json
```
If the proper libraries are installed, the GUI can be run simply by running `inference_gui2.py`. If everything goes well you should see something like this:
![](https://raw.githubusercontent.com/effusiveperiscope/so-vits-svc/eff/docs/gui.png)

All basic workflow occurs under the leftmost UI panel.

1. Select a speaker based on the listed names under `Speaker:`.
2. Drag and drop reference audio files to be converted onto `Files to Convert`. Alternatively, click on `Files to Convert` to open a file dialog.
3. Set desired transpose (for m2f vocal conversion this is usually 12 i.e. an octave, or leave it 0 if the reference audio is female) under `Transpose`.
4. Click `Convert`. The resulting file should appear under `results`.

The right UI panel allows for recording audio directly into the GUI for quick fixes and tests. Simply select the proper audio device and click `Record` to begin recording. Recordings will automatically be saved to a `recordings` folder. The resulting recording can be transferred to the so-vits-svc panel by pressing `Push last output to so-vits-svc`.

### Common issues
* When converting: `TypeError: Invalid file: WindowsPath('...')` Ensure that PySoundFile is not installed (`pip show pysoundfile`). PySoundFile is a deprecated version of SoundFile. After uninstalling pysoundfile, run `pip install soundfile==0.10.3.post1 --force-reinstall`
* When trying to run with TalkNet: `Couldn't parse TalkNet response.` Ensure that you are running `alt_server.py` in the TalkNet fork.

### Running with TalkNet 

For TalkNet support, you need to `pip install requests` and also install this [ControllableTalkNet fork](https://github.com/effusiveperiscope/ControllableTalkNet). Instead of running `talknet_offline.py`, run `alt_server.py` (if you use a batch script or conda environment to run TalkNet, you should use it to run `alt_server.py`). This will start a server that can interface with Inference GUI 2. The TalkNet server should be started before Inference GUI 2.

Next, starting Inference GUI 2 should show a UI like this:
![](https://raw.githubusercontent.com/effusiveperiscope/so-vits-svc/eff/docs/gui2.png)

The rightmost panel shows controls for TalkNet which are similar to those used in the web interface. Some items special to this interface:
* There is currently no "Custom model" option. To add additional models you should modify the model jsons in Controllable TalkNet.
* Recordings can be also be transferred from the recording panel to the TalkNet panel.
* Files can be provided under `Provide input audio` through clicking for a file dialog or drag-and-drop.
* In order to push output from TalkNet through so-vits-svc, check `Push TalkNet output to so-vits-svc`. For production work it is advised to first try one generation without this box checked to see if there are artifacts in the TalkNet output. The output will use the speaker selected in the leftmost panel.

## Updates
> According to incomplete statistics, it seems that training with multiple speakers may lead to **worsened leaking of voice timbre**. It is not recommended to train models with more than 5 speakers. The current suggestion is to try to train models with only a single speaker if you want to achieve a voice timbre that is more similar to the target.
> Fixed the issue with unwanted staccato, improving audio quality by a decent amount.\
> The 2.0 version has been moved to the 2.0 branch.\
> Version 3.0 uses the code structure of FreeVC, which isn't compatible with older versions.\
> Compared to [DiffSVC](https://github.com/prophesier/diff-svc) , diffsvc performs much better when the training data is of extremely high quality, but this repository may perform better on datasets with lower quality. Additionally, this repository is much faster in terms of inference speed compared to diffsvc.
# Training

## Dataset preparation
All that is required is that the data be put under the `dataset_raw` folder in the structure format provided below.
```shell
dataset_raw
├───speaker0
│   ├───xxx1-xxx1.wav
│   ├───...
│   └───Lxx-0xx8.wav
└───speaker1
    ├───xx2-0xxx2.wav
    ├───...
    └───xxx7-xxx007.wav
```

## Data pre-processing.
1. Resample to 32khz

```shell
python resample.py
 ```
2. Automatically sort out training set, validation set, test set, and automatically generate configuration files.
```shell
python preprocess_flist_config.py
# Notice.
# The n_speakers value in the config will be set automatically according to the amount of speakers in the dataset.
# To reserve space for additionally added speakers in the dataset, the n_speakers value will be be set to twice the actual amount.
# If you want even more space for adding more data, you can edit the n_speakers value in the config after runing this step.
# This can not be changed after training starts.
```
3. Generate hubert and F0 features/
```shell
python preprocess_hubert_f0.py
```
After running the step above, the `dataset` folder will contain all the pre-processed data, you can delete the `dataset_raw` folder after that.

## Training.
```shell
python train.py -c configs/config.json -m 32k
```

## Inferencing.
Use [inference_main.py](inference_main.py)
+ Edit `model_path` to your newest checkpoint.
+ Place the input audio under the `raw` folder.
+ Change `clean_names` to the output file name.
+ Use `trans` to edit the pitch shifting amount (semitones). 
+ Change `spk_list` to the speaker name.
