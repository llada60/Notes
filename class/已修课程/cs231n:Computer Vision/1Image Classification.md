### Image Classification

Contents

- Image Classification
  - [Nearest Neighbor Classifier](https://cs231n.github.io/classification/#nearest-neighbor-classifier)
  - [k - Nearest Neighbor Classifier](https://cs231n.github.io/classification/#k---nearest-neighbor-classifier)
  - [Validation sets for Hyperparameter tuning](https://cs231n.github.io/classification/#validation-sets-for-hyperparameter-tuning)
  - [Summary](https://cs231n.github.io/classification/#summary)
  - Summary: Applying kNN in practice
    - [Further Reading](https://cs231n.github.io/classification/#further-reading)

#### Challenge

- **Viewpoint variation**. A single instance of an object can be oriented in many ways with respect to the camera.
- **Scale variation**. Visual classes often exhibit variation in their size (size in the real world, not only in terms of their extent in the image).
- **Deformation**. Many objects of interest are not rigid bodies and can be deformed in extreme ways.
- **Occlusion**. The objects of interest can be occluded. Sometimes only a small portion of an object (as little as few pixels) could be visible.
- **Illumination conditions**. The effects of illumination are drastic on the pixel level.
- **Background clutter**. The objects of interest may *blend* into their environment, making them hard to identify.
- **Intra-class variation**. The classes of interest can often be relatively broad, such as *chair*. There are many different types of these objects, each with their own appearance.

#### The image classification pipeline

- **Input:** Our input consists of a set of *N* images, each labeled with one of *K* different classes. We refer to this data as the *training set*.
- **Learning:** Our task is to use the training set to learn what every one of the classes looks like. We refer to this step as *training a classifier*, or *learning a model*.
- **Evaluation:** In the end, we evaluate the quality of the classifier by asking it to predict labels for a new set of images that it has never seen before. We will then compare the true labels of these images to the ones predicted by the classifier. Intuitively, we’re hoping that a lot of the predictions match up with the true answers (which we call the *ground truth*).

#### K-Nearest Neighbor Classifier (KNN)

查看当前点p附近最近的K个点，哪种类别多则p为该类别。

1. Preprocess your data: **Normalize the features in your data** (e.g. one pixel in images) to have zero mean and unit variance. We will cover this in more detail in later sections, and chose not to cover data normalization in this section because pixels in images are usually homogeneous and do not exhibit widely different distributions, alleviating the need for data normalization.
2. If your data is very high-dimensional, consider using a **dimensionality reduction** technique such as PCA ([wiki ref](https://en.wikipedia.org/wiki/Principal_component_analysis), [CS229ref](http://cs229.stanford.edu/notes/cs229-notes10.pdf), [blog ref](https://web.archive.org/web/20150503165118/http://www.bigdataexaminer.com:80/understanding-dimensionality-reduction-principal-component-analysis-and-singular-value-decomposition/)), NCA ([wiki ref](https://en.wikipedia.org/wiki/Neighbourhood_components_analysis), [blog ref](https://kevinzakka.github.io/2020/02/10/nca/)), or even [Random Projections](https://scikit-learn.org/stable/modules/random_projection.html).
3. Split your **training data randomly into train/val splits**. As a rule of thumb, between 70-90% of your data usually goes to the train split. This setting depends on how many *hyperparameters* you have and how much of an *influence you expect them to have*. If there are many hyperparameters to estimate, you should err on the side of having larger validation set to estimate them effectively. If you are concerned about the size of your validation data, it is best to split the training data into folds and perform cross-validation. If you can afford the computational budget it is always safer to go with cross-validation (the more folds the better, but more expensive).
4. **Train and evaluate the kNN classifier on the validation data** (for all folds, if doing cross-validation) for many choices of **k** (e.g. the more the better) and across different distance types (L1 and L2 are good candidates)
5. If your kNN classifier is running too long, consider using an *Approximate Nearest Neighbor library* (e.g. [FLANN](https://github.com/mariusmuja/flann)) to <u>accelerate the retrieval (at cost of some accuracy).</u>
6. **Take note of the hyperparameters that gave the best results**. There is a question of whether you should use the full training set with the best hyperparameters, since the optimal hyperparameters might change if you were to fold the validation data into your training set (since the size of the data would be larger). In practice it is cleaner to not use the validation data in the final classifier and consider it to be *burned* on estimating the hyperparameters. Evaluate the best model on the test set. Report the test set accuracy and declare the result to be the performance of the kNN classifier on your data.

**L1 distance**
$$
d_1(I_1,I_2) = \sum_p |I_1^p-I_2^p|
$$
**L2 distance**
$$
d_2(I_1,I_2) = \sqrt{\sum_p(I_1^p-I_2^p)^2}
$$


#### hyperparameter tuning

**CANNOT use the test set for the purpose of tweaking hyperparameters**: this is very real danger is that you may tune your hyperparameters to work well on the test set, but if you were to deploy your model you could see a significantly reduced performance. <=> **overfit** to the test set.  (But use test set <u>once</u> at end, it remains a good proxy for measuring the **generalization** of your classifier.)

use **validation sets** as a fake test set to tune the hyperparameters

##### Cross-validation

the size of your training data (and therefore also the validation data) might be *small*

eg: 5-fold cross-validatoin, split the training data into 5 equal folds, use 4 of them for training and 1 for validation.

![image-20230926190635647](img/image-20230926190635647.png)

In practice, people prefer to avoid cross-validation in favor of having a single validation aplit, since cross-validation can be computationally expensive.

#### Pros and Cons of Nearest Neighbor classifier

Pros:

- takes no time to train

Cons:

- in practice, we care about the test time efficiency much more than the efficiency at training time.

Images that are nearby each other(L2) are much more a function of the general *color* distribution of the images/ the type of *background*  rather than *semantic* identity.

#### [t-SNE](https://lvdmaaten.github.io/tsne/) Visualization

t-Distributed Stochastic Neighbor Embedding (t-SNE) is a technique for dimensionality reduction that is particularly well suited for the visualization of high-dimensional datasets. The technique can be implemented via Barnes-Hut approximations, allowing it to be applied on large real-world datasets. 
