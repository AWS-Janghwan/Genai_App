# Amazon Bedrock으로 이미지 생성 웹 애플리케이션 만들기

텍스트를 세부적인 이미지로 변환해야 하는 앱 아이디어가 있는 개발자라고 가정해 보겠습니다. 개발을 하려는 주제가 게임일 수도 있고, 독특한 밈 생성기일 수도 있습니다. 이러한 애플리케이션을 제작하는 것은 자연어 처리와 컴퓨터 비전 모두에 대한 전문 지식이 필요한 복잡한 작업이라는 것을 알고 계실 것입니다.

아마존 베드락은 이미지 생성을 위한 Stable Diffusion 모델을 제공하며, Streamlit을 통해 인터랙티브 웹 앱을 손쉽게 만들수 있도록 합니다. 
이 조합을 통해 복잡한 과정을 생략하고 바로 원하는 생성형 애플리케이션 구축에 시작할 수 있습니다.


코딩을 시작하기 전에 우리가 사용할 두 가지 핵심 구성 요소를 이해해 보겠습니다.

- Streamlit: 이 오픈 소스 Python 라이브러리는 인터랙티브 웹 앱 구축의 판도를 바꾸고 있습니다. 빠르고 쉬운 프런트엔드 개발을 위한 도구라고 생각하시면 됩니다.

- Stable Diffusion: 2022년에 출시된 이 텍스트-이미지 변환 모델은 텍스트 설명에서 상세한 비주얼을 생성하는데 매우 뛰어난 성능을 보여줍니다.


아래 내용은 Amazon Bedrock을 통해 이미지 생성 Application을 만드는 예제 코드에 대한 설명입니다.

## Step #1: 모듈 및 환경변수 설정

```
import streamlit as st
import boto3
import json
from PIL import Image
import io


st.title("Building with Bedrock")  # Title of the application
st.subheader("Stable Diffusion Demo")


# List of Stable Diffusion Preset Styles
sd_presets = [
    "None",
    "3d-model",
    "analog-film",
    "anime",
    "cinematic",
    "comic-book",
    "digital-art",
    "enhance",
    "fantasy-art",
    "isometric",
    "line-art",
    "low-poly",
    "modeling-compound",
    "neon-punk",
    "origami",
    "photographic",
    "pixel-art",
    "tile-texture",
]
```

## Step #2: Amazon Bedrock 설정
Python 스크립트에서 Amazon Bedrock을 초기화하고 실행 준비를 합니다.

```
# Define Amazon Bedrock
bedrock_runtime = boto3.client(
    service_name='bedrock-runtime',
    region_name='us-west-2',
)
```

## Step #3: Stable Diffusion 모델 실행(Invoke)
베드락 설정이 끝났으면, Stable Diffusion 모델을 실행합니다.

```
# Amazon Bedrock api call to stable diffusion
def generate_image(text, style):
    """
    Purpose:
        Uses Bedrock API to generate an Image
    Args/Requests:
         text: Prompt
         style: style for image
    Return:
        image: base64 string of image
    """
    body = {
        "text_prompts": [{"text": text}],
        "cfg_scale": 10,
        "seed": 0,
        "steps": 50,
        "style_preset": style,
    }

    if style == "None":
        del body["style_preset"]

    body = json.dumps(body)

    modelId = "stability.stable-diffusion-xl"
    accept = "application/json"
    contentType = "application/json"

    response = bedrock_runtime.invoke_model(
        body=body, modelId=modelId, accept=accept, contentType=contentType
    )
    response_body = json.loads(response.get("body").read())

    results = response_body.get("artifacts")[0].get("base64")
    return results

```


## Step #4: 화면에 생성된 이미지 출력
모델을 통해 이미지를 생성한 후 Python 이미징 라이브러리(PIL)를 사용하여 이미지를 화면에 표시합니다.

```
# Turn base64 string to image with PIL
def base64_to_pil(base64_string):
    """
    Purpose:
        Turn base64 string to image with PIL
    Args/Requests:
         base64_string: base64 string of image
    Return:
        image: PIL image
    """
    import base64

    imgdata = base64.b64decode(base64_string)
    image = Image.open(io.BytesIO(imgdata))
    return image
```

## Step #5: 인터렉티브 애플리케이션 생성
 Streamlit을 사용하여 사용자가 직접 설명을 입력할 수 있는 텍스트 상자를 추가하고 Stable Diffusion이 생성하는 이미지를 실시간으로 확인할 수 있습니다.

 ```
# List of Stable Diffusion Preset Styles
#sd_presets = ["None","3d-model","analog-film","anime",...

# select box for styles
style = st.selectbox("Select Style", sd_presets)
# text input
prompt = st.text_input("Enter prompt")

#  Generate image from prompt,
if st.button("Generate Image"):
    image = base64_to_pil(generate_image(prompt, style))
    st.image(image)
```
