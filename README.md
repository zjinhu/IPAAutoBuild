# # # IPAAutoBuild
# # 1.把ipa_build文件夹拖放到你的工程根目录下边（文件夹与xcworkspace文件同级）
# # 2.工程配置Automatically manage signing是否都行
# # 3.xcode8以后打包需要ExportOptions.plist文件，也放在工程根目录(可以直接用archive导出来的那个ExportOptions.plist)
# # 4.然后双击Debug_ipa即可导出到桌面一个ipa的Debug包，同理Release_ipa是Release包，免除手动打包archive每次都需要去选择debug或者release
# # 5.可以根据自己的需要添加命令行发布到fir或者其他分发平台例子如下
```objc
 if [ "$output_ipa_path" != "" ];then
echo "🎉  🎉  🎉  🎉  🎉  🎉  开始上传fir! 🎉  🎉  🎉  🎉  🎉  🎉 "
fir login 96f234022c34f9a96a0d3c60be33388d
fir me
fir publish ${output_ipa_path} 
fi
```

# # 6.Jenkins打包
# fastlane 打包到处IPA
```
fastlane gym --export_method ad-hoc --output_name XXXX --clean --configuration Debug --export_xcargs -allowProvisioningUpdates
```
# curl命令上传ipa到蒲公英
```
curl -F "file=@$WORKSPACE/XXXX.ipa" -F "uKey=3e9a6893b76fd076d4148c600f1f728d" -F "_api_key=fca58fc9eba4bf1078e669fae79313df" https://www.pgyer.com/apiv1/app/upload
```
# 以下为打包上传完成后调用企业微信开放平台接口获取当前企业微信的token，并且发送消息

# 获取当前企业微信的token，需要安装JQ脚本用于解析json
```
curl -o token.txt -s https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=XXXXXXXXXXXXX\&corpsecret=SSSSSSSSSSSS
token=$(cat token.txt |jq -r '.access_token')
echo "这里是一个文件的内容 $token"
```
# 获取当前APP信息
```
InfoPlistPath="XXXX/Info.plist"
bundle_version=`/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" $InfoPlistPath` 
```
# 获取当前时间
```
current_data="$(date +%Y%m%d_%H%M%S)"
```
# Curl命令 POST消息 toparty为需要接收当前消息的小组ID（例如测试组，产品组，UI组。。。。。等等，ID自行获取企业微信有接口，拿到token后postman即可）
```
curl -H "Content-Type: application/json" -X POST -d '{"touser" : "","toparty" : "54|3|38|45|47|48|51|53|55|56|57","totag" : "","msgtype" : "text","agentid" : 1000003,"text" : {"content" : "XXXXAPP测试包'$bundle_version'版本已于'$current_data'打包成功,下载地址<a href=\"https://www.pgyer.com/iktjb\">点击跳转蒲公英下载</a>。"},"safe":0}' https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=$token
```
