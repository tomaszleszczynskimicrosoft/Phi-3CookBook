# **AI PCでのPhi-3推論**

生成AIの進化とエッジデバイスのハードウェア能力の向上に伴い、ますます多くの生成AIモデルがユーザーのBYODデバイスに統合されるようになっています。AI PCもその一例です。2024年から、Intel、AMD、QualcommはPCメーカーと協力し、ハードウェアの改良を通じてローカル生成AIモデルの展開を促進するAI PCを導入します。このディスカッションでは、Intel AI PCに焦点を当て、Intel AI PCでPhi-3を展開する方法を探ります。

### NPUとは

NPU（Neural Processing Unit）は、ニューラルネットワークの操作やAIタスクを加速するために特別に設計されたプロセッサまたは処理ユニットです。一般的なCPUやGPUとは異なり、NPUはデータ駆動型の並列計算に最適化されており、大量のマルチメディアデータ（ビデオや画像など）やニューラルネットワークのデータ処理に非常に効率的です。特に音声認識、ビデオ通話の背景ぼかし、オブジェクト検出などの写真やビデオ編集プロセスなどのAI関連タスクに優れています。

## NPU vs GPU

多くのAIや機械学習のワークロードはGPUで実行されますが、GPUとNPUには重要な違いがあります。GPUは並列計算能力で知られていますが、すべてのGPUがグラフィックス処理以外の効率が同じとは限りません。一方、NPUはニューラルネットワーク操作に関わる複雑な計算のために特別に設計されており、AIタスクに非常に効果的です。

まとめると、NPUはAI計算を加速する数学の天才であり、AI PCの新しい時代において重要な役割を果たしています！

***この例はIntelの最新のIntel Core Ultraプロセッサに基づいています***

## **1. NPUを使用してPhi-3モデルを実行する**

Intel® NPUデバイスは、IntelクライアントCPUに統合されたAI推論アクセラレータであり、Intel® Core™ Ultra世代のCPU（以前はMeteor Lakeとして知られていた）から始まります。これにより、人工ニューラルネットワークタスクのエネルギー効率の良い実行が可能になります。

![Latency](../../../../translated_images/aipcphitokenlatency.eed732e4809ddb0ed39f84f7c305e0ad0083dfa88b79290779fdfeb1ecab90ba.ja.png)

![Latency770](../../../../translated_images/aipcphitokenlatency770.a232135bd174047d373410b265a60f29b122101366a08bea6391e39d76b369ad.ja.png)

**Intel NPU Acceleration Library**

Intel NPU Acceleration Library [https://github.com/intel/intel-npu-acceleration-library](https://github.com/intel/intel-npu-acceleration-library) は、Intel Neural Processing Unit (NPU) のパワーを活用して互換性のあるハードウェア上で高速計算を実行することで、アプリケーションの効率を向上させるためのPythonライブラリです。

Intel® Core™ Ultraプロセッサを搭載したAI PCでのPhi-3-miniの例。

![DemoPhiIntelAIPC](../../../../imgs/03/AIPC/aipcphi3-mini.gif)

pipを使ってPythonライブラリをインストールする

```bash

   pip install intel-npu-acceleration-library

```

***Note*** プロジェクトはまだ開発中ですが、リファレンスモデルはすでに非常に完成度が高いです。

### **Intel NPU Acceleration Libraryを使用してPhi-3を実行する**

Intel NPUアクセラレーションを使用することで、このライブラリは従来のエンコードプロセスに影響を与えません。元のPhi-3モデルをFP16、INT8、INT4などに量子化するためにこのライブラリを使用するだけで済みます。

```python

from transformers import AutoTokenizer, pipeline,TextStreamer
import intel_npu_acceleration_library as npu_lib
import warnings

model_id = "microsoft/Phi-3-mini-4k-instruct"

model = npu_lib.NPUModelForCausalLM.from_pretrained(
                                    model_id,
                                    torch_dtype="auto",
                                    dtype=npu_lib.int4,
                                    trust_remote_code=True
                                )

tokenizer = AutoTokenizer.from_pretrained(model_id)

text_streamer = TextStreamer(tokenizer, skip_prompt=True)

```
量子化が成功したら、NPUを呼び出してPhi-3モデルを実行します。

```python

generation_args = {
            "max_new_tokens": 1024,
            "return_full_text": False,
            "temperature": 0.3,
            "do_sample": False,
            "streamer": text_streamer,
        }

pipe = pipeline(
            "text-generation",
            model=model,
            tokenizer=tokenizer,
)

query = "<|system|>You are a helpful AI assistant.<|end|><|user|>Can you introduce yourself?<|end|><|assistant|>"

with warnings.catch_warnings():
    warnings.simplefilter("ignore")
    pipe(query, **generation_args)


```

コードを実行すると、タスクマネージャーを通じてNPUの動作状況を確認できます。

![NPU](../../../../translated_images/aipc_NPU.5995e81d09fc503ab2c3b4d17954a9a68ff46f8f8ce62957344c4957baf105e6.ja.png)

***Samples*** : [AIPC_NPU_DEMO.ipynb](../../../../code/03.Inference/AIPC/AIPC_NPU_DEMO.ipynb)

## **2. DirectML + ONNX Runtimeを使用してPhi-3モデルを実行する**

### **DirectMLとは**

[DirectML](https://github.com/microsoft/DirectML) は、機械学習のための高性能でハードウェアアクセラレートされたDirectX 12ライブラリです。DirectMLは、AMD、Intel、NVIDIA、QualcommなどのベンダーからのすべてのDirectX 12対応GPUを含む、広範なサポートハードウェアとドライバで一般的な機械学習タスクのGPUアクセラレーションを提供します。

単独で使用される場合、DirectML APIは低レベルのDirectX 12ライブラリであり、フレームワーク、ゲーム、その他のリアルタイムアプリケーションなどの高性能で低遅延のアプリケーションに適しています。DirectMLのDirect3D 12とのシームレスな相互運用性、その低オーバーヘッド、およびハードウェア間の一貫性は、高性能が求められ、かつハードウェア間での結果の信頼性と予測可能性が重要な場合に、機械学習を加速するための理想的な選択肢です。

***Note*** : 最新のDirectMLはすでにNPUをサポートしています (https://devblogs.microsoft.com/directx/introducing-neural-processor-unit-npu-support-in-directml-developer-preview/)

### DirectMLとCUDAの能力とパフォーマンスの比較：

**DirectML** はMicrosoftが開発した機械学習ライブラリです。Windowsデバイス（デスクトップ、ラップトップ、エッジデバイスなど）で機械学習ワークロードを加速するために設計されています。
- DX12ベース: DirectMLはDirectX 12 (DX12) の上に構築されており、NVIDIAとAMDの両方を含むGPU全体で幅広いハードウェアサポートを提供します。
- 幅広いサポート: DX12を活用するため、DirectMLはDX12をサポートする任意のGPUで動作し、統合GPUでも動作します。
- 画像処理: DirectMLはニューラルネットワークを使用して画像やその他のデータを処理し、画像認識、オブジェクト検出などのタスクに適しています。
- セットアップの容易さ: DirectMLのセットアップは簡単で、特定のSDKやGPUメーカーのライブラリを必要としません。
- パフォーマンス: 場合によっては、DirectMLは良好なパフォーマンスを発揮し、特定のワークロードではCUDAよりも高速になることがあります。
- 制限: しかし、DirectMLは特にfloat16の大きなバッチサイズでは遅くなることがあります。

**CUDA** はNVIDIAの並列計算プラットフォームおよびプログラミングモデルです。開発者はNVIDIA GPUのパワーを活用して、機械学習や科学シミュレーションなどの一般的な計算を行うことができます。
- NVIDIA専用: CUDAはNVIDIA GPUと緊密に統合されており、特にNVIDIA GPU用に設計されています。
- 高度に最適化: GPUアクセラレートされたタスクに優れたパフォーマンスを提供し、特にNVIDIA GPUを使用する場合に最適です。
- 広く使用されている: 多くの機械学習フレームワークやライブラリ（TensorFlowやPyTorchなど）がCUDAをサポートしています。
- カスタマイズ: 開発者は特定のタスクのためにCUDA設定を微調整でき、最適なパフォーマンスを実現できます。
- 制限: しかし、CUDAはNVIDIAハードウェアに依存しているため、異なるGPU間での互換性を求める場合には制限があります。

### DirectMLとCUDAの選択

DirectMLとCUDAの選択は、具体的な使用ケース、ハードウェアの可用性、および好みに依存します。
より広範な互換性とセットアップの容易さを求める場合は、DirectMLが適しているかもしれません。しかし、NVIDIA GPUを持ち、高度に最適化されたパフォーマンスが必要な場合は、CUDAが強力な選択肢です。要約すると、DirectMLとCUDAの両方に強みと弱みがあるため、選択する際には要件と利用可能なハードウェアを考慮してください。

### **ONNX Runtimeによる生成AI**

AIの時代において、AIモデルの移植性は非常に重要です。ONNX Runtimeを使用すると、訓練されたモデルを簡単に異なるデバイスに展開できます。開発者は推論フレームワークに注意を払う必要がなく、統一されたAPIを使用してモデル推論を完了できます。生成AIの時代においても、ONNX Runtimeはコードの最適化を行っています (https://onnxruntime.ai/docs/genai/)。最適化されたONNX Runtimeを通じて、量子化された生成AIモデルを異なる端末で推論できます。生成AIをONNX Runtimeで実行する場合、Python、C#、C / C++を通じてAIモデルAPIを推論できます。もちろん、iPhoneへの展開はC++のGenerative AI with ONNX Runtime APIを活用できます。

[サンプルコード](https://github.com/Azure-Samples/Phi-3MiniSamples/tree/main/onnx)

***ONNX Runtimeライブラリで生成AIをコンパイル***

```bash

winget install --id=Kitware.CMake  -e

git clone https://github.com/microsoft/onnxruntime.git

cd .\onnxruntime\

./build.bat --build_shared_lib --skip_tests --parallel --use_dml --config Release

cd ../

git clone https://github.com/microsoft/onnxruntime-genai.git

cd .\onnxruntime-genai\

mkdir ort

cd ort

mkdir include

mkdir lib

copy ..\onnxruntime\include\onnxruntime\core\providers\dml\dml_provider_factory.h ort\include

copy ..\onnxruntime\include\onnxruntime\core\session\onnxruntime_c_api.h ort\include

copy ..\onnxruntime\build\Windows\Release\Release\*.dll ort\lib

copy ..\onnxruntime\build\Windows\Release\Release\onnxruntime.lib ort\lib

python build.py --use_dml


```

**ライブラリのインストール**

```bash

pip install .\onnxruntime_genai_directml-0.3.0.dev0-cp310-cp310-win_amd64.whl

```

実行結果は次の通りです

![DML](../../../../translated_images/aipc_DML.311f483cb2951360febbe203ff1f8d66049cbaaafdfa33d06f1f201167548191.ja.png)

***Samples*** : [AIPC_DirectML_DEMO.ipynb](../../../../code/03.Inference/AIPC/AIPC_DirectML_DEMO.ipynb)

## **3. Intel OpenVinoを使用してPhi-3モデルを実行する**

### **OpenVINOとは**

[OpenVINO](https://github.com/openvinotoolkit/openvino) は、ディープラーニングモデルの最適化と展開のためのオープンソースツールキットです。TensorFlow、PyTorchなどの人気のあるフレームワークからのビジョン、オーディオ、および言語モデルのディープラーニングパフォーマンスを向上させます。OpenVINOを使い始めましょう。OpenVINOは、CPUとGPUを組み合わせてPhi3モデルを実行することもできます。

***Note***: 現在、OpenVINOはNPUをサポートしていません。

### **OpenVINOライブラリのインストール**

```bash

 pip install git+https://github.com/huggingface/optimum-intel.git

 pip install git+https://github.com/openvinotoolkit/nncf.git

 pip install openvino-nightly

```

### **OpenVINOでPhi-3を実行する**

NPUと同様に、OpenVINOは量子化モデルを実行することで生成AIモデルの呼び出しを完了します。まずPhi-3モデルを量子化し、コマンドラインを通じてoptimum-cliでモデルの量子化を完了します。

**INT4**

```bash

optimum-cli export openvino --model "microsoft/Phi-3-mini-4k-instruct" --task text-generation-with-past --weight-format int4 --group-size 128 --ratio 0.6  --sym  --trust-remote-code ./openvinomodel/phi3/int4

```

**FP16**

```bash

optimum-cli export openvino --model "microsoft/Phi-3-mini-4k-instruct" --task text-generation-with-past --weight-format fp16 --trust-remote-code ./openvinomodel/phi3/fp16

```

変換された形式は次のようになります

![openvino_convert](../../../../translated_images/aipc_OpenVINO_convert.57010ce04f9c100fa55b9762e934818e663ec993f1bd026429c48fb9a65811ad.ja.png)

OVModelForCausalLMを通じてモデルパス（model_dir）、関連する設定（ov_config = {"PERFORMANCE_HINT": "LATENCY", "NUM_STREAMS": "1", "CACHE_DIR": ""}）、およびハードウェアアクセラレートされたデバイス（GPU.0）をロードします。

```python

ov_model = OVModelForCausalLM.from_pretrained(
     model_dir,
     device='GPU.0',
     ov_config=ov_config,
     config=AutoConfig.from_pretrained(model_dir, trust_remote_code=True),
     trust_remote_code=True,
)

```

コードを実行すると、タスクマネージャーを通じてGPUの動作状況を確認できます。

![openvino_gpu](../../../../translated_images/aipc_OpenVINO_GPU.5e46f3572708832f1b6ea786cb0b0a99a1b662dfd6a860ac3477fb8cd5e64037.ja.png)

***Samples*** : [AIPC_OpenVino_Demo.ipynb](../../../../code/03.Inference/AIPC/AIPC_OpenVino_Demo.ipynb)

### ***Note*** : 上記の三つの方法にはそれぞれの利点がありますが、AI PC推論にはNPUアクセラレーションの使用をお勧めします。

**免責事項**:
この文書は機械ベースのAI翻訳サービスを使用して翻訳されています。正確さを期しておりますが、自動翻訳には誤りや不正確さが含まれる可能性があることをご承知おきください。権威ある情報源としては、原文の母国語の文書を考慮すべきです。重要な情報については、専門の人間による翻訳をお勧めします。この翻訳の使用に起因する誤解や誤解釈について、当社は一切の責任を負いません。