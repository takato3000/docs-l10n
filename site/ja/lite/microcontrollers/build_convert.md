# モデルの構築と変換

マイクロコントローラの RAM とストレージは限られているため、機械学習モデルのサイズが制限されます。さらに、マイクロコントローラ向け TensorFlow Lite で現在サポートされている演算は限定されているため、すべてのモデルアーキテクチャは可能ではありません。

このドキュメントでは、TensorFlow モデルをマイクロコントローラで実行するように変換するプロセスについて説明します。また、サポートされている演算の概要と、限られたメモリに収まるようにモデルを設計およびトレーニングするためのガイダンスも提供します。

エンドツーエンドの実行可能なモデルの構築と変換の例については、次の Colab をご覧ください。これは <em>Hello World</em> の例の一部です。

train_hello_world_model.ipynb

## モデル変換

トレーニング済みの TensorFlow モデルをマイクロコントローラで実行するように変換するには、[ TensorFlow Lite コンバータ Python API ](https://www.tensorflow.org/lite/models/convert/)を使用する必要があります。これにより、モデルが[ `FlatBuffer`](https://google.github.io/flatbuffers/)に変換され、モデルのサイズが小さくなり、TensorFlow Lite 演算を使用するようにモデルが変更されます。

モデルサイズを最小化するには、[トレーニング後の量子化](https://www.tensorflow.org/lite/performance/post_training_quantization)の使用を検討してください。

### C 配列に変換

多くのマイクロコントローラプラットフォームは、ネイティブファイルシステムをサポートしていません。プログラムからモデルを使用する最も簡単な方法は、モデルを C 配列として組み込み、プログラムにコンパイルすることです。

次の unix コマンドは、TensorFlow Lite モデルを`char`配列として含む C ソースファイルを生成します。

```bash
xxd -i converted_model.tflite > model_data.cc
```

出力は以下のようになります。

```c
unsigned char converted_model_tflite[] = {
  0x18, 0x00, 0x00, 0x00, 0x54, 0x46, 0x4c, 0x33, 0x00, 0x00, 0x0e, 0x00,
  // <Lines omitted>
};
unsigned int converted_model_tflite_len = 18200;
```

ファイルを生成したら、プログラムに含めることができます。組み込みプラットフォームでのメモリ効率を向上させるには、配列宣言を`const`に変更することが重要です。

プログラムにモデルを含めて使用する方法の例については、*Hello World* の例の [`evaluate_test.cc`](https://github.com/tensorflow/tflite-micro/blob/main/tensorflow/lite/micro/examples/hello_world/evaluate_test.cc) をご覧ください。

## モデルのアーキテクチャとトレーニング

マイクロコントローラで使用するモデルを設計する場合、モデルのサイズ、ワークロード、および使用する演算を考慮することが重要です。

### モデルサイズ

モデルは、バイナリとしても実行時にも、プログラムの残りの部分と共にターゲットデバイスのメモリ内に収まるのに十分なサイズにする必要があります。

より小さなモデルを作成するためには、アーキテクチャで使用するレイヤーを少なくします。 ただし、小さいモデルは、適合不足になる可能性が高くなります。そのため、多くの問題では、メモリに収まる最大のモデルを使用してみてください。 ただし、より大きなモデルを使用すると、プロセッサのワークロードも増加します。

注意: マイクロコントローラ向け TensorFlow Lite のコアランタイムは、Cortex M3 の 16KB に収まります。

### ワークロード

モデルのサイズと複雑さは、ワークロードに影響を与えます。大規模で複雑なモデルでは、デューティサイクルが高くなる可能性があります。これは、デバイスのプロセッサがより多くの時間を費やしてアイドル時間が短縮されることを意味します。これにより、消費電力と熱出力が増加するためアプリによっては問題となる可能性があるます。

### 演算のサポート

マイクロコントローラ向け TensorFlow Lite は、現在、限られた TensorFlow 演算のサブセットをサポートしているため実行可能なモデルアーキテクチャが影響されますが、リファレンス実装と特定のアーキテクチャの最適化における演算のサポートの拡大に取り組んでいます。

サポートされている演算は、[`micro_mutable_ops_resolver.h`](https://github.com/tensorflow/tflite-micro/blob/main/tensorflow/lite/micro/micro_mutable_op_resolver.h) でご覧ください。
