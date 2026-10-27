# Photo Restoration: Deep Learning vs. Classical Inpainting

## Motivation

Genealogy is my hobby, and I keep running into old photos that are damaged: scratches, stains, folds, missing corners. Some show family members, others show places. I'd like to build a tool that helps me repair them, and this project is a first step in that direction.

I won't use or publish any of my own family photos. For training and testing I use public datasets, and the demo only shows freely licensed images.

## 1. Idea and approach

The task is image inpainting. The model gets a damaged photo and a mask that marks the broken pixels, and it has to fill them in so the result looks like the original. To get training data I damage clean photos myself, so the original is always there to compare with and I don't have to label anything by hand.

My plan:

1. Write a damage generator that creates masks for different kinds of damage and different severities.
2. Run classical inpainting methods as baselines: Telea [1] and Navier-Stokes [2] from OpenCV, plus one patch-based method [3, 4] from an existing PatchMatch-based implementation.
3. Train a U-Net as the deep learning model. According to my research, it's a standard image-to-image network that takes the damaged image and the mask and outputs the repaired image. I'll start there and decide what to improve (loss function, mask-aware layers, a pretrained model) based on the results and the hardware I have.
4. Compare everything on the same damaged test images with the same metrics, split by damage type and severity, not just one average number.

My guess is that the classical methods do fine on thin scratches and fall apart on big missing areas, because there the content has to be invented.

**Topic:** restoring damaged photos by image inpainting

**Project type:** "Beat the classics"

**Metrics:** PSNR (Peak Signal-to-Noise Ratio), SSIM (Structural Similarity Index) and LPIPS (Learned Perceptual Image Patch Similarity). I compute each of them on the whole image and also only inside the damaged region, because most pixels of a damaged image are untouched and would hide a bad fill. The numbers for the damaged region are the main ones. I chose the above metrics because they seem to be present in most (if not all) of the papers so it is a good way to compare different approaches.

**If there's time left:** grayscale/sepia photos, tests on real scratched photos, and finding the damage automatically instead of getting the mask.

## 2. Dataset

The data is clean photos with masks applied on top.

**Photos.** The main source is DIV2K (800 training, 100 test and 100 validation images, about 2K resolution, https://data.vision.ee.ethz.ch/cvl/DIV2K/), cut into 256×256 crops. Optionally I'll add a small subset of ImageNet (https://www.image-net.org/index.php) as unseen test photos, to see how well the models generalise. I assign each source image to train, validation or test before cropping, so crops of the same photo never end up in two splits.

**Masks.** There are two sources:

- The NVIDIA Irregular Mask Dataset (https://research.nvidia.com/labs/adlr/publication/partialconv-inpainting/ and https://github.com/NVIDIA/partialconv), introduced in [5]. It has about 12,000 masks, grouped by how much of the image they cover. It's widely used, so my results can be compared with other papers.
- My own generator, because those masks look like blobs and not like real damage. It creates thin scratches, dust spots, stains, folds/tears and large missing regions at four severity levels, defined by how much of the image is damaged:
  - light (about 1-5%)
  - medium (about 5-15%)
  - heavy (about 15-30%)
  - extreme (about 30-50%).
    The levels are based on the kind of damage I've seen on old photos in genealogy, and I'll fix the exact bounds once the generator works. The extreme level is there to find the limit. With half of the image gone nobody can restore it faithfully, a model can only guess something plausible, so I treat it as a limit test and don't expect good results there.

The test masks are generated once with a fixed seed and saved, so all methods see exactly the same damage. The test generator uses slightly different settings than the training one (for example other scratch widths), so the evaluation isn't too optimistic.

**Size.** I'm not fixing numbers yet. I'll start with a small proof of concept (a few hundred crops) to check the generator, the baselines, the metrics and the training loop end to end, and then scale up as far as time and hardware allow.

## 3. Work breakdown

This follows the ECTS breakdown of the course (3 ECTS = 75 h).

| Task                                                                                                            | Hours  |
| --------------------------------------------------------------------------------------------------------------- | ------ |
| Dataset collection (DIV2K and NVIDIA masks, damage generator, test masks, splits)                               | 7      |
| Designing and building the network (reading papers, planning, U-Net, data loading, training loop, tests and CI) | 14     |
| Training and fine-tuning (U-Net and one improvement, running the baselines, evaluation)                         | 16     |
| Building the demo application                                                                                   | 8      |
| Writing the final report                                                                                        | 6      |
| Preparing the presentation                                                                                      | 4      |
| **Total**                                                                                                       | **55** |

The stretch goals are not in this estimate. If I start one, the time comes out of training and fine-tuning. I'll keep track of my actual hours and compare them with this table in the final report.

## 4. References

1. A. Telea, _An Image Inpainting Technique Based on the Fast Marching Method_, Journal of Graphics Tools 9(1), 23-34, 2004. DOI 10.1080/10867651.2004.10487596. https://www.researchgate.net/publication/238183352_An_Image_Inpainting_Technique_Based_on_the_Fast_Marching_Method
2. M. Bertalmío, A. L. Bertozzi, G. Sapiro, _Navier-Stokes, Fluid Dynamics, and Image and Video Inpainting_, CVPR 2001. https://ieeexplore.ieee.org/document/990497
3. A. Criminisi, P. Pérez, K. Toyama, _Region Filling and Object Removal by Exemplar-Based Image Inpainting_, IEEE Transactions on Image Processing, vol. 13, no. 9, pp. 1200-1212, Sept. 2004. https://ieeexplore.ieee.org/document/1323101
4. C. Barnes, E. Shechtman, A. Finkelstein, D. B. Goldman, _PatchMatch: A Randomized Correspondence Algorithm for Structural Image Editing_, ACM Transactions on Graphics (SIGGRAPH) 28(3), 2009. https://dl.acm.org/doi/10.1145/1531326.1531330
5. G. Liu, F. A. Reda, K. J. Shih, T.-C. Wang, A. Tao, B. Catanzaro, _Image Inpainting for Irregular Holes Using Partial Convolutions_, Proceedings of the European Conference on Computer Vision (ECCV) 2018 (introduces the NVIDIA mask dataset). https://arxiv.org/abs/1804.07723

## Licences

DIV2K is for academic research only and ImageNet for non-commercial research and education, which fits a course project. The repository contains no dataset images, only code and download instructions. The example photos in the demo will be CC-licensed or public domain.

## AI usage disclosure

While working on this README I used an AI assistant (Claude) for:

- **Research:** finding datasets and existing implementations, and looking up their licences and availability.
- **Idea sparring:** talking through project ideas and discussing what is and isn't feasible.
- **Wording:** helping me phrase the README so that it stays clear and easy to follow.

The ideas and decisions about the project are mine.
