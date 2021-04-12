---
description: Custom model in Konduit-Serving
---

# Serving a BMI Model

### Introduction

Body Mass Index \(BMI\) is a well-used measure to describe weight status based on the ratio between an individual's height and weight. BMI is used to classify an individual's weight status as underweight, normal weight, overweight or obese. Even though this is a simple application to be used, it is necessary to monitor healthcare by providing weight status generally.

Findings from [Pursey et al.](https://pubmed.ncbi.nlm.nih.gov/24398335/) and [Stommel et al.](https://bmcpublichealth.biomedcentral.com/articles/10.1186/1471-2458-9-421) showed that adults tend to self-report inaccurate BMI, overestimate their weight, and underestimate the weight. Time-consuming in measuring weight and height also one of the factors that contribute to this issue. Thus, the BMI model is introduced as a new approach for estimating BMI using facial images that contain facial features.

The model is deployed in **Konduit-Serving** through the pipeline server from pre-processing step of input until producing output that a human can understand. The backend server used for taking image data and providing BMI values is served using Konduit-Serving, a high-performance model pipeline server.

The main workflow we'll look at in this document is how to serve a model that can see at a person's face and respond with a BMI value through REST API. We'll also be setting up a web server through Konduit-Serving "custom endpoints" that will make use of a webcam and label the canvas with the detected face along with the corresponding estimated BMI value.

Note that gathering, preparing dataset and model training are out of the scope of this document.

