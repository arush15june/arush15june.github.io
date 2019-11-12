+++
published = true
title = "Building ReverSEE"
date = 2018-08-02T00:00:00.000Z
categories = ["project"]
author = "batterycharger"
tags = ["machinelearning", "pytorch", "cuda", "windows", "javascript", "python", "flask"]
+++

## Building ReverSEE

This post aims to cover the developement process of [ReverSEE](https://github.com/arush15june/reversee), a reverse image search hoopla I built to essentialy learn to use PyTorch.

# a silly idea

**popular indian fashion website** allows you to throw a picture of a random tshirt in their search bar and they'll find you how a similar tshirt available in the store. 

**Idea**: How could i go about building something similar of my own? A Mini Reverse Image Search! Perfect oppurtunity to learn to use PyTorch 0.4.0, recently released on Windows, where with accidental CUDA 9.1 drivers, Tensorflow wouldn't run without building it from source, a task i refused to commit.

The basic architecture:

`Web Frontend <-- API --> Model` 

To fuel this idea with my limited skills in machine learning, I sought out to prepare a dataset of product images which would then be used to train a model, comparing an input image with th database with the most probable outputs be displayed on a webpage.

Quick google searches led me to the ideas of Siamese Networks, a kind of Convolutional Neural Networks, which could be trained and used for comparison, in this case, the tshirt database with an input image.

# The Dataset

Though, ReverSEE could be trained on a dataset of pretty much any product, I chose T-Shirts. Back to the popular indian fashion website, it has a client side rendered frontend, just like all modern websites, which means all the product data is procured by requesting an HTTP API using XML-HTTP Requests.

In the Network tab of the Chrome Developer Tools, filtering it to only display XHR, you could find out the API Endpoint used to get product data for a specific category, in this case, `Men's T-Shirts`. The endpoint was actually found out by a friend, [Ujjwal Gupta](https://github.com/slapbot).

With this endpoint we could request paginated tshirt data, a simple script, `model/scraper/scraper.py` to scrape this data was written which scraped around 10000 odd images of tshirts.

Although, after a lot of work had been done, It was found that there were a lot of duplicate images, around 2500 images, this was later fixed by a bash command/tool (WSL Bash runnning on Windows 10) which i have since lost. It removed duplicates based on their md5 hash. 7500~ pictures remain after this cleansing.

All of these pictures were downloaded at a high quality resolution of 1080x1440 pixels, later resized to 130x150 pixels using `model/scraper/resize.py` to be used for training the the model.

# The Model

Before beginning research on the internet on how I could compare two pictures using neural networks, My own idea of the model was to use a CNN to classify the input images as one of the output classes, the classes representing each picture in the database, with this structure we could get the most probable class for an input image, this was later implemented too.  

Research a.k.a using google, led to different articles and papers,	
- [https://hackernoon.com/one-shot-learning-with-siamese-networks-in-pytorch-8ddaab10340e](https://hackernoon.com/one-shot-learning-with-siamese-networks-in-pytorch-8ddaab10340e)
- [https://arxiv.org/pdf/1703.02344](https://arxiv.org/pdf/1703.02344)
- [https://www.cs.cmu.edu/~rsalakhu/papers/oneshot1.pdf](https://www.cs.cmu.edu/~rsalakhu/papers/oneshot1.pdf)
    
Teaching me about Siamese Networks, using two identical convolutional neural networks to generate a parameter which could be used to infer similarity between two images. With my limited understanding of CNN's and Siamese Networks, I will refrain from any technincal comments regarding how the process is done but it is covered in [https://www.cs.cmu.edu/~rsalakhu/papers/oneshot1.pdf](https://www.cs.cmu.edu/~rsalakhu/papers/oneshot1.pdf) for those interested.

The base code for the model was harvested from [https://github.com/harveyslash/Facial-Similarity-with-Siamese-Networks-in-Pytorch](https://github.com/harveyslash/Facial-Similarity-with-Siamese-Networks-in-Pytorch), It uses two images and a truth label to train the model, the `SiameseNetworkDataset` class was refactored to randomly, either use the same or get a new image from the tshirt database, and a truth label was generated alongside. The `SiameseNetwork` class was tweaked to handle our cropped RGB 130x150 images and the number of feature maps generatead tweaked for my laptop's feeble GTX1050Ti to handle.

The model was trained, `model/train.py` using the 130x150 pixel RGB Images on Windows 10 with CUDA 9.1 using PyTorch 0.4.0. It was trained for around 50 epochs.

Jupyter notebook `model/QueryTest.ipynb` can be used to test this trained Siamese Network, it iterated over all the images present in the database and compared it with the input. This task took around 100 seconds to complete on the 10000 image dataset on laptop, impractical to be used to power an HTTP API.

I could not formulate a quantitative method to judge the model, but it seemed to generate decent results based on my own judgement, atleast suggesting a few similarly coloured tshirts in the top 10 probable.

# The Second Model

The second model of a standard CNN, or the original model which was thought about, was built out of the `SiameseNetwork` architecture. 

Output labels based on the image dataset filenames were generated using `model/scraper/gen_classes.py`, The model was trained for 50 epochs. Classes for querying these models with input images were built, `QueryModelCNN` accepts a PIL.Image instance and returns top 10 similar image paths from the database.

Querying the CNN was a rather instant process compared to the 100 seconds taken by the Simaese Network, rather practical for powering an API.

`model/QueryTestCNN.ipynb` can be used to test the CNN on input image, again, there was a lack of a quantitative method of judging the model, based on the test of my eyes, for similar images the model seemed to perform worse than the Siamese Network.

The models were saved as files to be used for serving through the api.

# The API

Flask was used to build a RESTful API to serve the model. Helper classes in `app/helpers.py` queried the model with an image recieved by the API and returned a `pandas.DataFrame` of the top 10 most probable items.

POST `/api/matchcnn` accepted the image data sent by a client and sent back a JSON list of top 10 most probable items, which were paths of static files being served by Flask (atleast for the development server).

# The Frontend

Thanks to [Kautuk Kundan](https://github.com/kautukkundan) for contributing the frontend to the project. It uses the HTML5 Fetch API for requesting the API and updates the DOM.

![ReverSEE](https://i.imgur.com/2IKKQpz.jpg)

# The End

I apologize for any incorrect statements, inaccuracies and all grammatical errors. Constructive Criticism Appreciated.
