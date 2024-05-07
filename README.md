# Fooood map 美食地圖實作
以搜尋引擎為關鍵字找尋相關文章後，具體的專題實作內容構思類似於下圖，大致上為分成四步：爬取餐廳資料、情緒分析處理、推薦系統建立、網站開發。

![image](https://github.com/yyc0314/Fooood-map/blob/62e93cd806abbfe381ca584f6ed947ad023370d3/img/%E6%90%9C%E5%B0%8B%E5%BC%95%E6%93%8E%E6%9E%B6%E6%A7%8B.png)
圖源：[DAY 6 實作技術架構- 在IT 邦尋求答案是否少了些什麼|【搜尋引擎製作錄】](https://ithelp.ithome.com.tw/articles/10295830)

## 1.爬取資料：
以＂餐廳＂為關鍵字，選擇Google評論中前100家餐廳。建立名為restaurants的dictionary，並以餐廳名為key，儲存網址、評論tag（Google有將評論中常出現的關鍵字做出整理，數量大約10個上下）、comment、star(總評分)。並存成csv檔。

程式概述：
使用了 selenium 此套件來模擬使用瀏覽器，進而獲得更多頁面資訊。創建名為 restaurants的dictionary，存放的資料內容如下。
```python
restaurants={
    “restaurant_name1”: {
        “href”:””,
        “tag”: {“tag1”,”tag2”...},
        “comment”: {comment1”,comment2”...}
        “star”:” score”
　　}
　　“restaurant_name2”: {
        “href”:””,
        “tag”: {“tag1”,”tag2”...},
        “comment”: {comment1”,comment2”...}
        “star”:” score”
　　}
　　…
}
```
利用CSS_SELECTOR、XPATH等方法定位到tag、comment、score等所需要儲存的資訊，並以正則表達式初步清理評論資料，最後再以 writerow的方式，將餐廳此dictionary寫入 CSV 檔中。

## 2.情緒分析：
- 串接ChatGPT的api，給予餐廳的評論。
- 以Plutchik情緒模型中八大類情緒中的24種情緒詞彙``(狂喜/快樂/恬靜,警戒/期待/興趣,狂怒/憤怒/困惱,反感/厭惡/無聊,喪慟/悲傷/苦惱,驚異/驚訝/分心,恐怖/恐懼/擔心,渴慕/信任/認命)``分析評論中最符合文本的兩種情緒。
- 按照情緒正向程度，量化成0-10的數值。
- 由文本摘要判斷此評論中使用者關於餐廳最滿意和不滿意之處。

輸出格式為: 情緒1,情緒2｜情緒1轉換數值,情緒2轉換數值｜最滿意的地方(10字內，可寫無),最不滿意的地方(10字內，可寫無)｜

如下表中的示例：

|定義問題內容|
|:--|
|![image](https://github.com/yyc0314/Fooood-map/blob/7d41e12feef7484591d56c4875228a2760e38c2a/img/%E5%AE%9A%E7%BE%A9.png)|

|**評論1**|內容|
|:--|:--:|
|評論|![image](https://github.com/yyc0314/Fooood-map/blob/7d41e12feef7484591d56c4875228a2760e38c2a/img/google%E8%A9%95%E8%AB%961.png)|
|回饋|![image](https://github.com/yyc0314/Fooood-map/blob/7d41e12feef7484591d56c4875228a2760e38c2a/img/ChatGpt1.png)|
|**評論2**|**內容**|
|評論|![image](https://github.com/yyc0314/Fooood-map/blob/7d41e12feef7484591d56c4875228a2760e38c2a/img/google%E8%A9%95%E8%AB%962.png)|
|回饋|![image](https://github.com/yyc0314/Fooood-map/blob/7d41e12feef7484591d56c4875228a2760e38c2a/img/ChatGpt2.png)|


而其中在經過幾輪測試後，發現過短的評論通常所包含的資訊較少，依格式轉化的情緒、評分也不夠準確，而過長的評論在傳送時會消耗大量token，所以先篩選了評論長度介於30-80的評論進行更多的分析。

## 3.推薦系統：
　　當使用者輸入關鍵字時，可以基於餐廳的特徵（tag）和評論滿意與不滿意之處計算餐廳與關鍵字的相似度和關聯性，量化成推薦分數，並結合加權過後之情緒分析分數，排序想推薦給使用者的餐廳。

## 4.網站開發：
　　以自己學習過的程式語言進行網站開發，前端使用Html、CSS編寫，後端用PHP，資料庫用MySQL撰寫，並建立相關索引加速搜尋。
  
網站使用流程為：
1. 使用者輸入關鍵字，可以是餐廳名、菜名、料理口味等。
2. 根據關鍵字，系統從資料庫中檢索相關餐廳。
3. 利用推薦系統排序符合關鍵字的餐廳，生成推薦列表。
4. 回饋給使用者餐廳列表的資訊，包含餐廳連結、評分、特徵，也可以了解到多數使用者的情緒，屢次被提及與評論中的優缺點等。

