# AthenaServing Framework (ASF)
## Vision

Provide a consistent solution for engineering management of AI capabilities for any scenario that requires AI capabilities. Users of AI scenarios in any field can quickly implement their algorithm models into a unified standard HTTP API service.

## Framework introduction

`AthenaServing Framework (hereinafter referred to as ASF)` AI reasoning service framework relies on iFLYTEK's years of experience in cloud service of AI algorithm engine and continuous exploration and practice of cloud-native. Enjoy the convenience and speed of related cloud-native components through `ASF`. AI algorithm engine developers can focus on algorithm research and technical evolution. What's more, the developers do not need to be distracted by the management of hardware resources and the development and operation of cloud services.
`ASF` is a serverless and fully managed platform framework for AI algorithm engines specially designed for AI capability developers. You can quickly deploy the AI algorithm engine by integrating the plug-ins provided in `ASF`, using network and distribution strategies, data processing, and other supporting auxiliary systems. The engine hosting platform is committed to accelerating the cloud service of AI algorithm engines. And it also provides multiple guarantees for the stability of cloud services with the help of cloud-native architecture. Deploy, upgrade, scale, operate, and monitor engines securely.
Currently, the deployment of `ASF` requires developers to master some knowledge of K8s, helm, etc. Meanwhile, the installation and deployment depend on the online mirror warehouse and helm repo, which is somewhat weak in the face of the offline environment and various operating system deployment requirements. `Cluster mirror`, `images-shim`, and other solutions allow applications to make offline deployment very smooth without any additional operations. After sealos supports `ASF` cluster mirroring, anyone can work in any scene and environment. Delivers `ASF` that pulls up an `ASF` environment with just one command. Users can centrally deploy their AI capabilities on the `ASF` framework and provide HTTP APIs to the outside world.

## What is AIGES

`AIGES`, the Chinese name loader, is the core component of `ASF`. Mainly responsible for converting the user's reasoning
code (according to established standards) into grpc/HTTP services

* AIGES And Language Wrapper

- C/C++ Wrapper
  ![](/img/ai-applications/c++.png)

- Python Wrapper
  ![](/img/ai-applications/python.png)


## Scene Oriented

AI capabilities ultimately need to be implemented into engineering, and some companies lack a unified standard AI
engineering solution.

## Problem Solutions

* AI capabilities are more service-oriented, and there is no standard
* The AI capability service is online, and there is a lot of redundant work in the service process
* No AI engineering team
* No final standard service agreement

## Features

* Optimized XRPC framework based on GRPC
* Definition of unified standard service agreement
* Hundred billion of PV traffic polishing
* Support multiple load balancing strategies
* Added support for Python code reasoning
* ...

## Architecture

![img](https://github.com/iflytek/proposals/blob/main/athenaloader/athena.png?raw=true)

## Framework Install

### Requirements

Prepare a test machine (4c8G), hard disk >=40G

### Install

1. Install the sealos.4.0

```shell
$ wget -c https://sealyun-home.oss-cn-beijing.aliyuncs.com/sealos-4.0/latest/sealos-amd64 -O sealos &&  chmod +x sealos && mv sealos /usr/bin
```



2. Create the cluster

```shell
$ sealos run labring/kubernetes:v1.19.16 labring/calico:v3.24.1   --masters 192.168.64.2 -p <password>
```

![](/img/ai-applications/sealos4-run-k8s.png)
![](/img/ai-applications/sealos4-run-k8s-2.png)
![](/img/ai-applications/sealos4-run-k8s-3.png)

* Install helm

```shell
$ sealos run labring/helm:v3.8.2 
```
* Install openebs
```shell
$ sealos run labring/openebs:v1.9.0 

```

```shell
$ sealos run registry.cn-qingdao.aliyuncs.com/labring/athenaserving:v2.0.0rc1
```


3. HTTP call AI demo capability MMOCR capability

MMOCR is an open source toolbox based on PyTorch and mmdetection, focusing on text detection, text recognition and
corresponding downstream tasks such as key information extraction. It is part of the OpenMMLab
project. [Project address](https://github.com/open-mmlab/mmocr/blob/main/README_zh-CN.md)
In [wrapper.py](https://github.com/iflytek/aiges/blob/master/demo/mmocr/wrapper/wrapper_v2.py), we use python to easily
convert [text + detection and recognition ability] (https ://mmocr.readthedocs.io/zh_CN/latest/demo.html#id4)
encapsulates the ability to deploy into `ASF` as an HTTP API.
After deploying `ASF` using Sealos, you can use the following script to modify the `url` value to complete the call to
the `MMOCR (text + detection)` AI capability.

```python
import requests
import json
import base64

image = open("demo_text_det.jpg","rb")
img = base64.b64encode(image.read())


url = "http://<your nodeIP>:30889/mmocr"
url = "http://<your nodeIP:30889/v1/private/mmocr"
method = "POST"
headers = {"Content-Type":"application/json"}
data = {
    "header": {
        "app_id": "123456",
        "uid": "39769795890",
        "did": "SR082321940000200",
        "imei": "8664020318693660",
        "imsi": "4600264952729100",
        "mac": "6c:92:bf:65:c6:14",
        "net_type": "wifi",
        "net_isp": "CMCC",
        "status": 3,
        "request_id": None
    },
    "parameter": {
        "mmocr": {
            "category": "ai_category",
            "application_mode": "common_gpu",
            "gpu_id": "first",
            "gpu_type": "T4G16",
            "boxes": {
                "encoding": "utf8",
                "compress": "raw",
                "format": "json"
            }
        }
    },
    "payload": {
        "data": {
            "encoding": "jpg",
            "image": img.decode("utf-8"),
            "status": 3
        }
    }
}

# call the http api.
resp = requests.post(url,headers=headers,data=json.dumps(data))

print(resp.status_code)

if resp.status_code != 200:

    print(resp.json())

result = resp.json()['payload']['boxes']['text']
print("HTTP API response is : %s "% str(result))

print("########################################")

for box in result[0].get("result"):

    msg = "MMocr Result: box located at {box}, box score is {box_score}.  Detected text is {text} , text  score is {text_score}..."
    print(msg.format(**box))
```

***Call and the Response ***

```bash
cd /var/lib/sealos/data/default/rootfs/athenaserving/charts/mmocr_ase
# modify demo.py 's url, using your nodeIP
python3 demo.py
```

Response Is:

```bash
200
HTTP API response is : [{'filename': '0', 'result': [{'box': [190, 37, 253, 31, 254, 46, 191, 52], 'box_score': 0.9566415548324585, 'text': 'nboroughofs', 'text_score': 1.0}, {'box': [253, 47, 257, 36, 287, 47, 282, 58], 'box_score': 0.9649642705917358, 'text': 'fsouthw', 'text_score': 1.0}, {'box': [157, 59, 188, 41, 194, 52, 163, 70], 'box_score': 0.9521175622940063, 'text': 'londond', 'text_score': 0.9897959183673469}, {'box': [280, 58, 286, 50, 306, 67, 300, 74], 'box_score': 0.9397556781768799, 'text': 'thwark', 'text_score': 1.0}, {'box': [252, 78, 295, 78, 295, 98, 252, 98], 'box_score': 0.9694718718528748, 'text': 'hill', 'text_score': 1.0}, {'box': [165, 78, 247, 78, 247, 99, 165, 99], 'box_score': 0.9548642039299011, 'text': 'octavia', 'text_score': 1.0}, {'box': [164, 105, 215, 103, 216, 121, 165, 123], 'box_score': 0.9806956052780151, 'text': 'social', 'text_score': 1.0}, {'box': [219, 104, 294, 104, 294, 122, 219, 122], 'box_score': 0.9688025116920471, 'text': 'reformer', 'text_score': 1.0}, {'box': [150, 124, 226, 124, 226, 141, 150, 141], 'box_score': 0.9752051830291748, 'text': 'established', 'text_score': 1.0}, {'box': [229, 124, 255, 124, 255, 140, 229, 140], 'box_score': 0.94972825050354, 'text': 'this', 'text_score': 1.0}, {'box': [259, 125, 305, 123, 306, 139, 260, 142], 'box_score': 0.9752089977264404, 'text': 'garden', 'text_score': 1.1666666666666667}, {'box': [166, 142, 193, 141, 194, 156, 167, 157], 'box_score': 0.9731062650680542, 'text': 'hall', 'text_score': 1.0}, {'box': [198, 142, 223, 142, 223, 156, 198, 156], 'box_score': 0.9548938870429993, 'text': 'and', 'text_score': 1.0}, {'box': [228, 144, 286, 144, 286, 159, 228, 159], 'box_score': 0.977089524269104, 'text': 'cottages', 'text_score': 1.25}, {'box': [180, 158, 205, 158, 205, 172, 180, 172], 'box_score': 0.9400062561035156, 'text': 'and', 'text_score': 1.0}, {'box': [210, 160, 279, 158, 279, 172, 210, 174], 'box_score': 0.9543584585189819, 'text': 'pioneered', 'text_score': 1.0}, {'box': [226, 176, 277, 176, 277, 188, 226, 188], 'box_score': 0.9748533964157104, 'text': 'cadets', 'text_score': 1.0}, {'box': [183, 177, 223, 177, 223, 189, 183, 189], 'box_score': 0.9633153676986694, 'text': 'army', 'text_score': 1.0}, {'box': [201, 190, 235, 190, 235, 204, 201, 204], 'box_score': 0.9714152216911316, 'text': '1887', 'text_score': 1.25}, {'box': [175, 213, 180, 201, 211, 212, 206, 225], 'box_score': 0.9704344868659973, 'text': 'vted', 'text_score': 0.9191176470588236}, {'box': [241, 213, 278, 200, 283, 213, 246, 227], 'box_score': 0.9607459902763367, 'text': 'epeople', 'text_score': 1.0}, {'box': [208, 224, 210, 212, 223, 214, 220, 227], 'box_score': 0.9337806701660156, 'text': 'by', 'text_score': 1.0}, {'box': [223, 214, 240, 214, 240, 226, 223, 226], 'box_score': 0.969144344329834, 'text': 'the', 'text_score': 1.0}]}]
########################################
MMocr Result: box located at [190, 37, 253, 31, 254, 46, 191, 52], box score is 0.9566415548324585.  Detected text is nboroughofs , text  score is 1.0...
MMocr Result: box located at [253, 47, 257, 36, 287, 47, 282, 58], box score is 0.9649642705917358.  Detected text is fsouthw , text  score is 1.0...
MMocr Result: box located at [157, 59, 188, 41, 194, 52, 163, 70], box score is 0.9521175622940063.  Detected text is londond , text  score is 0.9897959183673469...
MMocr Result: box located at [280, 58, 286, 50, 306, 67, 300, 74], box score is 0.9397556781768799.  Detected text is thwark , text  score is 1.0...
MMocr Result: box located at [252, 78, 295, 78, 295, 98, 252, 98], box score is 0.9694718718528748.  Detected text is hill , text  score is 1.0...
MMocr Result: box located at [165, 78, 247, 78, 247, 99, 165, 99], box score is 0.9548642039299011.  Detected text is octavia , text  score is 1.0...
MMocr Result: box located at [164, 105, 215, 103, 216, 121, 165, 123], box score is 0.9806956052780151.  Detected text is social , text  score is 1.0...
MMocr Result: box located at [219, 104, 294, 104, 294, 122, 219, 122], box score is 0.9688025116920471.  Detected text is reformer , text  score is 1.0...
MMocr Result: box located at [150, 124, 226, 124, 226, 141, 150, 141], box score is 0.9752051830291748.  Detected text is established , text  score is 1.0...
MMocr Result: box located at [229, 124, 255, 124, 255, 140, 229, 140], box score is 0.94972825050354.  Detected text is this , text  score is 1.0...
MMocr Result: box located at [259, 125, 305, 123, 306, 139, 260, 142], box score is 0.9752089977264404.  Detected text is garden , text  score is 1.1666666666666667...
MMocr Result: box located at [166, 142, 193, 141, 194, 156, 167, 157], box score is 0.9731062650680542.  Detected text is hall , text  score is 1.0...
MMocr Result: box located at [198, 142, 223, 142, 223, 156, 198, 156], box score is 0.9548938870429993.  Detected text is and , text  score is 1.0...
MMocr Result: box located at [228, 144, 286, 144, 286, 159, 228, 159], box score is 0.977089524269104.  Detected text is cottages , text  score is 1.25...
MMocr Result: box located at [180, 158, 205, 158, 205, 172, 180, 172], box score is 0.9400062561035156.  Detected text is and , text  score is 1.0...
MMocr Result: box located at [210, 160, 279, 158, 279, 172, 210, 174], box score is 0.9543584585189819.  Detected text is pioneered , text  score is 1.0...
MMocr Result: box located at [226, 176, 277, 176, 277, 188, 226, 188], box score is 0.9748533964157104.  Detected text is cadets , text  score is 1.0...
MMocr Result: box located at [183, 177, 223, 177, 223, 189, 183, 189], box score is 0.9633153676986694.  Detected text is army , text  score is 1.0...
MMocr Result: box located at [201, 190, 235, 190, 235, 204, 201, 204], box score is 0.9714152216911316.  Detected text is 1887 , text  score is 1.25...
MMocr Result: box located at [175, 213, 180, 201, 211, 212, 206, 225], box score is 0.9704344868659973.  Detected text is vted , text  score is 0.9191176470588236...
MMocr Result: box located at [241, 213, 278, 200, 283, 213, 246, 227], box score is 0.9607459902763367.  Detected text is epeople , text  score is 1.0...
MMocr Result: box located at [208, 224, 210, 212, 223, 214, 220, 227], box score is 0.9337806701660156.  Detected text is by , text  score is 1.0...
MMocr Result: box located at [223, 214, 240, 214, 240, 226, 223, 226], box score is 0.969144344329834.  Detected text is the , text  score is 1.0...
```

## Integrate access to your custom AI capabilities

To implement new AI capabilities, you should develop and build your AI Capability image according to the loader Spec Sheet. And then deploy it to the cluster.

How to build your custom AI capability image, please refer
to:[Fastly create wrapper.py](https://iflytek.github.io/athena_website/docs/%E5%8A%A0%E8%BD%BD%E5%99%A8/Python%E6%8F%92%E4%BB%B6)

## More

* Focus on:

[![ifly](https://avatars.githubusercontent.com/u/26786495?s=96&v=4)](https://github.com/iflytek)

* Contact:

![weixin](https://raw.githubusercontent.com/berlinsaint/readme/main/weixin_ybyang.jpg)

