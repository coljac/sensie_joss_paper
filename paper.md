---
title: '``Sensie``: Probing the sensitivity of neural networks'

tags:
  - Python
  - machine learning
  - neural networks
authors:
  - name: Colin Jacobs
    orcid: 0000-0003-4239-4055
    affiliation: "1" 
affiliations:
 - name: Swinburne University of Technology
   index: 1
date: 20 March 2020
bibliography: paper.bib

---

# Introduction 

Deep neural networks (DNNs) are finding increasing application across a wide variety of fields, including in industry and scientific research. Although DNNs are able to successfully tackle data problems that proved intractable to other methods, for instance in computer vision, they suffer from a lack of interpretability. DNNs excel at creating extremely non-linear mappings between input and output spaces, and since such a network may have up to a billion trainable parameters or even more, understanding the reasons behind a classification, output or decision is far from straight forward. In applications such as finance, science and medicine, better understanding the reasons for a decision, and the weaknesses in a model, is an issue of increasing focus [see for example @montavonMethodsInterpretingUnderstanding2018].

Some well-known methods for visualising and interpreting the outputs of DNNs include directly inspecting the learned features of a model directly (e.g. convolutional kernels and activation maps); or producing "saliency maps" which allow a human researcher to inspect which parts of an input (such as an image) played the most crucial role in the model reaching a particular determination. These algorithms include occluding parts of an image [@zeilerVisualizingUnderstandingConvolutional2014], guided backpropagation [@simonyanDeepConvolutionalNetworks2013], and more complex algorithms such as SmoothGrad [@smilkovSmoothGradRemovingNoise2017] and Layer-wise relevance propagation [@binderLayerwiseRelevancePropagation2016].

However, simply inspecting the most salient regions of an input image (or other input) may not always be sufficiently interpretable. This may be particularly true in a scientific context, where the physical properties of or relating to an input value are highly significant to the task at hand. Inspecting a saliency map may indicate that the network is using a subset of the input more than others, but from this it is challenging to draw conclusions about the robustness of the model. 

Here we present ``Sensie``, a tool to quickly inspect, and quantify, the sensitivity of a trained model to user-specified properties of the input domain, or under the arbitrary transformation of a test set.

# Method

``Sensie`` helps the user explore the sensitivity of a machine learning model to properties of data (using a test set with known, correct outputs). ``Sensie`` wraps a pre-trained model, and assists in quantifying and visualising changes in the performance of the model as a function of either a specified property of the data (one that is explicitly passed to the model at test time, or other metadata); or a perturbation of the data, parameterised by a value *p*. In addition, ``Sensie`` offers a convenience method to display the sensitivity of a classification model to the class itself, though this should be used with caution.

``Sensie``'s algorithm is summarised as follows as applied to a network trained for classification, for two use cases:

**A)** Using data or metadata: a scalar property $p$ of the test set $X_{\textrm{test}}$, such that each example $\boldsymbol{x}_i$ has a corresponding value $p_i$ and a supplied ground truth $\hat{y}$:

1. Segment the data into bins by $p$.
2. Collect predictions from the model (using ``model.predict()`` or a user-supplied function), and note the scores for the correct classes $\hat{y}$.
3. Calculate the mean score $\bar{s}$ in the correct class for each bin, and the standard deviation.
4. Plot $\bar{s}$ as a function of $p$.
5. Estimate the significance of the effect using Bayesian linear regression, producing a scalar value representing $\partial \bar{s}/\partial p$, with 50% and 95% credible intervals.

**B)** Using a perturber function $f_{\textrm{perturb}}(\boldsymbol{x}, p)$---an arbitrary transformation of the inputs---applied to the test set, where the magnitude of perturbation is parameterised by $p$:

1. Choose discrete values of $p$ to be tested.
2. For each value of $p$ to be tested, $p_j$, transform the test set such that $X'_j = f_{\textrm{perturb}}(X_\textrm{test}, p_j)$.
3. Collect predictions from the model for $X'_j$ (using ``model.predict()`` or a user-supplied method), and note the scores for the correct classes, $\hat{y}$.
4. Calculate the mean score $\bar{s}$ in the correct class for each $p_j$, and the standard deviation.
5. Plot $\bar{s}$ as a function of $p$.
6. Estimate the significance of the effect using Bayesian linear regression[^bayesian], producing a scalar value representing $\partial \bar{s}/\partial p$, with 50% and 95% credible intervals.

In the first case, A), ``Sensie`` can optionally forego binning by P, and treat every element as a data point in determining the trend.

By this method, ``Sensie`` helps the model user to quickly identify sensitivities of the accuracy to various properties of the input. This may identify deficiencies in the training set; for instance, if we expect an image classifier to be robust under a particular transformation such as rotation or translation, ``Sensie`` can quickly quantify whether such a transformation has a statistically significant effect on the accuracy across the test set. Figure 1 shows two examples; on the left, an output plot from ``Sensie`` showing the sensitivity of a trained model to rotation of the input images; the fact that the curve is not flat indicates that the model is highly sensitive to input orientation and that this is a significant feature (or bug) of the training process. On the right, the sensitivity of a trained model to a Gaussian blur applied to the model, along with a fit indicating the magnitude of the effect.

![Left: Output from ``Sensie`` for a model trained to recognise handwritten digits, testing model sensitivity to rotation. Error bars show the standard deviation for the mean ground-truth-class score. Right: Sensitivity of a model to ann applied blir of the input image data, showing a linear fit to a significant region.](sensie_examples.png)

# Discussion and conclusion

The algorithm outlined above is simple, but can be used to give a quantitative indication of learned properties (or weaknesses) in a trained model. In scientific applications it can assist in quantifying the size of an effect, or the magnitude of sensitivity to a physical property of the inputs.

The sensitivity to any given property, i.e. the curve $\bar{s}$ vs $p$, may not be linear. In this case, a fit using linear regression may produce a meaningless result. Where a theoretical basis exists for an alternative relationship between the accuracy and property in question exists, then the user can attempt a fit using an alternative function using the arrays returned by ``Sensie``; the code can also fit a polynomial of arbitrary order to the curve. Alternatively, the algorithm can be re-run over a subset of the parameter space where a linear fit will provide a more accurate indication of model performance sensitivity.

``Sensie`` has been successfully applied to a model trained for astronomical classification [@jacobsExtendedCatalogGalaxy2019] to indicate the model's sensitivity to the scale of features present in the input; the signal-to-noise ratio of the input data; the colours of the input; and astrophysical properties of the objects present in an input image.

# References

[^bayesian]: See https://docs.pymc.io/notebooks/GLM-linear.html for an example using the PyMC3 probabilistic programming framework.
