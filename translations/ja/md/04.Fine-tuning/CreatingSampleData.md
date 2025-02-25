# Hugging Faceからデータセットと関連画像をダウンロードして画像データセットを生成する

### 概要

このスクリプトは、必要な画像をダウンロードし、画像のダウンロードに失敗した行をフィルタリングし、データセットをCSVファイルとして保存することで、機械学習用のデータセットを準備します。

### 前提条件

このスクリプトを実行する前に、以下のライブラリがインストールされていることを確認してください: `Pandas`, `Datasets`, `requests`, `PIL`, `io`。また、2行目の`'Insert_Your_Dataset'`をHugging Faceからのデータセット名に置き換える必要があります。

必要なライブラリ:

```python

import os
import pandas as pd
from datasets import load_dataset
import requests
from PIL import Image
from io import BytesIO
```

### 機能

スクリプトは以下の手順を実行します:

1. Hugging Faceからデータセットを`load_dataset()` function.
2. Converts the Hugging Face dataset to a Pandas DataFrame for easier manipulation using the `to_pandas()` method.
3. Creates directories to save the dataset and images.
4. Filters out rows where image download fails by iterating through each row in the DataFrame, downloading the image using the custom `download_image()` function, and appending the filtered row to a new DataFrame called `filtered_rows`.
5. Creates a new DataFrame with the filtered rows and saves it to disk as a CSV file.
6. Prints a message indicating where the dataset and images have been saved.

### Custom Function

The `download_image()`関数は、URLから画像をダウンロードし、Pillow Image Library (PIL)と`io`モジュールを使用してローカルに保存します。画像が正常にダウンロードされた場合はTrueを返し、そうでない場合はFalseを返します。リクエストが失敗した場合はエラーメッセージを含む例外を発生させます。

### どのように動作するか

download_image関数は2つのパラメータを取ります: image_urlはダウンロードする画像のURLで、save_pathはダウンロードした画像を保存するパスです。

関数の動作は以下の通りです:

最初に、requests.getメソッドを使用してimage_urlにGETリクエストを送信します。これにより、URLから画像データが取得されます。

response.raise_for_status()行は、リクエストが成功したかどうかを確認します。レスポンスのステータスコードがエラーを示す場合（例: 404 - Not Found）、例外を発生させます。これにより、リクエストが成功した場合のみ画像のダウンロードを進めることが保証されます。

次に、画像データをPIL (Python Imaging Library) モジュールのImage.openメソッドに渡します。このメソッドは、画像データからImageオブジェクトを作成します。

image.save(save_path)行は、画像を指定されたsave_pathに保存します。save_pathには、希望するファイル名と拡張子を含める必要があります。

最後に、関数は画像が正常にダウンロードおよび保存されたことを示すためにTrueを返します。プロセス中に例外が発生した場合は、例外をキャッチして失敗を示すエラーメッセージを表示し、Falseを返します。

この関数は、URLから画像をダウンロードしてローカルに保存するのに役立ちます。ダウンロードプロセス中の潜在的なエラーを処理し、ダウンロードが成功したかどうかのフィードバックを提供します。

requestsライブラリはHTTPリクエストを行うために使用され、PILライブラリは画像を操作するために使用され、BytesIOクラスは画像データをバイトストリームとして扱うために使用されます。

### 結論

このスクリプトは、必要な画像をダウンロードし、画像のダウンロードに失敗した行をフィルタリングし、データセットをCSVファイルとして保存することで、機械学習用のデータセットを準備する便利な方法を提供します。

### サンプルスクリプト

```python
import os
import pandas as pd
from datasets import load_dataset
import requests
from PIL import Image
from io import BytesIO

def download_image(image_url, save_path):
    try:
        response = requests.get(image_url)
        response.raise_for_status()  # Check if the request was successful
        image = Image.open(BytesIO(response.content))
        image.save(save_path)
        return True
    except Exception as e:
        print(f"Failed to download {image_url}: {e}")
        return False


# Download the dataset from Hugging Face
dataset = load_dataset('Insert_Your_Dataset')


# Convert the Hugging Face dataset to a Pandas DataFrame
df = dataset['train'].to_pandas()


# Create directories to save the dataset and images
dataset_dir = './data/DataSetName'
images_dir = os.path.join(dataset_dir, 'images')
os.makedirs(images_dir, exist_ok=True)


# Filter out rows where image download fails
filtered_rows = []
for idx, row in df.iterrows():
    image_url = row['imageurl']
    image_name = f"{row['product_code']}.jpg"
    image_path = os.path.join(images_dir, image_name)
    if download_image(image_url, image_path):
        row['local_image_path'] = image_path
        filtered_rows.append(row)


# Create a new DataFrame with the filtered rows
filtered_df = pd.DataFrame(filtered_rows)


# Save the updated dataset to disk
dataset_path = os.path.join(dataset_dir, 'Dataset.csv')
filtered_df.to_csv(dataset_path, index=False)


print(f"Dataset and images saved to {dataset_dir}")
```

### サンプルコードのダウンロード 
[新しいデータセット生成スクリプト](../../../../code/04.Finetuning/generate_dataset.py)

### サンプルデータセット
[LORAを使ったファインチューニングのサンプルデータセット例](../../../../code/04.Finetuning/olive-ort-example/dataset/dataset-classification.json)

**免責事項**:
この文書は機械ベースのAI翻訳サービスを使用して翻訳されています。正確さを期していますが、自動翻訳にはエラーや不正確さが含まれる場合があることをご承知おきください。元の言語での原文を権威ある情報源とみなすべきです。重要な情報については、専門の人間による翻訳をお勧めします。この翻訳の使用によって生じる誤解や誤認について、当社は一切の責任を負いません。