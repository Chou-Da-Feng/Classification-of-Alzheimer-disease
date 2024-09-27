# Classification-of-Alzheimer-disease

## 摘要

本研究旨在利用深度學習方法對阿茲海默症患者進行分類，探索分析不同深度學習模型在分類任務中的表現。
本研究收集了來自[ADNI](https://ida.loni.usc.edu/home/projectPage.jsp?project=ADNI)醫療公開數據庫來源的MRI影像數據，並使用多種深度學習方法進行分析。具體而言，我們應用了[Keras](https://keras.io/api/applications/)提供的卷積神經網路（CNN）的預訓練模型，以提高分類的準確性和穩定性。
在數據預處理階段，我們對數據進行了標準化處理，並且用[OpenCV](https://opencv.org/)此套件對數據影像進行切割腦室與海馬體。實驗結果表明，基於深度學習的方法在阿茲海默症分類中具有顯著的優勢，能夠有效區分阿茲海默症患者。


## 資料前處理
1. 讀取影像數據集與對應的標籤。  
    + 標籤 : 標籤分別有CN(健康大腦),MCI(輕度癡呆),AD(患有阿茲海默症大腦)。  

    + 影像切片 : 每筆原始影像均是三維的MRI影像，下圖為根據不同切面視覺化的影像切片。左圖:水平面,中間圖:冠狀面,右圖:矢狀面(縱切面)。

    ![image_origin](/README_resource/image_origin.jpg)

2. 找出腦室,海馬體的位置。


3. 利用OpenCV與numpy切除掉不必要的影像，並且把數據標準化，下圖為一筆腦室數據切割結果  
![crop_image_slide](/README_resource/crop_image_slide.jpg)

4. 再從中挑選出三張，進行灰階化處理，然後將這三張切片疊加，產生一張綜合的影像資料。

5. 最後，將切片調整大小為(256,256)並保存圖像。

## 模型選取

本研究中使用的神經網路結構基於Keras提供的卷積神經網路（CNN）預訓練模型，並在其上添加自定義的全連接層（denselayer）進行微調。  

在本研究中，我們採用了Keras提供常見的卷積神經網路預訓練模型，[VGG19](https://keras.io/api/applications/vgg/#vgg19-function)、[ResNet50](https://keras.io/api/applications/resnet/#resnet50-function)、[InceptionV3](https://keras.io/api/applications/inceptionv3/)，[MobileNetV2](https://keras.io/api/applications/mobilenet/#mobilenetv2-function) 這四種預訓練模型。這些預訓練模型已在大型數據集上進行了訓練，具有良好的特徵提取能力和泛化性能。

## 自定義架構
本方法採用了集成式學習。首先，從原始數據中分別提取出腦室和海馬體的相關信息，接著使用袋裝法對數據進行抽取。預訓練模型分別學習腦室和海馬體的特徵（下圖中所示的Ventricle Model 1 至 4 和Hippocampus Model 1 至 4）。隨後，通過堆疊法將這些學習好的模型結合起來，以獲得預測值。最後，使用加權平均法將腦室和海馬體模型堆疊後的預測結果融合，得出最終的總預測。
![net](/README_resource/net4.png)

## 實驗設計
我們在訓練神經網絡的前250個epochs期間，保持預訓練模型卷積層的權重不變，只更新自定義全連接層的權重。這樣可以加速模型的訓練過程，同時利用預訓練模型已經學習到的特徵。訓練完250個epochs後，我們再訓練100個epochs來微調預訓練模型的後幾層。數據量每一個標籤(CN,MCI,AD)均為3300筆資料。
本研究使用的優化器是Adam（AdaptiveMomentEstimation）。Adam優化器結
合了RMSProp和SGDwithmomentum的優點，能夠自適應地調整學習率，提高訓練速度和穩定性。它通過計算一階和二階動量來更新權重，從而更快地收斂到全局最優解。
