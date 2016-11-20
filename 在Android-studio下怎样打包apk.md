title: 在Android studio下怎样打包apk
date: 2015-12-02 13:07:03
tags: android
categories: [android,IDE]
---
本文将介绍如何利用Android sudio将我们的项目打包成一个可发布的Android应用。
<!--more-->
<h3>为什么要打包</h3>
&emsp;&emsp;打包其实就是给我们的Android程序生成一个独一无二的签名。如果我们需要将我们的Android应用发布出去，就要保证我们程序的独一无二性.由于开发商可能通过使用相同的包名来混淆替换已经安装的程序，签名可以保证相同名字，但是签名不同的包不被替换。APK如果使用一个key签名，发布时另一个key签名的文件将无法安装或覆盖老的版本，这样可以防止你已安装的应用被恶意的第三方覆盖或替换掉。这样签名其实也是开发者的身份标识。
<h3>打包的步骤</h3>
1.在Android studio菜单中选择Build->Generate Signed APK，弹出一个窗口，如下图：

![](http://7xo6vj.com1.z0.glb.clouddn.com/15-12-2/79936164.jpg)    

2.在弹出的窗口中选择要进行打包的module，我这选择的是ColorBoard，然后点击next

![](http://7xo6vj.com1.z0.glb.clouddn.com/15-12-2/81654179.jpg)  

3.我们要在这里选择一个key store的路径，如果之前建立过的话直接选择就好了，如果没有的话先新建一个，点击create new继续。

![](http://7xo6vj.com1.z0.glb.clouddn.com/15-12-2/86843303.jpg)
  
4.弹出此窗口来新建key store，需要填写的信息蛮多的，下面一个一个解释下：
>Key store path：密钥库文件的地址   
>Password/Confirm：密钥库的密码 
Key：   
Alias：密钥名称  
Password/Confirm：密钥密码   
Validity(years)：密钥有效时间   
First and Last Name：密钥颁发者姓名   
Organizational Unit：密钥颁发组织   
City or Locality：城市   
Country Code(XX)：国家     

ok新建好了以后点击ok，然后就会回到之前的页面，然后选择我们新建的key store就好了，点击next，跳到下一个窗口点击finish。
![](http://7xo6vj.com1.z0.glb.clouddn.com/15-12-2/5875659.jpg)