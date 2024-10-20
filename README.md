# RSNA Lumbar Degeneration Classification Competition

## Competition Overview
The lumbar reigion of the spine is a major cause of pain and disability worldwide, infact it is the leading cause of disability with around 600 million people suffering from the varying conditions arising from degeneration in the lumbar discs. The goal of the competition is to use deep learning methods to accurately and reliably predict whether the patient has a condition and the level of severity of said condition. 

The RSNA provides a dataset of N studies (patients) MRI scans with three differing camera settings/positions, these being Sagittal T1, Sagittal T2/STIR and Axial T2.
A sagittal T1 image is taken from the sagittal plane of the body (i.e side on) and the T1 indicates that fat is more visible in the scan.

A sagittal T2/STIR image is also taken from the sagittal plane, however the T2/STIR indicates that water is more visible in the scan, hence revealing the spinal cord more.
SAG T2 EXAMPLE IMAGES
An axial T2 image is taken from the axial plane (i.e top down of the body).
AX T2 EXAMPLE IMAGES
They also provide a file with the coordinates of the degenerative disc along with the condition name as well as the severity of the damage to the disc. These include:
IMAGES OF THE DEGENERATION TYPES WITH COORDINATES

Note: I decided early on that the premise of the competition was very interesting and I would persue the project more long-term (finishing past the deadline) however, the project naturally came to an end close to the deadline - which led me to attempting to make a submission, only to have issues with the GPU quota on Kaggle not resetting when advertised. using limited data and compute power/time to achieve a goal. In biomedical image segmentation, there is often very limited labelled data due to the expert-level and time consuming nature of segmenting MRI scans. Which is why the premise stuck out to me since the competition rules emulate that of a real life situation.

## Results
The key information first - how did I do? My approach resulted in:
-> 0.xx accuracy in identifying right/left neural foraminal narrowing and whether it was a normal/mild, moderate or severe case.
-> 0.yy accuracy in identifying right/left subarticular stenosis and whether it was a normal/mild, moderate or severe case.
-> 0.zz accuracy in identifying right/left spinal canal stenosis and whether it was a normal/mild, moderate or severe case.
Unfortunatley, due to an issue with the weekly GPU quota not resetting when expected, I could not make my final submission within the deadline - so a late submission was made on 13-10-2024.
On Kaggle this approach would have placed me at (still issues with gpu quota persist 17/10/24)

### My Approach
The process of applying deep learning to a new problem is challenging. First of all, there are a wide array of model architectures to choose from - for image classification, convolutional neural networks (CNNs) are usually the preffered choice. Secondly the data must be processed in suitable way for the model to understand the images and what categories they fall into. Then there are many parameters and hyperparameters to consider, such as the model depth, i.e how many layers (and of which type) will work for the data and task at hand.

My first approaches to this problem consisted of 5-10 layered convolutional neural networks coded in Pytorch. This is was a learning moment for me, I do not have to produce all of my results from scratch. Taking the time to test and trial different model architectures I have come up with when many brilliant engineers have created models which have been fine-tuned and trained on millions of images exist for the purpose of furthering others' research. I put models such as the ResNet18, ResNet152, EfficientNetB0 and DenseNet169 to work on my existing data pipeline. However while I saw some initial improvement, the model still failed to achieve high levels of accuracy - something was missing.

Coming back to the learning moment from before, I started to read more and more scientific literature around deep learning in biomedical image classification and found why my models hadn't performed all too well - CNNs fail to capture 'local features' in an image, therefore lack the spatial understanding of the relationships between organs, bones and how damage to these features look in biomedical images. In 2015 a revolutionary paper presenting a new architecture was published which adressed the aforementioned issues of the CNN. This architecture is called the U-Net (named for its U shape) and I will briefly explain how it works.

IMAGE OF UNET ARCHITECTURE


### Limitations With The Provided Data
Now armed with a new approach to the problem, an issue remained - U-Net needs images which have every single pixel labelled with its type a process called segmentation. An example of this is this image of a road.
IMAGE OF THE ORIGINAL ROAD AND SEGMENTED ROAD

As seen, the image is coloured accordingly to what the pixel represents - .... . 

Now applying this to the images of the spine - here the original image and the fully segmented image of the spine:
<img src="https://github.com/user-attachments/assets/e7282fb6-b80a-4467-8e96-7b850a678631" style="filter: grayscale(100%);"> <img src="https://github.com/user-attachments/assets/518cf58e-e088-4f62-8147-3574d08aaf2b" style="filter: grayscale(100%);">

The competition dictates that you may not use the internet to access anything external while running the submission notebook, however there is no rule on using publically avaliable data before the submission notebook is run on the Kaggle website. With that in mind, I was inspired by Kaggle user "" and their notebook "" which used a U-Net architecture with a ResNet backbone to train a model to label the vertebrae in an image of the spine, this used data from the Spider dataset. Here is what the evolution of the model through its training looked like:

IMAGE 1-15 OF TRAINING EPOCH PREDICTIONS

Now when tested on one of the MRI scans provided in the competition:

IMAGE OF SEGMENTED SPINE RSNA.

And we observe that it did a pretty good job! This is ample to move on to the next stage of segmenting the image, which is segmenting the discs in the image. My approach was to do this using the coordinates provided in the competition to make an educated guess to where the disc should be. Here's some examples of what I managed to generate for all N MRI scans.

### nnU-Net Model
Now, my original idea was to now use a second U-Net similar to the one used to segment the vertebrae and then use a CNN to classify the conditions and severities, however there is a very clever pre-built model by "" called the nnU-Net, which ... It is made for people with no programming experience and configures all parameters and hyperparameters for the user, due to its ease of use, I highly reccomend this as a base model in other biomedical image classification tasks.

IMAGE OF NNUNET ARCHITECTURE

Description of the nnunet architecture...

This yeilded impressive segmentation masks of the provided images, however there was an issue. During training this message shows: Pseudo dice [0.7606,.., 0.0, nan, nan, nan, 0.0, 0.0, 0.4707, 0.4522, 0.4519, 0.3517,,,0, nan, nan] which gives the accuracy of the model in predicting each class in the segmentation mask, most have been removed here but the closer the value to 1, the better the model is in predicting that class, however there were nan values, which indicated that there were missing samples, so for example the model will never see an MRI image with severe l4 l5 right neural foraminal narrowing, so it cannot then go on to predict if an image contains this condition or not. This required me to go back and ensure that all classes were equally represented.


### Limitations In External Data
There are few papers on the segmentation of the Axial T2 images to predict conditions in the lumbar region and their datasets are all request-only. To my knowledge there are no easily accessible segmentations of Axial T2 MRI scans. Due to this as well as the encroaching deadline, my approach to this was to create an extremely primitive segmentation mask which just segmented a 10x10 box around the coordinates given in the RSNA data, then running this segmentation mask and the original image through the nnU-Net.






























