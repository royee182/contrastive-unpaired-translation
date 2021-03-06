

# Contrastive Unpaired Translation (CUT)

### [video](https://youtu.be/Llg0vE_MVgk) | [website](http://taesung.me/ContrastiveUnpairedTranslation/) |   [paper](https://arxiv.org/pdf/2007.15651)
<br>

<img src='imgs/gif_cut.gif' align="right" width=960>

<br><br><br>



We provide our PyTorch implementation of unpaired image-to-image translation based on patchwise contrastive learning and adversarial learning.  No hand-crafted loss and inverse network is used. Compared to [CycleGAN](https://github.com/junyanz/CycleGAN), our model training is faster and less memory-intensive. In addition, our method can be extended to single image training, where each “domain” is only a *single* image.




[Contrastive Learning for Unpaired Image-to-Image Translation](http://taesung.me/ContrastiveUnpairedTranslation/)  
 [Taesung Park](https://taesung.me/), [Alexei A. Efros](https://people.eecs.berkeley.edu/~efros/), [Richard Zhang](https://richzhang.github.io/), [Jun-Yan Zhu](https://www.cs.cmu.edu/~junyanz/)<br>
UC Berkeley and Adobe Research<br>
 In ECCV 2020


<img src='imgs/patchnce.gif' align="right" width=960>

<br><br><br>

### Pseudo code
```python
import torch
cross_entropy_loss = torch.nn.CrossEntropyLoss()

# Input: f_q (BxCxS) and sampled features from H(G_enc(x))
# Input: f_k (BxCxS) are sampled features from H(G_enc(G(x))
# Input: tau is the temperature used in PatchNCE loss.
# Output: PatchNCE loss
def PatchNCELoss(f_q, f_k, tau=0.07):
    # batch size, channel size, and number of sample locations
    B, C, S = f_q.shape

    # calculate v * v+: BxSx1
    l_pos = (f_k * f_q).sum(dim=1)[:, :, None]

    # calculate v * v-: BxSxS
    l_neg = torch.bmm(f_q.transpose(1, 2), f_k)

    # The diagonal entries are not negatives. Remove them.
    identity_matrix = torch.eye(S)[None, :, :]
    l_neg.masked_fill_(identity_matrix, -float('inf'))

    # calculate logits: (B)x(S)x(S+1)
    logits = torch.cat((l_pos, l_neg), dim=2) / tau

    # return PatchNCE loss
    predictions = logits.flatten(0, 1)
    targets = torch.zeros(B * S, dtype=torch.long)
    return cross_entropy_loss(predictions, targets)
```
## Example Results

### Unpaired Image-to-Image Translation
<img src="imgs/results.gif" width="800px"/>

### Single Image Unpaired Translation
<img src="imgs/singleimage.gif" width="800px"/>


### Russian Blue Cat to Grumpy Cat
<img src="imgs/grumpycat.jpg" width="800px"/>

### Parisian Street to Burano's painted houses
<img src="imgs/paris.jpg" width="800px"/>



## Prerequisites
- Linux or macOS
- Python 3
- CPU or NVIDIA GPU + CUDA CuDNN

### Getting started

- Clone this repo:
```bash
git clone https://github.com/taesungp/contrastive-unpaired-translation CUT
cd CUT
```

- Install PyTorch 1.1 and other dependencies (e.g., torchvision, visdom, dominate, gputil).

  For pip users, please type the command `pip install -r requirements.txt`.

  For Conda users,  you can create a new Conda environment using `conda env create -f environment.yml`.


### CUT and FastCUT Training and Test

- Download the `grumpifycat` dataset (Fig 8 of the paper. Russian Blue -> Grumpy Cats)
```bash
bash ./datasets/download_cut_dataset.sh grumpifycat
```
The dataset is downloaded and unzipped at `./datasets/grumpifycat/`.

- To view training results and loss plots, run `python -m visdom.server` and click the URL http://localhost:8097.

- Train the CUT model:
```bash
python train.py --dataroot ./datasets/grumpifycat --name grumpycat_CUT --CUT_mode CUT
```
 Or train the FastCUT model
 ```bash
python train.py --dataroot ./datasets/grumpifycat --name grumpycat_FastCUT --CUT_mode FastCUT
```
The checkpoints will be stored at `./checkpoints/grumpycat_*/web`.

- Test the CUT model:
```bash
python test.py --dataroot ./datasets/grumpifycat --name grumpycat_CUT --CUT_mode CUT --phase train
```

The test results will be saved to a html file here: `./results/grumpifycat/latest_test/index.html`.

### CUT, FastCUT, and CycleGAN
<img src="imgs/horse2zebra_comparison.jpg" width="800px"/><br>

CUT is trained with the identity preservation loss and with `lambda_NCE=1`, while FastCUT is trained without the identity loss but with higher `lambda_NCE=10.0`. Compared to CycleGAN, CUT learns to perform more powerful distribution matching, while FastCUT is designed as a lighter (half the GPU memory, can fit a larger image), and faster (twice faster to train) alternative to CycleGAN. Please refer to the [paper](https://arxiv.org/abs/2007.15651) for more details.

In the above figure, we measure the percentage of pixels belonging to the horse/zebra bodies, using a pre-trained semantic segmentation model. We find a distribution mismatch between sizes of horses and zebras images -- zebras usually appear larger (36.8\% vs. 17.9\%). Our full method CUT has the flexibility to enlarge the horses, as a means of better matching of the training statistics than CycleGAN. FastCUT behaves more conservatively like CycleGAN.

### Training using our launcher scripts

Please see `experiments/grumpifycat_launcher.py` that generates the above command line arguments. The launcher scripts are useful for configuring rather complicated command-line arguments of training and testing.

Using the launcher, the command below generates the training command of CUT and FastCUT.
```bash
python -m experiments grumpifycat train 0   # CUT
python -m experiments grumpifycat train 1   # FastCUT
```

To test using the launcher,
```bash
python -m experiments grumpifycat test 0   # CUT
python -m experiments grumpifycat test 1   # FastCUT
```

Possible commands are run, run_test, launch, close, and so on. Please see `experiments/__main__.py` for all commands



### Apply a pre-trained CUT model and evaluate

The tutorial for using pretrained models will be released soon.

### SinCUT Single Image Unpaired Training

The tutorial for the Single-Image Translation will be released soon.



### Citation
If you use this code for your research, please cite our [paper](https://arxiv.org/pdf/2007.15651).
```
@inproceedings{park2020cut,
  title={Contrastive Learning for Unpaired Image-to-Image Translation},
  author={Taesung Park and Alexei A. Efros and Richard Zhang and Jun-Yan Zhu},
  booktitle={European Conference on Computer Vision},
  year={2020}
}
```

If you use the original [pix2pix](https://phillipi.github.io/pix2pix/) and [CycleGAN](https://junyanz.github.io/CycleGAN/) model included in this repo, please cite the following papers
```
@inproceedings{CycleGAN2017,
  title={Unpaired Image-to-Image Translation using Cycle-Consistent Adversarial Networkss},
  author={Zhu, Jun-Yan and Park, Taesung and Isola, Phillip and Efros, Alexei A},
  booktitle={Computer Vision (ICCV), 2017 IEEE International Conference on},
  year={2017}
}


@inproceedings{isola2017image,
  title={Image-to-Image Translation with Conditional Adversarial Networks},
  author={Isola, Phillip and Zhu, Jun-Yan and Zhou, Tinghui and Efros, Alexei A},
  booktitle={Computer Vision and Pattern Recognition (CVPR), 2017 IEEE Conference on},
  year={2017}
}
```


### Acknowledgments
We thank Allan Jabri and Phillip Isola for helpful discussion and feedback. Our code is developed based on [pytorch-CycleGAN-and-pix2pix](https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix). We also thank [pytorch-fid](https://github.com/mseitzer/pytorch-fid) for FID computation,  [drn](https://github.com/fyu/drn) for mIoU computation, and [stylegan2-pytorch](https://github.com/rosinality/stylegan2-pytorch/) for the PyTorch implementation of StyleGAN2 used in our single-image translation setting.
