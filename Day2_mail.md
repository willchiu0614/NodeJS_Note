# Day2 mail認證

## 安裝nodemailer
```npm i nodemailer ```
## 取得密碼
Step 1. 登入 [Google Account](https://myaccount.google.com/)
Step 2. 點選左邊的 Security
![](https://i.imgur.com/kdxOkPR.png)
Step 3. 開啟兩階段認證
Step 4. 取得應用程式密碼
![](https://i.imgur.com/rqmjTjE.png)
Step 5. 應用程式選其他(自訂名稱)
Step 6. 產生應用程式密碼
![](https://i.imgur.com/yiRbTL1.jpg)

## 註冊
前情提要
ViewController/register.js
```
$(document).ready(function () {
    $('#registerAlert').css("display", "none");
    console.log("[viewController/register]register ready");
})

$(function () {
  $('#registerForm').submit(function (event) {
    event.preventDefault(); // Prevent the form from submitting via the browser
    var form = $(this);
    $.ajax({
      type: form.attr('method'),
      url: form.attr('action'),
      data: form.serialize(),
      success: function (res) {
        switch (res.returnData.status) {
          case 0:
            $('#registerAlert').css("display", "none");
            break;
          case 1:
            $('#registerAlert').css("display", "none");
            window.location.href="/home/mailPassCheck/"+res.returnData.email;
            break;
          case 2:
            $('#registerAlert').text("email 已經被註冊過");
            $('#registerAlert').css("display", "block");
            break;
          case 3:
            $('#registerAlert').text("密碼錯誤");
            $('#registerAlert').css("display", "block");
            break;
        }
      }
    }).then(success,fail)
  });
});
var success=(data)=> {
  console.log('[viewController/register]成功送出');}
var fail=(data)=> {
  console.log('[viewController/register]無法送出');}
```
![](https://i.imgur.com/gFoXMpC.jpg)


## 當成功註冊時跳轉頁面
前端register.js判斷註冊狀態成功時會呼叫
```window.location.href="/home/mailPassCheck/"+res.returnData.email;```
跳轉網頁

Controller/pageController

```
router.get('/mailPassCheck/:mail', async function(req, res, next) {
    res.render('mailPassCheck',{
        mail:req.params.mail,
    });
});
```

View/mailCheck.ejs

```
<%- include('header') %>
 <body >
    <div class="container">
        <div class="card-wrap">
            <div class="card border-0 shadow card--register is-show" id="mailPassCheck">
                <div class="card-body">
                    <h2 class="card-title">驗證碼確認</h2>
                    <div>以寄送驗證碼置信箱:<%=mail%></div>
                    <div class="alert alert-danger" role="alert" id="alert" style="display:none ;">
                            This is a danger alert—check it out!</div>
                    <form id="mailPassCheckForm" method="POST" action="/api/mailCheck">
                        <div class="form-group">
                            <div>驗證碼</div>
                            <input class="form-control" type='text' name="mailPassNum" placeholder="" required="required">
                            <input type="hidden" value='<%=mail%>' name="mailAddress" id="mail">
                            <div  class="d-flex justify-content-end ">
                                <div id="time" class="d-inline  p-2  text-red">10</div>                         
                                    <input type="button" value="重送" id="resendBtn" class="d-inline p-2  btn btn-link  ">
                                </div>                     
                            </div>
                            <div>
                                <input type="submit" class="btn btn-lg m-1 p-0" id="mailPassCheckBtn" value="確認">
                            </div>  
                        </form>
                    </div>
                </div>
            </div>
        </div>
        <script src="/js/mailPassCheck.js"></script>
    </body>
    <%- include('footer') %>
```

![](https://i.imgur.com/jyuPcQR.jpg)

## 寄信
在註冊成功時，後端的user.js會呼叫```mail(data.body.mail)```

Controller/nodemailer.js
```
const nodemailer = require('nodemailer');
const usersModel = require('../Model/user');

//建立一個smtp伺服器
const config = {
    service: 'Gmail',
    auth: {
        user: 'willchou0614@gmail.com', 
        pass: 'wpletzwubwlqisju' //郵箱的授權碼，不是註冊時的密碼,等你開啟的stmp服務自然就會知道了
    }
};
// 建立一個SMTP客戶端物件
const transporter = nodemailer.createTransport(config);

// 生成6位隨機數
randomFns=()=> {
    let code = ""
    var arr=['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z']
    for(var i=0;i<6;i++){
      pos=parseInt(Math.random()*(arr.length-1));
      code+=arr[pos];
    }
   
    return code 
}

//傳送郵件
module.exports =async function (email){
    
    let code=randomFns();
    transporter.sendMail(options= {
        from: 'willchou0614@gmail.com',//寄件者
        to:email, //收件者
        subject: '測試信件', //主旨
        //內文
        text: '驗證', //存文字
        html: '<p>--你好！</p>
               <p>您正在註冊帳號</p>
               <p>你的驗證碼是：<strong style="color: #ff4e2a;">'+code+'</strong></p>
               <p>***該驗證碼5分鐘內有效***</p>', //html格式
      }, function(error, info){
        if(error) {
            return console.log(error);
        }
        else
        {
            usersModel.findOneAndUpdate(
                {
                  Mail: email
                },
                {
                    MailCheckVal: code
                },
                {
                  new: true,
                }
              ).then((data) => {
                console.log('data', data)
              })
        }
        console.log('mail sent:', info.response);
    });
};

module.exports.check = async function (mailAddress,val){
    let result=await usersModel.findOne({ Mail:mailAddress })
    let code=result.MailCheckVal;
    if(val==code)
    {
        console.log("驗證成功")
        usersModel.findOneAndUpdate(
            {
              Mail: mailAddress
            },
            {
              State: 1
            },
            {
              new: true,
            }
          ).then((data) => {
            console.log('data', data)
          })
          
        return 1;
    }
    return 0;  
}
```
## 重寄驗證碼/判斷認證
跳轉網頁後會開始倒數
* 當點擊resendBtn時會透過ajax通知後端nodemailer產生新的認證碼並重寄
* 當觸發submit時會透過ajax通知後端nodemailer做判斷
    * 將mail與輸入的驗證碼值傳給後端
    * 後端透過mail查詢驗證碼
    * 將查詢的驗證碼與傳入使用者輸入的值做判斷

ViewController/mailPassCheck.js
```
$(document).ready(function () {
    $('#registerAlert').css("display", "none");
})
var curTime
window.onload = function(){
  Init();
}
function Init(){
  //只要text裏面的數值還未到0，則不停地以每秒減一的速度進行着
  curTime=10;
  if(curTime>0){
    start();
  //一旦清零，就停止
  }else{
    stop();
  }
}
function start(){
  //開啓計時器，每秒text框中的數值自減1
  timer = setInterval(function(){
    curTime--;
    $('#time').text(curTime);
    //當text框中的數值爲0時，停止計時器
    if(parseInt($('#time').text())<=0){
      $('#time').text('逾期');
      clearInterval(timer);
    }
  },1000)
}
//封裝一個清楚延時器的函數
function stop(){
  clearTimeout(timer);
}

//resend
$("#resendBtn").click(function(){
  $.ajax({
    type: 'GET',
    url: '/api/resendMail',
    data:{
      mailAddress:$("#mail").val(),
    },
    success: function (res) {
      stop();
      Init();
    }
  })
});

//mailPassCheck
$(function () {
  $('#mailPassCheckForm').submit(function (event) {
    event.preventDefault(); // Prevent the form from submitting via the browser
    var form = $(this);
    $.ajax({
      type: form.attr('method'),
      url: form.attr('action'),
      data: form.serialize(),
      success: function (res) {
        switch (res.returnData) {
          case 0:
            $('#alert').text("驗證碼錯誤");
            $('#alert').css("display", "block");
            break;
          case 1:
            $('#alert').css("display", "none");
            window.location.href="/home/login";
            break;
          
        }
      }
    }).then(success,fail)
  });
});
var success=(data)=> {
  console.log('[viewController/register]成功送出');}
var fail=(data)=> {
  console.log('[viewController/register]無法送出');}
```