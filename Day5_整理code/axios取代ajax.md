# Day5 整理code/axios取代ajax
## 將register.js與login.js合併
未修改register.js
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
    
var success=(data)=> {
  console.log('[viewController/register]成功送出');}
var fail=(data)=> {
  console.log('[viewController/register]無法送出');}
```
未修改login.js
```
$(document).ready(function () {
  $('#loginAlert').css("display", "none");
  $('#loginEmail').text("11");
  $('#loginPW').text("");
})

$(function () {
  $('#loginForm').submit(function (event) {
    event.preventDefault(); // Prevent the form from submitting via the browser
    var form = $(this);
    $.ajax({
      type: form.attr('method'),
      url: form.attr('action'),
      data: form.serialize(),
      success: function (res) {
        switch (res.status) {
          case 0:
            $('#loginAlert').css("display", "none");
            break;
          case 1:
            $('#loginAlert').css("display", "none");
            window.location.href="/home/allUsers2";
            break;
          case 2:
            $('#loginAlert').text("無此email");
            $('#loginAlert').css("display", "block");
            break;
          case 3:
            $('#loginAlert').text("密碼錯誤");
            $('#loginAlert').css("display", "block");
            break;
        }
      }
    }).done(function (data) {
      console.log('[viewController/login]成功送出');
    }).fail(function (data) {
      console.log('[viewController/login]無法送出');
    })
  });
});
  
```
## step1 將id registerAlert跟loginAlert都改成alert
因為只會載入一個頁面所以無論哪個初始都是要隱藏alert內容
創建user.js
```
$(document).ready(function () {
    $('#alert').css("display", "none");
  })

//register
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
              $('#alert').css("display", "none");
              break;
            case 1:
              $('#alert').css("display", "none");
              window.location.href="/home/mailPassCheck/"+res.returnData.email;
              break;
            case 2:
              $('#alert').text("email 已經被註冊過");
              $('#alert').css("display", "block");
              break;
            case 3:
              $('#alert').text("密碼錯誤");
              $('#alert').css("display", "block");
              break;
          }
        }
      }).then(success,fail);
    });
  });
  
//login
$(function () {
    $('#loginForm').submit(function (event) {
      event.preventDefault(); // Prevent the form from submitting via the browser
      var form = $(this);
      $.ajax({
        type: form.attr('method'),
        url: form.attr('action'),
        data: form.serialize(),
        success: function (res) {
          switch (res.status) {
            case 0:
              $('#loginAlert').css("display", "none");
              break;
            case 1:
              $('#loginAlert').css("display", "none");
              window.location.href="/home/allUsers2";
              break;
            case 2:
              $('#loginAlert').text("無此email");
              $('#loginAlert').css("display", "block");
              break;
            case 3:
              $('#loginAlert').text("密碼錯誤");
              $('#loginAlert').css("display", "block");
              break;
          }
        }
      }).then(success,fail);
    });
  });
    
  var success=(data)=> {
    console.log('成功送出');}
  var fail=(data)=> {
    console.log('無法送出');}
```
## 耍白癡
這邊把register相關命名改成signIn
## axios instance
ViewController/pageApi.js
```
//const axios = require('axios');

// User相關的 api
const userRequest = axios.create({
  baseURL: 'http://localhost:8080/home/'
});
// 文章相關的 api
const articleRequest = axios.create({
  baseURL: 'http://localhost:8080/article/'
});
// 搜尋相關的 api
const searchRequest = axios.create({
  baseURL: 'http://localhost:8080/api/search/'
});

export const axiosMethod= function(instance,method, url, data = null, config) {
method = method.toLowerCase();
switch (method) {
    case "post":
    return instance.post(url, data, config);
    case "get":
    return instance.get(url, { params: data });
    case "delete":
    return instance.delete(url, { params: data });
    case "put":
    return instance.put(url, data);
    case "patch":
    return instance.patch(url, data);
    default:
    console.log(`未知的 method: ${method}`);
    return false;
}
}

export const userSignUp = (signUpData) => {
  return axiosMethod(userRequest,"post", "/signIn", signUpData)
}

export const userLogIn = (logInData) => {
  return axiosMethod(userRequest,"post", "/logIn", logInData)
}

export const userLogOut = () => {
  return axiosMethod(userRequest,"get", "/logOut")
}

export const userDelete = (userNo) => {
  return axiosMethod(userRequest,"delete", "/delete", userNo)
}
```
user.js前面加上```import { userSignUp,userLogIn } from "/js/pageApi.js"; ```
register.ejs將
```<form id="registerForm" method="POST" action="/home/register">```
修改為
```<form id="registerForm" >```

user.js
```
//SignIn
$(function () {
    $('#registerForm').submit(function (event) {
      event.preventDefault(); // Prevent the form from submitting via the browser
      var form = $(this);
      userSignUp(form.serialize())
      .then((res)=>{
        var returnData=res.data.returnData
          switch (returnData.status) {
            case 0:
              $('#alert').css("display", "none");
              break;
            case 1:
              $('#alert').css("display", "none");
              window.location.href="/home/mailPassCheck/"+returnData.email;
              break;
            case 2:
              $('#alert').text("email 已經被註冊過");
              $('#alert').css("display", "block");
              break;
            case 3:
              $('#alert').text("密碼錯誤");
              $('#alert').css("display", "block");
              break;
          }
      })
      .then(success,fail);
    });
  });
  
//login
$(function () {
    $('#loginForm').submit(function (event) {
      event.preventDefault(); // Prevent the form from submitting via the browser
      var form = $(this);
      userLogIn(form.serialize()).then(res=>{
        switch (res.data.status) {
          case 0:
            $('#loginAlert').css("display", "none");
            break;
          case 1:
            $('#loginAlert').css("display", "none");
            window.location.href="/home/allUsers2";
            break;
          case 2:
            $('#loginAlert').text("無此email");
            $('#loginAlert').css("display", "block");
            break;
          case 3:
            $('#loginAlert').text("密碼錯誤");
            $('#loginAlert').css("display", "block");
            break;
        }
      }).then(success,fail);
    });
  });
```