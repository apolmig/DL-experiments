# DL-experiments
deep learning experiments

Classifying White Blood Cells With Deep Learning (Code and data included!)
You can follow all the code and recreate the results of this post here.
Introduction
What would healthcare look like if you could discover your blood cell count as quickly and cheaply as your temperature? At Athelas, we are fascinated by this possibility and believe that such a future is within sight thanks to modern Deep Learning techniques.
In this post, we’ll demonstrate a simple toy example of how we can leverage deep learning techniques to classify white blood cell images.
Our example model will classify white blood cells as Polynuclear or Mononuclear with an accuracy of 98% on our reference dataset.

Polynuclear cells have nuclei that are segmented into multiple pieces while mononuclear cells have nuclei that are single pieces.
Caveats
There are a few caveats worth mentioning before going too much further into this post! Here they are:
This is not a rigorous, peer-reviewed paper that proves the accuracy of such techniques on a variety of datasets and lighting conditions. This post is simply meant to showcase how effective CNNs can be on traditional problems in pathology.
The code and techniques mentioned in this post are not used by us in production. Our production code is tested and benchmarked to meet FDA standards. Moreover, in production we provide a range of metrics on the Complete Blood Count (CBC). The problem in this post represents a small subset of this overall problem space.
With that in mind, let’s take a look at why this is a fascinating problem space in the first place.
White Blood Cells and Why They’re Important

A classic blood test.
If you’ve ever had your blood drawn, chances are that you’ve had a blood test where your doctor looked up the following statistics about your blood among others:
The total number of White Blood Cells (WBCs) in your blood stream.
The number of Neutrophils, Lymphocytes, Basophils, and Eosinophils (all types of WBCs) in your cell. This is known as a differentiated blood cell count.
The density of WBCs in our blood stream provides a glimpse into the state of our immune system and any potential risks we might be facing. In particular, a dramatic change in the WBC count relative to your baseline is generally a sign that your body is currently being affected by an antigen. Moreover, a variation in a specific type of WBC generally correlates with a specific type of antigen.

Leukemia patients see far more Lymphocytes in their blood. Image Source: MedGurus.org
For instance, Leukemia patients often see a dramatically higher level of Lymphocytes in their blood stream relative due to a malfunctioning immune system. Likewise, people fighting allergies generally see an increase in their Eosinophil counts as these WBCs are key to fighting allergens.
As such, understanding the count of White Blood Cells in our blood stream can provide us a powerful quantitative picture of our health.
How We Currently Count WBCs
There are two main ways we count the different types of WBCs in your blood stream: the automated way and the manual way.
The Automated Way: Coulter Counters and Laser Flow Cytometry
The earliest automated way uses a device invented in the 1950s known as a Coulter Counter.

A Coulter Counter counts and classifies blood cells by passing them through a channel and measuring the change in impedance. Image source: Wikipedia.
At a high level, a Coulter Counter uses the fact that WBCs are poor conductors of electricity to classify and count them. In particular, the counter has two baths of a conductive solution along with a small channel through which a current runs. As the cell passes through the channel, the current through the channel decreases proportional to the shape and size of the type of cell. By measuring this change, the machine classifies the type of cell.

Laser Flow Cytometers. Image source: Wikipedia.
More recently, Laser Flow Cytometers have emerged as an improved alternative to Coulter Counters. These devices shine a laser across a channel and measure how the light from the laser is refracted as cells pass through (see the image above). Based on the refraction of light, the device aims to classify the given cell. At a high level, these devices improve on Coulter Counters by using light instead of current as the medium to measure disturbances against.
Both Coulter Counters and Laser Flow Cytometers cost in the tens of thousands of dollars. However, there’s no doubt that have played a fundamental role in improving our quantitative understanding of our health.
The Manual Way: Inspecting a Sample Under a Microscope

A pathologist examining a blood sample manually. Image source: wisegeek.com
The manual approach is just as it sounds - a sample of blood is placed under a microscope and a pathologist manually counts the number of cells in each frame. The total count is then extrapolated by assuming that the distribution is uniform across the entire blood sample and multiplying up.
The ML Way: Models That Improves with Time
The Machine Learning approach that we will be exploring through the rest of this blog post is a potentially promising advancement over such techniques due to a few reasons:
It requires far cheaper equipment thanks to being reliant on simple imaging.
It provides results nearly instantaneously unlike the above methods.
Like all Machine Learning, it promises to get better over time as we classify and count more and more blood cells and increase our dataset sizes. Moreover, given that it is software based, we can continuously update it over the air and provide consumers an experience that continually improve.
A Deep Learning Based Solution
Now that we have the necessary background, let’s jump into our specific problem and analyze the dataset, methodology, and results of our classifier.
Problem: Given a stained image of a white blood cell, classify it as either polynuclear or mononuclear. Note that Eosinophils and Neutrophils are polynuclear while Lymphocytes and Monocytes are mononuclear.
Dataset

Examples of images in our dataset.
Our dataset is composed of 352 images of dyed WBCs. Each image is of size 640 x 480. Below are the counts and distributions of the cells:
21 Monocyte (Mononuclear)
33 Lymphocyte (Mononuclear)
207 Neutrophil (Polynuclear)
88 Eosinophil (Polynuclear)
3 Basophil (Mononuclear)

Distribution of the blood cells in our dataset. As can be seen, our dataset is heavily biased towards Neutrophils.
The images have been hand labeled by a Pathologist and were collected from an existing dataset. They were augmented with images we took with our own microscope and later dyed.
Cleaning and Preprocessing the Dataset
As with all good Machine Learning, a large part of our focus was actually cleaning and preprocessing the dataset. Here are the specific steps we took to clean our dataset:

An example of an image with multiple cells. Such images were removed as part of the preprocessing.
Remove images with multiple cells (see the above image).
Remove Basophils (we had too few Basophils to generalize from).
Down-sample the images to 120 x 160 so that we can train faster.
Convert labels from Monocyte, Eosinophil, Neutrophil, or Lymphocyte to Polynuclear or Mononuclear. We essentially converted the categorical classification problem into a binary classification problem so that it would be more tractable given the size of the dataset.
Use image augmentation (flips, rotations, and shears) to increase the size of the training set and balance out the classes so that we had 2500 of each type of WBC.
Divide image arrays by 255 to bring the range of values between 0 and 1.
Image Augmentation

The original image is rotated, flipped, and sheared to generate new images.
Given that cells naturally don’t have an expected orientation, we were able to flip, rotate, and shear the images of our cells to dramatically increase our training set size. We increased the size of our training from 352 images to 10,000 images using this process. Moreover, we also used augmentation to make sure we have the same number of samples for each class of blood cell (2500) thereby balancing out the dataset.
Model

We used a simple LeNet architecture with a few modifications.
We chose to start with the simplest model that would work and then iterate. As such, LeNet was the natural starting point for our model.
We chose to increase the dropout to be 0.7 in the final fully connected layer to be more aggressive against overfitting. You can see our final architecture, with its modifications below:
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
lambda_2 (Lambda)            (None, 120, 160, 3)       0         
_________________________________________________________________
conv2d_4 (Conv2D)            (None, 118, 158, 32)      896       
_________________________________________________________________
activation_6 (Activation)    (None, 118, 158, 32)      0         
_________________________________________________________________
max_pooling2d_4 (MaxPooling2 (None, 59, 79, 32)        0         
_________________________________________________________________
conv2d_5 (Conv2D)            (None, 57, 77, 32)        9248      
_________________________________________________________________
activation_7 (Activation)    (None, 57, 77, 32)        0         
_________________________________________________________________
max_pooling2d_5 (MaxPooling2 (None, 28, 38, 32)        0         
_________________________________________________________________
conv2d_6 (Conv2D)            (None, 26, 36, 64)        18496     
_________________________________________________________________
activation_8 (Activation)    (None, 26, 36, 64)        0         
_________________________________________________________________
max_pooling2d_6 (MaxPooling2 (None, 13, 18, 64)        0         
_________________________________________________________________
flatten_2 (Flatten)          (None, 14976)             0         
_________________________________________________________________
dense_3 (Dense)              (None, 64)                958528    
_________________________________________________________________
activation_9 (Activation)    (None, 64)                0         
_________________________________________________________________
dropout_2 (Dropout)          (None, 64)                0         
_________________________________________________________________
dense_4 (Dense)              (None, 2)                 130       
_________________________________________________________________
activation_10 (Activation)   (None, 2)                 0         
=================================================================
Total params: 987,298.0
Trainable params: 987,298.0
Non-trainable params: 0.0
_________________________________________________________________
Results

The accuracy and loss of our model over 20 epochs. 20% of the dataset was held out for validation.
We tested our model against a test set of 71 images (20% of our dataset) drawn from the original data’s distribution (so it was not balanced). We trained for 20 epochs with a batch size of 32 on an Amazon P2 instance.
On our reference dataset, we obtained an accuracy of 98.6% for the classification task. Our Confusion Matrix can be seen below:

You can recreate all our results by following our notebook linked here.
Limitations
The above results should be seen with the following limitations in mind:
Our dataset had only a few examples of Mononuclear cells. It would be important to obtain a dataset with more such cell images to feel fully confident in the classifier. Moreover, most images are under the same lighting and microscopy conditions and it would be worth obtaining images under more varied conditions to prove generalizability.
The same model does not perform quite as well on the task of classifying the WBCs into one of Lymphocyte, Monocyte, Eosinophil, or Neutrophil (accuracy of 86%). Further modifications are required to obtain a confident classifier there.
This work is not yet peer reviewed although you can replicate all the results by following the code along with the dataset here.
What Didn’t Work
There were a lot of methods and ideas we tried that did not work and we strongly believe it’s worth sharing those also to give a full picture of the process of creating this model. Here are a few strategies we tried that did not improve results:
Preprocessing the images with a color mask to only select the nuclei of the cells.

Left: the original image. Right: the image after applying a color mask.
We tried adding a simple color mask to preprocess the images to just be the strong purple hues (nuclei). This reduced the accuracy of our classifier to around 88%. We assume this is because the range of the color mask selected did not generalize well to all images in our dataset (due to different lighting conditions) as well as the fact that the cytoplasm of the cells were also important in predicting their type.
2. Training on an unbalanced dataset.
Initially, we augmented the training dataset so that the distribution amongst classes of the original dataset was preserved. However, the classifier that resulted from this dataset was unsurprisingly heavily biased towards Polynuclear cells and had a low recall on mononuclear cells.
We then ensured that the training dataset was balanced across classes and saw improved results.
3. Using VGGNet trained on ImageNet with a Fully Connected Layer on top.

The Imagenet dataset.
We tried applying transfer learning by using VGGNet trained on ImageNet and adding a new fully connected layer on top and training just that fully connected layer. We found weak results using this approach and we believe its because the image dataset we trained on is too different from ImageNet. The last few layers of VGGNet are known to activate on final classes of the ImageNet dataset such as cats, cars etc. We think therefore that such a VGGNet model isn’t well suited for detecting cell nucleation patterns.
4. Using VGGNet w/ ImageNet weights but retraining the last CNN layer and a new Fully Connected Layer.
This did in fact lead to similar performance to our final model on binary classification but also did not perform well on the multi-class classification problem. As such, we chose to stay with the simpler LeNet based architecture that we started with.
Conclusion
Through this post, we were able to use a simple CNN model to classify the white blood cells in our dataset with an accuracy of 98% just based on image level data. We hope the results and methodology shared here provide a glimpse into how promising Deep Learning techniques are in the field of cell imaging and classification.
While the model and problem statement discussed in this post are simple, we believe they can be extended to more challenging problems with multiple classes, more varied lighting conditions, and new cell types. In particular, we are extremely excited about using such techniques in not just datasets of WBCs but a wide variety of other cells such as platelets, sickle cells, and even tumor cells.
With this software-first approach to morphology, we think we can apply Machine Learning to healthcare in a meaningful, valuable way. Most importantly we hope that we can enable:
Faster iteration cycles and improvements (as with all software).
Increased accessibility to high quality, quantitative assessments.
Lower costs and better patient outcomes.
If you’re interested in applying Machine Learning to important problems in pathology, join our small team of 6 at Athelas. We’re always looking for passionate, curious, and hard working people to join us!
Just email me at dhruv@getathelas.com for more information or see our website below:
Athelas

In-home blood diagnostic
athelas.com	
Thanks to Pranav Ramakrishnan, Bharath Ramasundar, Eric Jang, Tanay Tandon, Oliver Cameron, and Deepika Bodapatti for help with this post.

