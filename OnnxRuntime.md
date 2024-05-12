## 1) ONNX Runtime?

**💡 ONNX(Open Neural Network Exchange) 란?**

- 딥러닝 모델을 서로 다른 프레임워크 간에 서로 옮길 수 있도록 하는 오픈 소스 프로젝트입니다. ONNX는 모델을 중간 계층 형식으로 변환하고 모델을 실행하는 데 필요한 라이브러리를 제공합니다.
- 간략히 말해, 다양한 플랫폼 환경(Java, JS, C, C#, C++)에서 환경에 제약 없이 구현된 ‘ML 모델’을 호출하고 수행하여 수행 결과 값을 반환받는 것을 의미합니다.

**💡 ONNX Runtime 이란?**

- **ONNX 모델을 실행하기 위한 엔진입니다. ONNX 모델을 실행하기 위해 ONNX 런타임은 빠른 추론을 위한 최적화된 커널을 사용합니다.**
- 또한, ONNX Runtime은 CPU, GPU 및 딥러닝 가속기(DNNL, NNAPI, OpenVINO)를 지원합니다.
- 따라서, ONNX Runtime은 ONNX 모델을 실행하기 위한 최적화된 런타임 환경을 제공합니다.
    
    [[RN] ONNX(Open Neural Network Exchange) 이해하기 -1: React Native 활용](https://adjh54.tistory.com/203)
    

## 2) 모델 변경 : pth to onnx file

### 2.1 pth file

> **ChatGPT**
> 
> 
> **`model.pth`** 파일은 일반적으로 머신 러닝 및 딥 러닝 모델을 저장하는 데 사용되는 파일입니다. "pth"는 여기서 "PyTorch"를 의미하는 경우가 많습니다.
> 
> PyTorch는 파이썬 기반의 오픈 소스 머신 러닝 프레임워크로, 신경망을 구축하고 훈련시키는 데 사용됩니다. **`.pth`** 확장자를 가진 파일은 주로 PyTorch 모델의 가중치(weight)와 구조를 저장하는 데 사용됩니다.
> 
> 따라서 **`model.pth`** 파일을 보면, 그 안에는 특정 모델의 학습된 가중치(weight)가 저장되어 있을 가능성이 높습니다. 이 가중치는 해당 모델을 로드하고 사용하여 추론(inference)이나 추가적인 훈련을 수행하는 데 사용될 수 있습니다.
> 

### **2.2. Pth_to_Onnx.py**

```python
from Anal_lib.utils import load_json, make_dir
import AI_lib.utils as ai_utl 
import os
import json
import numpy as np
import argparse
import torch.onnx 
import torch.nn as nn
import torch.nn.functional as F

arg_parser = argparse.ArgumentParser(description='pre-processing for PL2Map dataset')
arg_parser.add_argument('--single_uid_num', type=int, default=0, 
						help='Input single uid number to evaluate')
arg_parser.add_argument('--eval_single_uid', type=int, default=0, choices=[0,1], 
						help='Evaluate single uid or not')
arg_parser.add_argument('--top_app', type=int, default=5, 
						help='It must match with training top_app')
args, _ = arg_parser.parse_known_args()
ori_number_top_app = args.top_app

# pth model 생성시에 사용한 동일한 Network 사용
class Network(nn.Module):
	def __init__(self):
		super(Network, self).__init__()
		self.lr1 = nn.Linear(input_dim, 25)
		self.lr2 = nn.Linear(25, 50)
		self.lr3 = nn.Linear(50, 25)
		self.lr4 = nn.Linear(25, output_dim)
	def forward(self, x):
		x = F.relu(self.lr1(x))
		x = F.relu(self.lr2(x))
		x = F.relu(self.lr3(x))
		x = self.lr4(x)
		return x

#Function to Convert to ONNX 
def Convert_ONNX(): 

    # set the model to inference mode 
    model.eval() 

    # Let's create a dummy input tensor  
    dummy_input = torch.randn(1, input_dim, requires_grad=True)	

    make_dir("pre-train-onnx")
    # Export the model	
    torch.onnx.export(model,         # model being run 
         dummy_input,       # model input (or a tuple for multiple inputs) 
         "pre-train-onnx/model" + str(uid) + ".onnx",       # where to save the model  
         export_params=True,  # store the trained parameter weights inside the model file 
         opset_version=10,    # the ONNX version to export the model to 
         do_constant_folding=True,  # whether to execute constant folding for optimization 
         input_names = ['modelInput'],   # the model's input names 
         output_names = ['modelOutput'], # the model's output names 
         dynamic_axes={'modelInput' : {0 : 'batch_size'},    # variable length axes 
                                'modelOutput' : {0 : 'batch_size'}}) 
    print(" ") 
    print("Model has been converted to ONNX : model" + str(uid) + ".onnx")

# MAIN
# number of features (len of X cols)
input_dim = 3
# number of hidden layers
hidden_layers = [25, 50, 25]
# number of classes (unique of y)
output_dim = 5

num_uids = ai_utl.count_files_in_directory("processed_uids/data")
for uid in range(num_uids):
    number_top_app = ori_number_top_app
    if args.eval_single_uid:
        uid = args.single_uid_num

    
    # 파일 존재 여부 체크
    if not os.path.exists(f'processed_uids/data/{uid}.txt'):
        continue  # 파일이 없으면 루프의 다음 반복으로 넘어갑니다.

    # 파일이 존재하면 JSON 데이터 로드
    with open(f'processed_uids/data/{uid}.txt', 'r') as file:
        data = load_json(f'processed_uids/data/{uid}.txt')

    length_data = len(data)
    if length_data < 10:
    	# skip uid with less than 10 data samples
        continue
    list_of_apps = []
    for i in data:
        if i[1] not in list_of_apps:
            list_of_apps.append(i[1])

    print("Number of Sample is: {}".format(length_data))
    output_dim = len(list_of_apps)
    print(f"Number of Classes is {output_dim}")
    path = "pre-train/model"+str(uid)+".pth" 

    # 파일 존재 여부 체크
    if not os.path.exists(path):
        continue  # 파일이 없으면 루프의 다음 반복으로 넘어갑니다.

    # Let's load the model we just created and test the accuracy per label 
    model = Network()

    # model.load_state_dict(torch.load(path), strict=False) 
    model.load_state_dict(torch.load(path), strict=False) # 일치하지 않는 키를 무시하기 위해 strict=False 사용

    # Conversion to ONNX 
    Convert_ONNX()
    if args.eval_single_uid:
        break

print("--- DONE ---")
```

```bash
// Conda 실행 해야함. (README.md 참고)
$ python Pth_to_Onnx.py --eval_single_uid 0 --top_app 5
```

## 3) Android App 구현

### 3.1 Gradle 설정
참고 : [Install ONNX Runtime](https://onnxruntime.ai/docs/install/#install-on-web-and-mobile) 
1. build.gradle(:app) 에 dependencies 추가

![image](https://github.com/SubingJeong/Studying/assets/74831049/77751514-a6e4-471a-8828-fc4138410c68)


```groovy
dependencies {
    implementation 'com.microsoft.onnxruntime:onnxruntime-android:latest.release'
}
```

1. Import

```java
import ai.onnxruntime.*;
```

1. Sample code

```java
import ai.onnxruntime.*;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // ONNX Runtime 환경 설정
        OrtEnvironment env = OrtEnvironment.getEnvironment();

        try {
            // 모델 로드
            String modelPath = getExternalFilesDir(null) + "/model.onnx";
            OnnxTensor modelTensor = OnnxTensor.createTensorFromModelFile(modelPath);
            

            // TODO: 실제 입력 데이터로 채우기
            // 입력 데이터 생성 (이 예시에서는 무작위 값으로 생성)
            float[] inputData = new float[224 * 224 * 3];
            
            // ONNX 모델 실행
            OnnxTensor inputTensor = OnnxTensor.createTensor(env, inputData);
            OnnxTensor outputTensor = env.run(modelTensor, new String[]{"output"});
            
            // 결과 출력 (이 예시에서는 단순히 출력 타입만 출력)
            System.out.println(outputTensor.getInfo().getType());
        } catch (OrtException e) {
            e.printStackTrace();
        }
    }
}

```
