# Day4 JWT/cookie

[![hackmd-github-sync-badge](https://hackmd.io/kQ-jVjhMQ_qmmBXniUtULA/badge)](https://hackmd.io/kQ-jVjhMQ_qmmBXniUtULA)

## Cookie
Signed Cookie
當伺服器端在產生 cookie 時，都會加上 secret 來作 hash，來保證回來的資料沒有被更動過
```{ dotcom_user: 'will_chiu' }```
搭配上一段秘密字串
```
var r = sha1('this_is_my_secret' + 'will_chiu') 
// d01a3d595af33625be4159de07a20b79a1540e54 
```
註:SHA-1:雜湊函式
最後回傳到用戶端的 cookie 為
```
{ dotcom_user: 'will_chiu’,
 'dotcom_user.sig': 'd01a3d595af33625be4159de07a20b79a1540e54' } 
```
如果用戶端更改了 cookie，因為不知道秘密字串無法產生正確的 hash 值，在校對時會出錯。因此避免掉被竄改 cookie 的可能


---

## Session
Session 的機制就像是你去飲料店點餐後，得到號碼牌(店員只認號碼牌)
店員會跟據這號碼牌，認定你是顧客、是否點過餐、知道你點了什麼東西，然後可以接著給你屬於你的飲料。
一般而言，session 可以存放在：
1. 記憶體内存
1. cookie本身
1. redis 或 memcached 等缓存中（常見）
1. server數據庫中 ex. mongoDB。

以存放server來說，可以記錄client的編號跟相關資訊，例如說：「client1 coffee=1」，接著只在前端cookie裡面存「 client001 」
下次透過編號比對紀錄就知道相關資訊。
由於不把主要資訊存在client，所以也不會有被竄改的風險。
但是如果有人把編號改成其他人的怎麼辦？就可以偽造其他人的身份。
解決:
* **解法1:把 Cookie 裡面的內容給加密**
這種方式就稱之為 Cookie-based session，意思就是你把所有的 Session 狀態都存在 Cookie 裡面
    * 優 : 不須同源
        ```同源 : 所謂的同源政策是用來限制網路資源共享的規則，當網路資源有著相同的 (1 協議 (2 網域 (3 端口 時，這些資源才可以彼此共享。```
    * 缺 : 
    1.    Cookie 的大小是有限制的，超過大小的話瀏覽器就不幫你存，就要考慮第二種
    1.    會遇到回放攻擊
    ```回放攻擊 : 指的是，比如一個用戶，它現在有100 積分，積分存在session 中，session 保存在cookie 中。他先複製下現在的這段cookie，然後去發個帖子，扣掉了20 積分，於是他就只有80 積分了。而他現在可以將之前複製下的那段cookie 再粘貼回去瀏覽器中，於是服務器在一些場景下會認為他又有了100 積分。```
* **解法2:就是透過一個 Session ID 來辨識身份**
Server 只在 前端Cookie 裡面存一個 Session ID，其餘的狀態存在 Server，需要時透過 Session ID 到後端取Session資料
    * 不會遇到回放攻擊
    * 需要同源對超大流量的網站服務來說，因為他們有無數台對外的 Server，無法使用此方法
![](https://i.imgur.com/qSt3Xfa.png)


---

## Token
* 一種訪問時所需要的憑證
* 組成:  唯一Id+時間戳記+簽名(UID +Time stamp+signature)
* 每次攜帶Token都會放到Http的header
* Token不受限於同源問題

流程
1. Client要求登入
1. Server驗證成功會發送Token到Client
1. Client收到請求會儲存在Cookie或localStorage
1. Client每次請求都會帶Token
1. Server每次收到請求都會驗證Token


---

## JWT (JSON Web Token)
JWT是目前最流行的跨網域解決方案，一般被用來傳遞使用者的資訊
* 在雙方之間安全地將訊息作為 JSON 物件傳輸
* 訊息是經過數位簽章(Digital Signature)，因此可以被驗證及信任。
    * 可以使用 密碼(經過 HMAC 演算法) 或用一對 公鑰/私鑰(經過 RSA 或 ECDSA 演算法) 來對 JWT 進行簽章

驗證流程
1. Client登入，Server認證成功之後送一組JWT到Client。
1. Client收到請求會儲存JWT在Cookie或localStorage。
1. 當使用者想要訪問受到保護的route或是資源的時候，需要在header的Authorization加入Bearer模式
1. Server的Router將會檢查Authorization
JWT可夾帶使用者訊息，因此減少了訪問資料庫的動作
使用者的狀態不再儲存到Server，所以是一種無狀態驗證機制
* **自帶狀態**
可以自帶狀態，不用把狀態存在後端增加後端負擔。適合運行在 無狀態 架構的後端
若是把 token 過期時間放在 payload 中，就可以為 token 加入效期特性
* **自驗性**
只要有同樣的 secret ，再做一次算法簽名看結果和 Signature 是否一樣，就可以判斷 JWT 是否有效
* **易變性／完整性(integrality)**
跟 hashing 函數一樣，只要 Header 或 Payload 的內容修改，Signature 的結果就有很大的改變，所以保証資料的完整性

**差異**
* Token : 驗證Client送過來的Token時，還需要使用資料庫查詢資料，再驗證Token是否有效
* JWT : 將Token和payload加密後儲存在Client，Server只需要使用key進行解密，不需要額外再使用資料庫，因為JWT已經包含使用者資訊和加密訊息。


**由```標頭(Header).內容(Payload).簽名(Signature)```組成**
* Header(標頭): 用來指定 hash algorithm(預設為 HMAC SHA256)
* Payload(內容): 可以放一些自己要傳遞的資料
* Signature(簽名): 為簽名檢查碼用，會有一個 serect string 來做一個字串簽署

### Signature
在環境變數裡設定SECRET為簽章(值可自取)

process.env
```
SECRET=TOKENSECRET //設定
```
### 安裝 jsonewbtoken
```npm install jsonwebtoken --save```
### 登入成功時創建token
Controller/pageController.js
```
const jwt = require('jsonwebtoken') // 雜湊加密
...
module.exports.postLoginData = async function (data) {
  //宣告回傳內容(新增token資訊)
  var state={
    'status':null,
    'name':null,
    'token':null,
  }
  
  // 取得使用者註冊資料
  email = data.body.mail
  password = data.body.password

  // 檢查該信箱是否已經註冊
  const user = await usersModel.findOne({  Mail:email });
  if (user) {     //信箱存在
     isPass = await bcrypt.compareSync(password, user.Password);
    if (isPass) { //密碼正確
      state.status = type.LoginEnum.sucess;
      state.name=user.Name;

      // 驗證通過 給客戶端返回token 以備後面驗證登陸
      // 生成token
      const payload={
        user_name:user.Name,
        user_mail:user.Mail,
        user_sex:user.Sexual,
        user_age:user.Age
      }
      const token = jwt.sign({
        payload,
        exp: Math.floor(Date.now() / 1000) + (60 * 15)
      }, process.env.SECRET)
      //exp: 設定簽發的 JWT 多久到期，這裡設 15分鐘
      state.token=token;
      Object.assign({ code: 200 }, { message: '登入成功', token }); 
    }else{ //密碼錯誤
      state.status = type.LoginEnum.passwordError;
    }
  } else {//信箱不存在
    state.status = type.LoginEnum.mailError;
  }
  return state;
}
```
### 將token儲存cookie
```res.cookie("token", postState.token, { maxAge: process.env.COOKIE_EXPIRES_IN, httpOnly: true });```
1. 把 回傳的資料postState.token 存在名為 "token" 的 cookie裡
1. 在環境變數裡設定 COOKIE_EXPIRES_IN 為 cookie 有效期間
1. HttpOnly 為 true，讓cookie token 不能用 javascript 取出


Controller/pageController.js

```
router.post('/login', async (req, res) => {
    postState = await user.postLoginData(req);
    userState=postState;

    //將token存在cookie
    res.cookie("token", postState.token, { maxAge: process.env.COOKIE_EXPIRES_IN, httpOnly: true });
    
    res.send({
        status: postState.status,
      });
})

```
### token權限判斷
假設我們有個api或是頁面需要登入後才能使用、閱覽
我們就可以藉由toke來判斷權限
在此我們假設```http://localhost:8080/home/allUsers```需要登入權限才可閱覽

Controller/pageController.js
```
var tokenFunc= async (req,res,next)=>{// 解析token
    const user = await usersModel.findOne({Name:req.payload.payload.user_name});
    if (user) {
        req.user = user;
        next();
    } else {
        res.send({
            errmsg: 'token已過期'
        })
    }
}

// 可直接使用 controller 的方法拿取資料和進行 render
router.get('/allUsers',authenticate,tokenFunc,async (req,res)=>{
    //token驗證成功後的動作
    res.render('allUsers')
  })
```
創建了一個authenticate模組用來判斷token
我們用 ```jwt.verify(token,signature,callback)``` 方法進行 JWT 驗證
* token: API Token 也就是 JWT 而變數 token 是由 article.controller.js 中傳過來的
* signature: 簽署密碼為字串型態記得要與當時登入時所簽署的密碼一樣否則會出問題
* callback : 回傳結果
    *  err 為錯誤訊息
    *  payload 為 JWT 解密後儲存的資料(前面生成token時所儲存的payload資料)

Controller/authenticate.js
```
const jwt=require('jsonwebtoken');
module.exports =  async (req, res, next) => {
    //利用cookie紀錄token
    const token  = req.cookies.token
    if (typeof token !== 'undefined') {
        
        return new Promise((resolve, reject) => {
            jwt.verify(token, process.env.SECRET, (err, payload) => {
              if (err) {
                reject(err); //驗證失敗回傳錯誤
              } else {
                //在response中建立一個參數
                req.payload=payload;
                req.token = token; 
                next();
                resolve(payload); //驗證成功回傳 returnData
              }
            });
          }).then((result)=>{
            console.log("token 驗證成功,returnData:",result)
          }).catch((err)=>{ return res.status(401).send(err); }) //失敗回傳錯誤訊息
    }
    else
    {
        res.status(403).send(Object.assign({ code: 403 }, { message: '您尚未登入！' })); // Header 查無 Rearer Token
    }
}
```