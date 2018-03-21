---
layout: post
title:  "ES调用身份证阅读器实践"
categories: 日志
tags:  excel服务器 黑科技 VBA 村长 身份证阅读器 身份证扫描识别
---

* content
{:toc}

* 好久没有更新博客了，最近收到一台`@宁波-老方`送来的神盾ICR-100U身份证阅读器，于是花了半天捣鼓了一下ES玩转身份证扫描识别。

* 本次实践参考了宝典中的[ES调用身份证阅读器方案](https://esbook.erp8.net/c4/04.15.html)，感谢`@云`童鞋提供的宝贵教程。

## 设备开箱
一机器，一USB线，一驱动盘而已。

<img src="/img/idcardscan1.jpg">

<img src="/img/idcardscan2.jpg">

## 安装驱动
我的电脑是win10，安装完32位驱动后还要手工更新驱动到64位，具体方法驱动盘里有pdf资料，这里就不写了。
<img src="/img/idcardscan3.jpg">

## 二次开发
* 参考宝典中的教程，先把sdk的demo中的dll等文件都拷贝到EXCEL安装目录

<img src="/img/idcardscan5.jpg">

* 然后在人事档案模板中插入一个模块，加入demo中的代码进行修改，修改完后是这样

<img src="/img/idcardscan6.jpg">

```vb
Public Declare Function InitComm Lib "termb.dll" (ByVal port As Integer) As Integer
Public Declare Function InitCommExt Lib "termb.dll" () As Integer
Public Declare Function Authenticate Lib "termb.dll" () As Integer
Public Declare Function AuthenticateExt Lib "termb.dll" () As Integer
Public Declare Function Read_Content_Path Lib "termb.dll" (ByVal fileName As String, ByVal active As Integer) As Integer
Public Declare Function Read_Content Lib "termb.dll" (ByVal active As Integer) As Integer
Public Declare Function CloseComm Lib "termb.dll" () As Integer
Public Declare Function GetSAMID Lib "termb.dll" () As String
'状态1
Const ReadState = "读卡状态"
Const DebugState = "调试状态"
'状态2
Const OpenPortError = "打开串口失败!"
Const TimeOutError = "通讯超时!"
Const RecError = "操作失败!"
Const XpError = "相片解码错误!"
Const FileExtError = "wlt文件后缀错误!"
Const FileOpenError = "wlt文件打开错误!"
Const FileFormatError = "wlt文件格式错误!"
Const JmError = "软件未授权!"
Const CardError = "卡认证错误!"
Const UnknowError = "未知错误!"
'状态3
Const Swipe = "请放卡..."
Const ReadOK = "读卡成功!请放下一张卡..."
Const ReadError = "读卡失败!请重新放卡..."
Const NewAddError = "读最新住址失败!"
Const IINSNDNError = "读芯片号失败!"
Const Reading = "正在读卡..."
'路径
Const strPathName = "C:"
'变量
Dim bcc, TimeOutFlag As Byte
Dim OutByte() As Byte
Dim RecCount, i, j As Long
Dim PortNum As Integer
Dim ComPort, ReadMode, tmp As String
Dim nametmp, sextmp, nationtmp, borntmp, addresstmp, IDNtmp, regtmp, datetmp As String
Dim RecTmp(), RecByte() As String
'读卡按钮代码
 Sub ReadCard()
    ans = InitCommExt        '开串口
    If ans = 0 Then
        PortNum = 1001
        ans = InitComm(PortNum)         '开USB口
        If ans <> 1 Then
            ret = MsgBox("打开端口失败！", , "错误")
            End
        End If
    End If    
    If ans >= 1001 Then Application.StatusBar = "连接USB口，请放卡..."      
    Dim strSAMID As String '* 37
    strSAMID = GetSAMID()    
    Dim s
    s = Split(strSAMID, "-", -1, 1)
    If UBound(s) > 3 Then Application.Caption = "(" + "授权号: " + s(2) + "-" + s(3) + ") "
    '卡认证
    ans = Authenticate()    
    '卡认证成功
    If ans = 1 Then
        Application.StatusBar = Reading        
        ans = Read_Content_Path(strPathName, 1)
        Select Case ans
           Case 1                      '读卡成功
              Application.StatusBar = ReadOK
              Call Display(strPathName) 'App.Path)
           Case -1                     '相片解码错误
              Call Display(App.Path)
              Application.StatusBar = XpError
           Case -2                     'wlt文件后缀错误
              Application.StatusBar = FileExtError
           Case -3                     'wlt文件打开错误
              Application.StatusBar = FileOpenError
           Case -4                     'wlt文件格式错误
              Application.StatusBar = FileFormatError
           Case -5                     '软件未授权
              Application.StatusBar = JmError
           Case Else                   '读卡失败
              Application.StatusBar = ReadError
        End Select
    End If    
    CloseComm
End Sub
'显示信息
Private Sub Display(ByRef strFilePath As String)
    Dim tmp1 As Byte
    Dim tmp2 As Byte
    Dim rddata As String    
    Open strFilePath & "\wz.txt" For Binary As #1
        Do While Not EOF(1)   ' 检查文件尾。
            Get #1, , tmp1
            Get #1, , tmp2    
            rddata = rddata + ChrW(tmp2 * CLng(256) + tmp1)
        Loop
    Close #1    
    '姓名
    nametmp = Mid(rddata, 1, 15)    
    '性别
    sextmp = Mid(rddata, 16, 1)    
    '民族
    nationtmp = Mid(rddata, 17, 2)    
    '出生日期
    borntmp = Mid(rddata, 19, 8)    
    '住址
    addresstmp = Mid(rddata, 27, 35)    
    '公民身份号码
    IDNtmp = Mid(rddata, 62, 18)    
    '签发机关
    regtmp = Mid(rddata, 80, 15)    
    '有效期限
    ValidDatetmp = Mid(rddata, 95, 16)    
    '【姓名单元格】 请改成自己的range
    Range("C4").Value = nametmp    
    '【性别单元格】 请改成自己的range
    Select Case sextmp
        Case "0"
            Range("E4").Value = "未知"
        Case "1"
            Range("E4").Value = "男"
        Case "2"
            Range("E4").Value = "女"
        Case Else
            Range("E4").Value = "未说明"
    End Select
    '【民族单元格】 请改成自己的range
    Dim nationtmp1 As String
    ans = GetNation(nationtmp, nationtmp1)
    Range("E5").Value = nationtmp1    
    '【地址单元格】 请改成自己的range
    Range("E10").Value = addresstmp
    '【身份证号单元格】 请改成自己的range
    Range("G8").Value = IDNtmp    
    '【照片单元格】 请改成自己的range
    Range("H4").Select
    Application.COMAddIns("ESClient10.Connect").Object.AddPicture strFilePath & "\zp.bmp", 1, 4, 8 ' 插入图片
End Sub
'民族代码查表
Public Function GetNation(ByVal strNationcode As String, ByRef strNation As String)
    Dim strNationArray As Variant    
    strNationArray = Array("汉", "蒙古", "回", "藏", "维吾尔", "苗", "彝", "壮", "布依", "朝鲜", _
                        "满", "侗", "瑶", "白", "土家", "哈尼", "哈萨克", "傣", "黎", "傈僳", _
                        "佤", "畲", "高山", "拉祜", "水", "东乡", "纳西", "景颇", "柯尔克孜", "土", _
                        "达斡尔", "仫佬", "羌", "布朗", "撒拉", "毛南", "仡佬", "锡伯", "阿昌", "普米", _
                        "塔吉克", "怒", "乌孜别克", "俄罗斯", "鄂温克", "德昂", "保安", "裕固", "京", "塔塔尔", _
                        "独龙", "鄂伦春", "赫哲", "门巴", "珞巴", "基诺")    
    If Trim(strNationcode) <> "" Then
        If ((CByte(Trim(strNationcode)) - 1) >= 0) And ((CByte(Trim(strNationcode)) - 1) <= 55) Then
            strNation = strNationArray(CByte(Trim(strNationcode)) - 1)
        Else
            strNation = "其他"
        End If
    End If    
End Function
```

最终效果，点击【读卡】就可以了，棒棒哒，偶尔读不了，是设备问题，拿起再放下就能读了。

<img src="/img/idcardscan4.jpg">

## 拓展
有些童鞋对【插图】按钮的代码感兴趣，这里也一起提供了，嗯，不要谢谢村长，请直接发红包，○( ＾皿＾)っHiahiahia…逃~

```vb
Sub IPIC()
'  function AddPicture(path:BSTR; sh:I2; r:I4; c:I4);  ES vba 接口
    Dim fn                         '存放打开的文件
    '弹出文件打开选框
    fn = Application.GetOpenFilename("图片文件(*.JPG;*.PNG;*.BMP),*.JPG;*.PNG;*.BMP", , "打开（可多选）")
    If fn = "" Then Exit Sub                                     '用户未选择文件
    Cells(4, 8).Select '图片单元格，4，8改成自己的
    Application.COMAddIns("ESClient10.Connect").Object.AddPicture fn, 1, 4, 8 ' 插入图片，4，8改成自己的
End Sub
```

<hr>
by @一零村长

2017双十二
