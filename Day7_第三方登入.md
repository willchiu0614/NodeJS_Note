# Day7 第三方登入

[![hackmd-github-sync-badge](https://hackmd.io/hZLORA0vTS-4dzZwiID6Qw/badge)](https://hackmd.io/hZLORA0vTS-4dzZwiID6Qw)

https://hackmd.io/@Mix/rkXLoyItt

https://medium.com/alexchanglife/%E7%94%A2%E5%93%81%E9%96%8B%E7%99%BC-%E5%A6%82%E4%BD%95%E5%9C%A8localhost%E6%B8%AC%E8%A9%A6fb-login-sdk%E5%8A%9F%E8%83%BD-mac-5098973b9901

https://www.youtube.com/watch?v=y7VQzIHCAMc&ab_channel=ProgrammingExperience

https://developers.facebook.com/docs/facebook-login/web?locale=zh_TW

https://icguanyu.github.io/other/facebook/

## ngrok
https://dashboard.ngrok.com/get-started/setup
1. 打開ngrok.exe
2. 輸入
```ngrok config add-authtoken 你的token```
3. 輸入
```ngrok http 本地端要連接的port```
下圖為 ngrok http 8080
https://fa2a-2001-b400-e25e-7e44-3119-5245-2560-e19f.ngrok.io 為對應的https
![](https://i.imgur.com/cIeLzRw.png)

# facebook
https://developers.facebook.com/docs/facebook-login/web

1. 進入[facebook developers](https://developers.facebook.com/)
2. 右上角「我的應用程式」中選擇「建立應用程式」
3. 新增應用程式名稱(e.g 臉書登入測試)+應用程式聯絡電子郵件地址
4. 進入應用程式的管理頁面![](https://i.imgur.com/O5w4YdP.png)
5. 應用程式的管理頁面，選擇「Facebook 登入」右下角的「設定」
6. 選擇網站![](https://i.imgur.com/4Ul3eNl.jpg)
7. 開啟ngrok
8. 輸入 ngrok http 8080
9. ![](https://i.imgur.com/NzI4LUk.jpg)
10. 填入上述網址![](https://i.imgur.com/G8ZnwNi.png)
11. 複製SDK代碼![](https://i.imgur.com/AJkczvp.png)
12. 貼到需要使用第三方登入的HTML ![](https://i.imgur.com/6T2b38D.jpg)
    * '{your-app-id}'就是開發人員頁面上方的 「應用程式編號」
13. [登入按鈕設計](https://developers.facebook.com/docs/facebook-login/web/login-button)
14. 選擇樣式![](https://i.imgur.com/HQ8riwX.png)
15. 取得程式碼![](https://i.imgur.com/msKMQKr.png)
16. 將程式碼貼入HTML內![](https://i.imgur.com/yFa4qqn.jpg)

## 問題

1. 點選臉書登入後![](https://i.imgur.com/pFXULfL.png)
1. 跳出未開啟JSSDK![](https://i.imgur.com/24YlSSi.png)
1. 應用程式模式改為上線/跳出提醒需要設定基本資料![](https://i.imgur.com/vJDAYdq.png)
1. 填入網站網址![](https://i.imgur.com/OPXnOZv.jpg)
1. 將應用程式模式切為上線
1. 應用程式審查/權限與功能/email/取得進階存取權限![](https://i.imgur.com/DIBQiq5.jpg)
1. 應用程式審查/權限與功能/public_profile/取得進階存取權限![](https://i.imgur.com/Hkn86Xy.jpg)
1. facebook登入/設定![](https://i.imgur.com/eMY79bT.jpg)
1. ![](https://i.imgur.com/SsXPxCw.jpg)




---
![](https://i.imgur.com/pa4hRrV.png)
原因:被當成廣告攔截了


## 問題2
說明:登入過程一直讀不到cookie
原因:因為ngrok把url改了但是pageApi裡面忘記改
![](https://i.imgur.com/Q8HAnVh.jpg)

## 
* Facebook SDK for Javascript 會自動在瀏覽器中處理存取權杖儲存和登入狀態追蹤，因此您無須做任何動作來將存取權杖儲存在瀏覽器中
* FB.api() 會自動在呼叫中加入存取權杖

https://blog.camel2243.com/2016/07/29/facebook-fb-login-api-javascript-sdk%EF%BC%8C%E5%AE%A2%E8%A3%BD%E5%8C%96%E6%8C%89%E9%88%95%E5%8F%8A%E4%B8%B2%E6%8E%A5fb%E7%99%BB%E5%85%A5%E5%80%8B%E4%BA%BA%E8%B3%87%E6%96%99/