# Image loading pipeline steps

The purpose of an image loading step is to transform an input image into an NDArray.

```python
ImageLoadingStep(
    input_schemas=None,
    output_schemas=None,
    input_names=None,
    output_names=None,
    input_column_names=None,
    output_column_names=None,
    original_image_height=None,
    original_image_width=None,
    update_ordering_before_transform=None,
    dimensions_configs=None,
    image_processing_required_layout=None,
    image_processing_initial_layout=None,
    image_transform_processes=None,
    object_detection_config=None
)
```

    :param input_schemas: Input konduit.SchemaTypes, see konduit.PipelineStep.
    :param output_schemas: Output konduit.SchemaTypes, see konduit.PipelineStep.
    :param input_names: list of step input names, see konduit.PipelineStep.
    :param output_names: list of step input names, see konduit.PipelineStep.
    :param input_column_names: Input name to column name mapping, see konduit.PipelineStep.
    :param output_column_names: Input name to column name mapping, see konduit.PipelineStep.
    :param original_image_height: input image height in pixels
    :param original_image_width: input image width in pixels
    :param update_ordering_before_transform: boolean, defaults to False
    :param dimensions_configs: dictionary defining input shapes per input name, e.g. {"input", [28,28,3]}
    :param image_processing_required_layout: desired channel ordering after this pipeline step has been applied,
           either "NCHW" or "NHWC", defaults to the prior
    :param image_processing_initial_layout: channel ordering before processing, either "NCHW" or "NHWC", defaults to the prior
    :param image_transform_processes: a DataVec ImageTransformProcess
    :param object_detection_config: konduit.ObjectDetectionConfig
```python
image_loading_step = ImageLoadingStep(
    input_column_names={"default": ["first"]}, 
    input_names=["default"], 
    input_schemas={"default": ["NDArray"]}, 
    output_names=["default"], 
    output_schemas={"default": ["NDArray"]},
    output_column_names={"default": ["first"]},
    dimensions_config={"default": [32, 32, 3]}
)
```



```yaml
  image_loading_step:
    type: IMAGE_LOADING
    image_processing_initial_layout: NCHW # I'm sending images, why is this necessary....
    image_processing_required_layout: NCHW
    input_column_names: # Why do we need to define this three times?
      default: # A default that's not 'by default'...
        - first
    input_names: # dictionary.keys() can help here...
      - default
    input_schemas:
      default:
        - NDArray # Why is this an NDArray?  I thought this step's job is to take IMAGE?
    output_names:
      - default # Yeah, default... I don't think it means what you think it means.
    output_schemas:
      default:
        - NDArray # The NUMPY vs NDARRAY vs NDArray this is very annoying...
    output_column_names:
        default:
          - first
    dimensions_configs:
      default:
          - 32
          - 32
          - 3
```



```java
ImageLoadingStep imageLoadingStepConfig = ImageLoadingStep.builder()
            .inputNames(Collections.singletonList("image_tensor"))
            .outputNames(Collections.singletonList("detection_classes"))
            .objectDetectionConfig(objectRecognitionConfig)
            .build();
```

