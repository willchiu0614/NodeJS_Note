# Day1 註冊

## 前端網頁
```
<%- include('header') %>
    <body >
        <div class="container">
            <div class="card-wrap">
                <div class="card border-0 shadow card--register is-show" id="register">
                    <div class="card-body">
                        <h2 class="card-title">註冊</h2>
                        <div class="alert alert-danger" role="alert" id="registerAlert" style="display:none ;">
                            This is a danger alert—check it out!
                        </div>
                        <form id="registerForm" method="POST" action="/home/register">
                            <div class="form-group">
                                <div>使用者名稱</div>
                                <input class="form-control" type='text' name="username" placeholder="" required="required">
                            </div>
                            <div class="form-group">
                                <div>年齡</div>
                                <input class="form-control" type='number' name="age" placeholder="" required="required">
                            </div>
                            <div class="form-group">
                                <div>帳號信箱</div>
                                <input class="form-control" type="email" name="mail" id="mail" placeholder="" required="required">
                            </div>
                            <div class="form-group">
                                <div>密碼</div>
                                <input class="form-control" type="password" name="password" placeholder="" required="required">
                            </div>
                            <div class="form-group">
                                <div>密碼確認</div>
                                <input class="form-control" type="password" name="password2" placeholder="" required="required">
                            </div>
                            <div><input type="submit" class="btn btn-lg m-1 p-0" id="registerBtn" value="註冊"></div>
                        </form>
                    </div>
                </div>
            </div>
        </div>
        <script src="/js/register.js"></script>
    </body>
    <%- include('footer') %>
```
Controller/pageController.js
```
router.get('/register', async function(req, res, next) {
    res.render('register');
});
```
![](https://i.imgur.com/gFoXMpC.jpg)

form裡面有幾個重點
method="POST" 
action="/home/register"
這是使用ajax時需要填入的方法跟路徑
先來看看前端js寫了什麼

ViewController/register.js
```
$(document).ready(function () {
    $('#registerAlert').css("display", "none");
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
### 建立form的送出事件
submit後透過ajax與後端溝通

Controller/pageController.js
```
var user = require('./user');
router.post('/register', async (req, res) => {
    postData = await user.postRegisterData(req);
    res.send({ // ViewController/register.js
        returnData: postData,
      })
})
```

### 判斷資料/資料存入資料庫
#### 設計狀態
Controller/type.js
```
module.exports.RegisterEnum= Object.freeze({"faild":0,"sucess":1,"mailRepeat":2, "passwordNotEqual":3})
```
#### 判斷
Controller/user.js
```
var express = require('express');
const usersModel = require('../Model/user');
const type = require('./type');
const mail=require('./nodemailer')

module.exports.postRegisterData = async function (data) {
    var status
    var returnData={
        -1,
        data.body.username,
        data.body.mail
    }
    // 檢查該信箱是否已經註冊
    await usersModel.findOne({ Mail:data.body.mail }).then(user => {
      // 如果已經註冊：退回原本畫面
      if (user) {
        status = type.RegisterEnum.mailRepeat
      } 
      else {
        //密碼二次判斷
        if (data.body.password != data.body.password2) {
          status = type.RegisterEnum.passwordNotEqual
        }
        else {
          // 如果還沒註冊：寫入資料庫
          /*
          下面create為了異步所以要加上return
          https://stackoverflow.com/questions/46457071/using-mongoose-promises-with-async-await
          */
          return usersModel.create({
            Name:data.body.username,
            Age:data.body.age,
            Sexual:data.body.sexual,
            Mail:data.body.mail,
            Password:data.body.password,
            MailCheckVal:'noCode',
          })
            .then(() => {
              status = type.RegisterEnum.sucess;
              mail(data.body.mail);
              //return 回到pageController.js
            })
            .catch(err => console.log(err))
        }
      }
    })
    returnData.status=status
    return returnData
}
```
### 前端渲染
無論是否註冊成功都要回傳狀態
pageController收到postData後透過res.send方式將returnData傳給前端網頁做出相對應渲染
(ajax的success會收到res.send的內容)
* 若returnData.status是0(RegisterEnum.success)則跳轉信箱驗證
* 若returnData.status是1~3則跳出相對應警告

註解:我們會在判斷成功且資料庫成功建立資料後，呼叫**mail(data.body.mail)** 寄出驗證碼信件