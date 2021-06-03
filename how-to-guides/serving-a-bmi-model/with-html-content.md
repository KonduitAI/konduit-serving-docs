---
description: Visualizing the deployment with HTML
---

# With HTML Content

In this section, we're implementing visualization via HTML for:

1. Metrics
2. Web Application

## Metrics

The metrics are data related to the process of every execution on Konduit-Serving. The REST API handles the metrics endpoint of Konduit-Serving and returns to the Prometheus. Several metrics return to the metrics endpoint by default, which are:

* Time taken for request and execution
* CPU usage and current available memory
* GPU usage, GPU temperature and GPU current used memory
* Result of execution

Prometheus and Grafana are used to store the data and visualize the metrics onto the dashboard. 

### Prometheus

Prometheus is an open-source system monitoring and alerting toolkit widely used in time series database for tracking system metrics used for debugging production systems. The Prometheus collects all the metrics from metrics endpoint and formatting into something that Prometheus can read, present in the form of time series data for every time it polls the server for metrics scraping. A Konduit-Serving instance exposes metrics to be picked up by Prometheus and shows the metrics on `localhost:9090`.

### Grafana

Grafana is a dashboard system for pulling data from different sources and displaying it in real-time. It can take the data from multiple sources. One of those is that Grafana takes the data from Konduit-Serving through Prometheus and visualizes all the data on the dashboard.

Let's run the HTML code like the following \(assuming you're still running the notebook from the demo file\).

```text
%%html
<div style="display: flex; justify-content: center; align-items: center; border: 1px solid black;">
    <iframe src="http://localhost:3000/d/lP_JcnHWz/pipeline-metrics?orgId=1&refresh=5s&kiosk&var-serverName=bmi-onnx-pytorch" width=1500 height=1500>
</div>
```

Once the cell is committed, the Grafana dashboard shows the metrics previously run on the test image or can open the [browser here](http://localhost:3000/d/lP_JcnHWz/pipeline-metrics?orgId=1&refresh=5s&kiosk&var-serverName=bmi-onnx-pytorch) to view the metrics while the kernel is running.

## Web Application

To make the deployment of Konduit-Serving more natural, we provide a simple web application to ensure the BMI model can also be tested on your local machine with a video feed via HTML. By just running the cell similar to the below, you can start to get your BMI status. \(Note: The result may not be precisely accurate. This is only for guidance purpose.\)

```text
%%html
<div style="display: flex; justify-content: center; align-items: center; border: 1px solid black;">
    <iframe src="http://localhost:9009/web-app/index.html" allow="camera;microphone", width=1000 height=1000></iframe>
</div>
```

This web application is configured on `static_content` in the YAML configuration file, `bmi-onnx-pytorch.yaml` to implement via HTML. The directory of the HTML file is `web-app/index.html`, which is executing video streaming from the local machine in a small window of running cell and predict BMI from the image captured when the start button is clicked.

You'll notice that the dashboard is updating the metrics for each time the prediction is made continuously on the running stream video feed and producing the time series data for all available metrics until the stop button is clicked.

