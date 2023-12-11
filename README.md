# 智能客服對話系統(Github版本)

## 寫規格書
>[excel](https://docs.google.com/spreadsheets/d/11xuymKi_N9tnzYNH91TL-1xPYTQeXWQXa8ySI2whLpY/edit#gid=0)

>需求 : 在客服對話方面，訓練一個生成式模型，該模型需要具備地政士法律專業知識，以盡可能提高準確性，確保模型能夠正確回答使用者的問題。 最後，將此與Line Bot的API進行串接，使其能夠在Line官方帳號頻道上提供服務。
時程 : 
總預算 : 預算5萬左右
人力需求 :
(1). 網路爬蟲蒐集資料
(2). 法律專業檢查資料
(3). 資料預處理&訓練並調整NLG模型&測試模型
(4). Line Bot開發者
(5). 資料庫開發者
(6). API 開發者

- 法律專業檢查資料
    - (1).  蒐集資料者一名
    - (2).  法律專業知識者一名
    - (3).  NLP領域者一名(或多名)
- 轉接功能: 通常是回答完之後，詢問客人是否滿意，不滿意就要轉接人工


## 資料格式
1. 對話資料
    - 有關智能客服的資料準備，我認為的規格是，每筆問答資料希望平均在2-5次的對話輪數(多一點也可以)，以使用者和客服開頭。
    - 格式預想是如下面範例(輪數為6)(by chatGPT):
    ```=
    使用者：我想諮詢關於土地買賣合約的問題。
    客服：當然，請告訴我您的具體問題。
    使用者：在土地交易合約中，需要包括哪些重要條款？
    客服：土地交易合約通常包括買賣雙方的資訊、土地的詳細描述、價格、付款方式、交貨日期等重要條款。
    使用者：如果我想要解除土地交易合同，我應該如何操作？
    客服：土地交易合約的解除通常需要根據合約中的解除條款進行操作，同時需要符合台灣法律的規定。 您需要提供合約詳情以獲得更多協助。
    ```
    - 主題是指地政士常見文題
    - [國泰世華客服](https://www.cathaybk.com.tw/ChatWeb/chat?traceType=branchId&traceValue=Chatweb)
2. 法律資料
    - [地政士法](https://law.moj.gov.tw/LawClass/LawAll.aspx?pcode=D0060081&kw=%e5%9c%b0%e6%94%bf%e5%a3%ab)
    - [地政士簽證責任及簽證基金管理辦法](https://law.moj.gov.tw/LawClass/LawAll.aspx?pcode=D0060084&kw=%e5%9c%b0%e6%94%bf%e5%a3%ab)
    - [地政士法施行細則](https://law.moj.gov.tw/LawClass/LawAll.aspx?pcode=D0060082&kw=%e5%9c%b0%e6%94%bf%e5%a3%ab)
    - [地政士及不動產經紀業防制洗錢及打擊資恐辦法](https://law.moj.gov.tw/LawClass/LawAll.aspx?pcode=D0060124&kw=%e5%9c%b0%e6%94%bf%e5%a3%ab)
3. 其他延伸閱讀資料
    - [Amazing talker 的AI使用](https://www.youtube.com/watch?v=KFsSAEV1_vk)
    - 要多少的資料才能足夠?
        - GPT3使用的資料量:
        - 499 億 token，49900000000/3 = *16,633,333,333.33* 
            - 166億個中文字 = 16,633,333,333.33 / 3188000 = *5,217.48*  

## 串接AI API

### LLAMA
- [FastAPI](https://colab.research.google.com/github/LiuYuWei/Llama-2-cpp-example/blob/main/Llama_2_FastAPI_Service_Colab_Example.ipynb)
- [huggingface的clone教學](https://huggingface.co/welcome)
- zsh
- conda env list
- conda create -n (改成你想要的環境名稱) python=(改成你要的版本號)
    - conda create -n TW_LLAMA python=3.10
- source activate TW_LLAMA
- pip install fastapi nest-asyncio pyngrok uvicorn accelerate transformers
- CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 pip install llama-cpp-python==0.1.77 --force-reinstall --upgrade --no-cache-dir --verbose
- pip install huggingface_hub

### 加速生成

- huggingface-cli login
    - 輸入 accsess token
- git clone https://huggingface.co/yentinglin/Taiwan-LLM-13B-v2.0-chat
    - 主要是前面是用GGML，所以不能用到GPU﹐改成使用這個可以使用 huggingface 裡面的 transformer
    - 下載過程也要一直貼上 accsess token
    - 在 text-generation 的使用時後會產生 cuda out of memory error，所以改成使用7B的吧
- git clone https://huggingface.co/yentinglin/Taiwan-LLM-7B-v2.1-chat
    - 要先下載，下面的text-generation才能用
- 使用 text-generation
    - [url](https://github.com/huggingface/text-generation-inference)
    - 先設定
        -  model=yentinglin/Taiwan-LLM-7B-v2.1-chat
        -  volume=$PWD/data
            -  echo $volume 可以看到剛剛設了什麼值給 volume
        -  token=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
        -  docker run --gpus all --shm-size 1g -e HUGGING_FACE_HUB_TOKEN=$token -p 8080:80 -v $volume:/data ghcr.io/huggingface/text-generation-inference:1.2 --model-id $model
### 包裝request 
- [LangChain framework](https://python.langchain.com/docs/get_started/introduction)
- `PromptTemplate` 提示模板: `from langchain.prompts import PromptTemplate`
- 再使用 `chain` 將 prompt 和 model 串起來
    - from langchain.chains import LLMChain
    - chain = LLMChain(llm=llm, prompt=prompt)
    - chain.run("有關購買不動產稅金，我需要了解哪些事情？")
- 其中LLAMA2和text-generation 使用 post 來跟 LangChain 互通
    - [url](https://huggingface.co/docs/text-generation-inference/quicktour)
- 問答（question answering） : 第二個重大的LangChain 使用用例。僅利用這些文件中的資訊來建立答案，回答特定文件中的問題。
    - [url](https://www.langchain.asia/use_cases/question_answering)
