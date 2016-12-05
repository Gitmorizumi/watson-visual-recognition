## IBM Watsonで画像を学習させてみよう
はじめに
1. IBM Bluemixのアカウントを作成してください。[アカウントの作成方法はこちら。](https://youtu.be/rdudF22X68Q?list=PLSWt5tGb2SKzefQ3xnDULIrvTy2wVj0h1)

2. curlをインストールしてください。


***
### 概要
IBM Watsonのサービスの一つであるVisual Recognition APIを使って画像を学習させるチュートリアルです。
IBM Watsonにカレーやハヤシライスやカツカレーが学習できるか挑戦してみるチュートリアルです。
***
## 1.BluemixでNode.jsサーバーを立てる
[サーバーの立て方はこちらをご参考。](https://youtu.be/n3muCRup9MI?list=PLSWt5tGb2SKzefQ3xnDULIrvTy2wVj0h1)

## 2.WatsonのAPIをバインドする
アプリ一覧の右上のアプリ作成をクリック。
![1](readme_images/1.png)

カタログメニューの左側からWatsonを選択。
![2](readme_images/2.png)

メニューからVisual Recognitionを選択。
![3](readme_images/3.png)

接続するアプリケーションを選択。その後、右下の作成ボタンをクリック。
![4](readme_images/4.png)

アプリを最ステージ後、アプリケーションのランタイムメニューから環境変数を選択。
WatsonのAPIを呼び出す際にapi_keyを使うのでメモっておく。
![5](readme_images/5.png)


## 3.画像を学習する
ひとまずサンプルのカレー画像が認識されるかWatsonくんに聞いてみます。

Terminalを立ち上げ画像を解凍したフォルダから以下のコマンドでテストしてみます。


```
curl -X POST -F "images_file=@test_curry.jpg" "https://gateway-a.watsonplatform.net/visual-recognition/api/v3/classify?api_key=1e4712a0201aa48d643616cebb5940678784880b&version=2016-05-20"
```
結果のほどは・・・。

```
{
    "custom_classes": 0,
    "images": [
        {
            "classifiers": [
                {
                    "classes": [
                        {
                            "class": "food",
                            "score": 0.973403
                        }
                    ],
                    "classifier_id": "default",
                    "name": "default"
                }
            ],
            "image": "images8.jpg"
        }
    ],
    "images_processed": 1
}
```

ほー・・・。foodとな。そう来るか・・・Watsonくん。ということで、Watsonにカレー大食い選手権に出てもらいましょう。

```
curl -X POST -F "curry_positive_examples=@Curry.zip" -F "negative_examples=@Others.zip" -F "name=food" "https://gateway-a.watsonplatform.net/visual-recognition/api/v3/classifiers?api_key=1e4712a0201aa48d643616cebb5940678784880b&version=2016-05-20"
```

吐き出されたWatsonのレスポンス。どうやらカレーという存在を認識したようだ。

```
{
    "classifier_id": "food_733278282",
    "name": "food",
    "owner": "4febae1a-a8e1-43d4-9597-2ae7a48734d9",
    "status": "training",
    "created": "2016-12-05T17:43:13.433Z",
    "classes": [{"class": "curry"}]
}
```
進行状況をチェックしてみます。

```
curl -X GET "https://gateway-a.watsonplatform.net/visual-recognition/api/v3/classifiers/food_733278282?api_key=1e4712a0201aa48d643616cebb5940678784880b&version=2016-05-20"
```
では、さっそく試してみましょう。この識別クラスを使うための設定ファイルを使用しするために以下のようjsonファイルをつくります。

```
{"classifier_ids": ["food_733278282"], "threshold":0.2 }
```

それでは、この識別クラスを使ってさっそく認識できるかテストしてみます。Curryというクラスが高得点で出力されれば成功です！

```
curl -X POST -F "images_file=@test_curry.jpg" -F "parameters=@params-curry.json" "https://gateway-a.watsonplatform.net/visual-recognition/api/v3/classify?api_key=1e4712a0201aa48d643616cebb5940678784880b&version=2016-05-20"
```
果たして結果は・・。

```
{
    "custom_classes": 1,
    "images": [
        {
            "classifiers": [
                {
                    "classes": [
                        {
                            "class": "food",
                            "score": 0.992608
                        },
                        {
                            "class": "gravy",
                            "score": 0.268941,
                            "type_hierarchy": "/foods/sauces/gravy"
                        }
                    ],
                    "classifier_id": "default",
                    "name": "default"
                },
                {
                    "classes": [
                        {
                            "class": "curry",
                            "score": 0.64824
                        }
                    ],
                    "classifier_id": "food_733278282",
                    "name": "food"
                }
            ],
            "image": "test_curry.jpg"
        }
    ],
    "images_processed": 1
}
```
Curryのスコアが0.648という高得点を叩き出しました！

## 3.ハヤシライスとの区別はできるかWatsonくん

ハヤシライスの大食い選手権に出て頂きましょう！Food in!!

識別クラスは1つしかないので、さきほど作成したクラスを一旦削除します。

```
curl -X DELETE "https://gateway-a.watsonplatform.net/visual-recognition/api/v3/classifiers/food_733278282?api_key=1e4712a0201aa48d643616cebb5940678784880b&version=2016-05-20"
```


```
curl -X POST -F "curry_positive_examples=@Curry.zip" -F "hayashi_positive_examples=@Hayashi.zip" -F "katsu_curry_positive_examples=@KatsuC.zip" -F "negative_examples=@Others.zip" -F "name=food" "https://gateway-a.watsonplatform.net/visual-recognition/api/v3/classifiers?api_key=1e4712a0201aa48d643616cebb5940678784880b&version=2016-05-20"
```
どうだ。

```
{
    "classifier_id": "food_341155540",
    "name": "food",
    "owner": "4febae1a-a8e1-43d4-9597-2ae7a48734d9",
    "status": "training",
    "created": "2016-12-05T18:26:00.192Z",
    "classes": [
        {"class": "katsu_curry"},
        {"class": "hayashi"},
        {"class": "curry"}
    ]
}
```

```
curl -X GET "https://gateway-a.watsonplatform.net/visual-recognition/api/v3/classifiers/food_341155540?api_key=1e4712a0201aa48d643616cebb5940678784880b&version=2016-05-20"
```

```
{
    "classifier_id": "food_341155540",
    "name": "food",
    "owner": "4febae1a-a8e1-43d4-9597-2ae7a48734d9",
    "status": "ready",
    "created": "2016-12-05T18:26:00.192Z",
    "classes": [
        {"class": "katsu_curry"},
        {"class": "hayashi"},
        {"class": "curry"}
    ]
}
```


クラスもready状態なのでさっそく試してみましょう。この識別クラスを使うための設定ファイルを使用するために以下のようjsonファイルを編集します。

```
{"classifier_ids": ["food_341155540","default"], "threshold":0.2 }
```

```
curl -X POST -F "images_file=@test_curry.jpg" -F "parameters=@params-new.json" "https://gateway-a.watsonplatform.net/visual-recognition/api/v3/classify?api_key=1e4712a0201aa48d643616cebb5940678784880b&version=2016-05-20"
```
実行結果は
```
{
    "custom_classes": 3,
    "images": [
        {
            "classifiers": [
                {
                    "classes": [
                        {
                            "class": "food",
                            "score": 0.992608
                        },
                        {
                            "class": "gravy",
                            "score": 0.268941,
                            "type_hierarchy": "/foods/sauces/gravy"
                        }
                    ],
                    "classifier_id": "default",
                    "name": "default"
                },
                {
                    "classes": [
                        {
                            "class": "curry",
                            "score": 0.858555
                        },
                        {
                            "class": "katsu_curry",
                            "score": 0.293688
                        }
                    ],
                    "classifier_id": "food_341155540",
                    "name": "food"
                }
            ],
            "image": "test_curry.jpg"
        }
    ],
    "images_processed": 1
}
```

```
curl -X POST -F "images_file=@test_hayashi.jpg" -F "parameters=@params-new.json" "https://gateway-a.watsonplatform.net/visual-recognition/api/v3/classify?api_key=1e4712a0201aa48d643616cebb5940678784880b&version=2016-05-20"
```

```
{
    "custom_classes": 3,
    "images": [
        {
            "classifiers": [
                {
                    "classes": [
                        {
                            "class": "food",
                            "score": 0.973403
                        },
                        {
                            "class": "dinner",
                            "score": 0.28905,
                            "type_hierarchy": "/activities/events/dinner"
                        },
                        {
                            "class": "restaurant",
                            "score": 0.268941,
                            "type_hierarchy": "/organizations/businesses/restaurant"
                        }
                    ],
                    "classifier_id": "default",
                    "name": "default"
                },
                {
                    "classes": [
                        {
                            "class": "hayashi",
                            "score": 0.95124
                        }
                    ],
                    "classifier_id": "food_341155540",
                    "name": "food"
                }
            ],
            "image": "test_hayashi.jpg"
        }
    ],
    "images_processed": 1
}
```


```
curl -X POST -F "images_file=@test_katsuc.jpg" -F "parameters=@params-new.json" "https://gateway-a.watsonplatform.net/visual-recognition/api/v3/classify?api_key=1e4712a0201aa48d643616cebb5940678784880b&version=2016-05-20"
```

```
{
    "custom_classes": 3,
    "images": [
        {
            "classifiers": [
                {
                    "classes": [
                        {
                            "class": "food",
                            "score": 0.973403
                        }
                    ],
                    "classifier_id": "default",
                    "name": "default"
                },
                {
                    "classes": [
                        {
                            "class": "katsu_curry",
                            "score": 0.908661
                        }
                    ],
                    "classifier_id": "food_341155540",
                    "name": "food"
                }
            ],
            "image": "test_katsuc.jpg"
        }
    ],
    "images_processed": 1
}
```
