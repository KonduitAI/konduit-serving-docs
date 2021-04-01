# Pipeline Steps

Konduit-Serving works by defining a series of steps. These include operation such as:

1. Pre-processing steps
2. Running one or more ML/DL models
3. Post-processing steps, transforming the output in a way that humans can understand

{% hint style="info" %}
A reference Java project is provided in the Example repository from [https://github.com/KonduitAI/konduit-serving-examples](https://github.com/KonduitAI/konduit-serving-examples) with a Maven pom.xml dependencies file. If using the IntelliJ IDEA IDE, open the java folder as a Maven project and run the main function of each of example class.
{% endhint %}

Here are the articles in this section:

{% page-ref page="image-to-ndarray-step.md" %}

{% page-ref page="dl4j-step.md" %}

{% page-ref page="keras-step.md" %}

{% page-ref page="onnx-step.md" %}

{% page-ref page="tensorflow-step.md" %}

