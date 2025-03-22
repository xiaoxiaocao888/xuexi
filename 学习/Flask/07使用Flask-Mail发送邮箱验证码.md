##### 第一安装Flask-Mail
```
pip install flask-mail
```

##### 第二配置邮箱参数
想要发送邮箱，必须要有一个邮箱服务器。对于中小型公司可以向第三方提供商购买邮箱服务，比如网易企业邮箱、腾讯企业邮箱、阿里企业邮箱等等。
使用Flask-mail发送邮箱，需要使用SMTP协议。开启qq邮箱的
点击设置-账号-往下面滑动找到SMTP开启服务，根据提示发送短信，复制好授权码
![[Pasted image 20250211134521.png]]

代码
在exts.py文件里面引入邮箱，创建对象
```
from flask_mail import Mail
mail=Mail()
```
在app.py文件里面对对象进行初始化
```
from exts import db,mail
mail.init_app(app)
```
在auth.py文件里面引入邮箱和信息，写测试邮箱函数
```
from exts import mail
from flask_mail import Message
@bp.route("/mail/test")
def mail_test():
    message=Message(subject="邮箱测试",recipients=["1455997947@qq.com"],body="这是一条测试邮件")
    mail.send(message)
    return "邮箱发送成功"
```
在config.py文件里面进行邮箱配置
```
MAIL_SERVER="smtp.qq.com"
MAIL_USE_SSL=True
MAIL_PORT=465
MAIL_USERNAME="2769260170@qq.com"
MAIL_PASSWORD="igcjdzstkkekdejh"#开启SMTp服务时生成的授权码
MAIL_DEFAULT_SENDER="2769260170@qq.com"
```
###### 第三发送邮件
运行app.py文件，打开http://127.0.0.1:5000/auth/mail/test
![[Pasted image 20250211144405.png]]
![[Pasted image 20250211143715.png]]

##### 发送验证码
```
import random
import string
from flask import request
@bp.route("/captcha/email")
def get_email_captcha():
    #传参的方式/captcha/email/<email>或/captcha/email?email=xx@qq.com
    email=request.args.get("email")
    #验证码：4/6:随机数组、字母、数字和字母的组合
    source=string.digits*4
    captcha=random.sample(source,4)
    captcha="".join(captcha)
    message=Message(subject="知了传课注册验证码",recipients=[email],body=f"您的验证码是：{captcha}")
    mail.send(message)
    #怎么保存验证码用来验证呢，memcached/redis,我们这里用数据库
    email_captcha=EmailCaptchaModel(email=email,captcha=captcha)
    db.session.add(email_captcha)
    db.session.commit()
    #返回的数据要满足RESTful API格式{"code":200/400/500,message:"","data":{}}
    #print(captcha)
    #return seccess
    return jsonify({"code":200,"message":"","data":None})
```
![[Pasted image 20250211151045.png]]
![[Pasted image 20250211151106.png]]
在models.py文件里面创建email表来保存验证码数据
```
class EmailCaptchaModel(db.Model):
    __tablename__="email"
    id=db.Column(db.Integer,primary_key=True,autoincrement=True)
    email=db.Column(db.String(100),nullable=False)
    captcha=db.Column(db.String(100),nullable=False)
```
![[Pasted image 20250211202003.png]]
也可以看到数据插入到email表了
![[Pasted image 20250211202028.png]]

使用jquery来获取到前端输入的邮箱及发送验证码，在static中新建js文件夹，在js文件夹下面新建register.js文件
```
function bindEmailCaptchaClick(){
    $("#captcha-btn").click(function(event){ //#captcha-btn就是去找id=captcha-btn的那个按钮
        //$this:代表的是当前按钮的jquery对象
        var $this=$(this);
        //阻止默认的事件，大概就是不要在点击发送验证码的时候发送所有的注册信息
        event.preventDefault();
        var email=$("input[name='email']").val();
        //alert(email)
        $.ajax({

            url:"/auth/captcha/email?email="+email,

            method:"GET",

            success:function(result){

                var code=result['code'];

                if(code==200){

                    var countdown=5;

                    //开始倒计时之前，就取消按钮的点击事件

                    $this.off("click");

                    var timer=setInterval(function(){

                        $this.text(countdown);

                        countdown-=1;

                        //倒计时结束的时候执行

                        if(countdown<=0){

                            //清掉定时器

                            clearInterval(timer);

                            //将按钮的文字重新修改回来

                            $this.text("获取验证码");

                            //重新绑定点击事件

                            bindEmailCaptchaClick();

                        }

                    },1000);

                    //alert("邮箱验证码发送成功！");

                }else{

                    alert(result['message']);

                }

            },

            fail:function(error){

                console.log(error);

            }

        })

       })

}

//整个网页都加载完毕后再执行的

$(function(){

  bindEmailCaptchaClick();

})
```

在register.html里面补全head
```
{% block head %}

<script src="{{url_for('static',filename='jquery/jquery.3.6.min.js')}}"></script>

<script src="{{url_for('static',filename='js/register.js')}}"></script>

{% endblock %}
```
这样就可以实现输入邮箱后，点击获取验证码就会发送验证码到邮箱，还会显示有倒计时
![[Pasted image 20250211214952.png]]