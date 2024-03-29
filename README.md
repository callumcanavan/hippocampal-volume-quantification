# hippocampal-volume-quantification

Code for training a 3D image segmentation algorithm and integrating it into a PACS system for calculating hippocampal volumes in cropped brain scans.
The architecture for the system can be seen below. A DICOM file is checked to see if it has been cropped around the hippocampus through HippoCrop, and if so it is divided into slices along the axial dimension and padded with zeros in the sagittal and coronal planes to form 64x64 slices. This tensor is then passed through a segmentation model, and a DICOM report is created and sent back to the viewer with segmentation results and volume calculations. A mock validation plan can be found in Validation Plan.pdf.

<img src="https://github.com/callumcanavan/hippocampal-volume-quantification/blob/main/images/architecture.png" alt="drawing" width="1000"/>

For segmentation, the slices obtained above are forward-propagated in parallel through U-Net [1](https://arxiv.org/pdf/1505.04597.pdf) with 3 output classes. The argmax of the 3x64x64 tensor thus obtained is taken along its first axis, resulting in a 64x64 mask of class predictions (0 for background, 1 for anterior hippocampus and 2 for posterior hippocampus). The sum of voxels for the latter two classes is calculated over each volume of masks to obtain the anterior and posterior hippocampus volumes, respectively, and added together to obtain the total predicted hippocampus volume in units of voxel volume (in the dataset used here this was 1mm<sup>3</sup>). 
The training dataset consists of 260 volumes from the “Hippocampus” dataset in the Medical Decathlon competition [2](https://arxiv.org/pdf/1902.09063.pdf). Each T2 MRI scan found in this dataset was performed on an adult patient (including both healthy adults and adults with non-affective psychotic disorders). Each image has been cropped to a small area around the hippocampus by radiologists prior to training. Ground truth segmentation was performed by experts using three-dimensional software that allows simultaneous analysis of sagittal, coronal and axis images [3](https://www.researchgate.net/publication/12547862_Volumetry_of_Hippocampus_and_Amygdala_with_High-resolution_MRI_and_Three-dimensional_Analysis_Software_Minimizing_the_Discrepancies_between_Laboratories). Outlier volumes were also removed prior to training (those with total volume greater than 300cm<sup>3</sup> and those without matching labels).

The dataset was randomly partitioned into training, validation and test sets (60%, 20% and 20% of the whole dataset, respectively). Performance of the model during training was measured in terms of categorical cross entropy loss, evaluated on both the training and validation sets. The former decreased from over 0.7 to around 0.011 over around 10 epochs, while the latter decreased from around 0.016 to 0.014. Performance of the algorithm in the real world is estimated here using four metrics (Jaccard index, Sorensen-Dice coefficient, sensitivity and specificity), as evaluated on the hold-out test set. For the purposes of calculating these metrics, the background was taken to be the negative class and all non-background labels (i.e. both anterior and posterior hippocampus labels) were taken to be the positive class. The means of Jaccard and Dice scores on the test set were 0.816 and 0.898, respectively. This implies an over 80% overlap between predicted and ground truth values on average. The mean sensitivity was found to be 0.907 while the mean specificity was found to be 0.997, implying that hippocampus volume underestimation is more likely than overestimation. 

Below is a TensorBoard visualisation showing the progress of loss during training.

<img src="https://github.com/callumcanavan/hippocampal-volume-quantification/blob/main/images/loss.png" alt="drawing" width="700"/>

[1] [U-Net: Convolutional Networks for Biomedical Image Segmentation, Ronneberger et al., 2015]( https://arxiv.org/pdf/1505.04597.pdf)

[2] [A large annotated medical image dataset for the development and evaluation of segmentation algorithms, Simpson et al., 2019](https://arxiv.org/pdf/1902.09063.pdf)

[3] [Volumetry of Hippocampus and Amygdala with High-resolution MRI and Three-dimensional Analysis Software: Minimizing the Discrepancies between Laboratories, Pruessner et al., 2000]( https://www.researchgate.net/publication/12547862_Volumetry_of_Hippocampus_and_Amygdala_with_High-resolution_MRI_and_Three-dimensional_Analysis_Software_Minimizing_the_Discrepancies_between_Laboratories)

# Depends
```
torch scikit-learn numpy
```
