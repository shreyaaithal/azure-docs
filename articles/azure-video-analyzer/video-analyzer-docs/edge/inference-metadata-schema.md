---
title: Inference metadata schema - Azure
description: In Azure Video Analyzer, each inference object regardless of using HTTP-based contract or gRPC based contract follows the object model described in this topic.
ms.topic: reference
ms.date: 06/01/2021
ms.custom: ignite-fall-2021
---

# Inference metadata schema 

[!INCLUDE [header](includes/edge-env.md)]

In Azure Video Analyzer, each inference object regardless of using HTTP-based contract or gRPC based contract follows the object model described in this topic.

## Object model

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/inference-metadata-schema/object-model.png" alt-text="Azure Video Analyzer Oot model" lightbox="./media/inference-metadata-schema/object-model.png":::
 

|Type Definition|Description|
|---|---|
|Tag|Tag or label associated with the result. Along with tagging, you also get the confidence value associated with the tag.|
|Attribute|Additional attributes associated with the result. You can add new attributes that you receive from the inferencing engine along with the confidence value.|
|Attribute List|List of	Optional attributes.|
|Rectangle|Rectangular region relative to the image top-left corner. The required properties will be `left`, `width`, `height` and `top` from the origin". <br/>The left, top, width, and height of the bounding box are normalized in a scale of 0.0 to 1.0. The `left` (x-coordinate) and `top` (y-coordinate) are coordinates that represent the top and left sides of the bounding box. The `left` and `top` values returned are ratios of the overall video dimensions. For example, if the video dimensions are 1920 x 1080 pixels, and the top-left coordinate of the bounding box is (350,50), the bounding box has a `left` value of 0.1822 (350/1920) and a `top` value of 0.0462 (50/1080). <br/>The `width` and `height` values represent the dimensions of the bounding box as a ratio of the overall video dimension. For a video with dimension s of 1920x1080, if the bounding box width is 70 pixels, the width returned is 0.0364 (70/1920).|
|Classification|Label of classifier often applicable to the whole sample. With the help of "tag", you can classify the result.|
|Entity|Entity detected or identified on the sample. When an entity is detected by the inferencing engine, it gets a "tag", additional attributes that were inferred and the coordinates of a rectangular box around the found entity is returned.|
|Event|Event detected on the sample. When an event is detected in the sample, the name of the event and the event-specific properties are returned.|
|Motion|Motion detected on the sample. When motion is detected in the sample, the coordinates of a rectangular bounding box where motion is detected, is returned.|
|Text|Text associated with the sample along with the start and end timestamp of the text is returned.|
|Other|Returns other generic payload information.|

The example below contains a single event with some supported inference types:

```
[ 
  // Light Detection 
  { 
    "type": "classification", 
    "subtype": "lightDetection", 
    "classification": { 
      "tag": { "value": "daylight", "confidence": 0.86 }, 
      "attributes": [ 
          { "name": "isBlackAndWhite", "value": "false", "confidence": 0.71 } 
      ] 
    } 
  }, 
 
  // Motion Detection 
  { 
    "type": "motion", 
    "subtype": "motionDetection", 
    "motion": 
    { 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    } 
  }, 
 
  // Yolo V3 
  { 
    "type": "entity", 
    "subtype": "objectDetection",     
    "entity": 
    { 
      "tag": { "value": "dog", "confidence": 0.97 }, 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    } 
  }, 
 
  // Vehicle Identification 
  { 
    "type": "entity", 
    "subtype": "vehicleIdentification",     
    "entity": 
    { 
      "tag": { "value": "007-SPY", "confidence": 0.82 }, 
      "attributes":[   
        { "name": "color", "value": "black", "confidence": 0.90 }, 
        { "name": "body", "value": "coupe", "confidence": 0.87 }, 
        { "name": "make", "value": "Aston Martin", "confidence": 0.35 }, 
        { "name": "model", "value": "DBS V12", "confidence": 0.33 } 
      ], 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    } 
  }, 
 
  // People Identification 
  { 
    "type": "entity", 
    "subtype": "peopleIdentification",     
    "entity": 
    { 
      "tag": { "value":"Erwin Schrödinger", "confidence": 0.50 }, 
      "attributes":[   
        { "name": "age", "value": "73", "confidence": 0.87 }, 
        { "name": "glasses", "value": "yes", "confidence": 0.94 } 
      ], 
      "box": { "l": 0.0, "t": 0.0, "w": 0.0, "h": 0.0 } 
    }, 
 
    // Open type coming from the gRPC Map 
    "extensions":  
    { 
      "vector": "e1xkaXNwbGF5c3R5bGUgaVxoYmFyIHtcZnJhYyB7ZH17ZHR9fVx2ZXJ0IFxQc2kgKHQpXHJhbmdsZSA9e1xoYXQge0h9fVx2ZXJ0IFxQc2kgKHQpXHJhbmdsZSB9KQ==", 
      "skeleton": "p1,p2,p3,p4" 
    } 
  }, 
 
  // Captions 
  {     
    "type": "text", 
    "subtype": "speechToText",   
    "text": 
    { 
      "value": "Humor 75%. Confirmed. Self-destruct sequence in T minus 10… 9…", 
      "language": "en-US", 
      "startTimestamp": 12000, 
      "endTimestamp": 13000 
    } 
]
```

## Next steps

- [gRPC data contract](./grpc-extension-protocol.md)
- [HTTP data contract](./http-extension-protocol.md)
