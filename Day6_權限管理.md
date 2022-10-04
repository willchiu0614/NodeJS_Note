# Day6 權限管理
## 前端
在表單裡新增權限下拉選項
![](https://i.imgur.com/Q32KdfV.jpg)

register.ejs
```
<div class="form-group">
    <div>權限</div>
    <select id="AuthoritySelect">                                             
        <option value="0">Visiter</option>
        <option value="1">User</option>
        <option value="2">Admin</option>
    </select> 
</div>
```
user.js
```
var  authority=0;
$("#AuthoritySelect").selectmenu();
$('#AuthoritySelect').on('selectmenuchange', function() {
 authority=$(this).val();
});
```
新增setAlert函式處理不同狀況跟警告
當登入成功網頁導向/home/authority
```
$(document).ready(function () {
  console.log("載入頁面完成")
  console.log("alertStatus:",$('#alertStatus').text())
  if($('#alertStatus').text()==6)
  {
    setAlert(7);
  }
  else if($('#alertStatus').text()==7)
  {
    setAlert(8);
  }
  else
  {
    setAlert(0);
  }
})

//register略過...

//login
$(function () {
    $('#loginForm').submit(function (event) {
      event.preventDefault(); // Prevent the form from submitting via the browser
      var form = $(this);
      userLogIn(form.serialize()).then(res=>{
        setAlert(res.data.status,"/home/authority")
      }).then(success,fail);
    });
  });
    
  var success=(data)=> {
    console.log('成功送出');}
  var fail=(data)=> {
    console.log('無法送出',data);}

var setAlert=(status,url=null)=>{
  switch (status) {    
    case 0:
      $('#alert').css("display", "none");
      break;
    case 1:
      $('#alert').css("display", "none");
      window.location.href=url;
      break;
    case 2:
      $('#alert').text("email 已經被註冊過");
      $('#alert').css("display", "block");
      break;
    case 3:
      $('#alert').text("密碼錯誤");
      $('#alert').css("display", "block");
      break;
    case 4:
      $('#alert').css("display", "none");
      window.location.href=url;
      break;
    case 5:
      $('#alert').text("無此email");
      $('#alert').css("display", "block");
      break;
    case 6:
      $('#alert').text("密碼錯誤");
      $('#alert').css("display", "block");
      break;
    case 7:
      $('#alert').text("逾時");
      $('#alert').css("display", "block");
      break;
    case 8:
      $('#alert').text("發生問題，請重新登入");
      $('#alert').css("display", "block");
      break;
  }
}
```
## 後端
在user.js更新payload新增Authority資料
**user.js**
```
const payload={
        user_name:user.Name,
        user_mail:user.Mail,
        user_sex:user.Sexual,
        user_age:user.Age,
        user_auth:user.Authority
      }
```
在pageController.js新增classifyUserFunc做身分權限判斷
判斷身分後使用重新定向res.redirect取代res.render
因為redirect才會更改網址的url
**pageController.js**
```
var classifyUserFunc= async (req,res,next)=>{// 解析token
    switch(req.payload.payload.user_auth)
    {
        case 0:
            break;
        case 1: 
            const singleUser = await usersModel.findOne({Name:req.payload.payload.user_name});
            if (singleUser) {
                req.user = singleUser;
                //next();
            } else {
                res.send({
                    errmsg: '找不到使用者'
                })
            }
            res.redirect('/home/allUsers/'+singleUser.Name)
            /*res.render('singleUser', { 
                name:singleUser.Name,
              })*/
            break;
        case 2:
            allUserData=await user.getAll()
            res.redirect('/home/allUsers/');
            /*res.render('allUsers', {
                name:req.payload.payload.user_name+'('+req.payload.payload.user_auth+':admin)',
                eachUsers: allUserData
                })*/
            break;
    }
    
    next()
}

router.get('/authority',authenticate,classifyUserFunc)
```

