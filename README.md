设备管理这块实际应用中有些新的需求需要调整，需要帮我根据需求设计好这些功能
1、device/display-queue/list 这个接口已购买的数字藏品队列和免费队列合并起来，不要分成两组添加一个字段标识是藏品还是免费的
2、增加一个回调接口，目的是设备轮播图片的时候，回调以方便服务端知道对应的设备当前显示的画面。
3、设备激活就是新增的时候返回的数据添加一个key，这个key的作用是设备用来请求接口使用，现有接口需要token的，但是设备获取的相关接口认证使用token或者key都要可以才行
4、图片投屏因为部分图片和屏幕像素不对，人工编辑图片，裁剪、留白、或者填充或者缩放等等，这个要记录，下发的时候是编辑后的图片以适配屏幕显示


"AccessKeyId": "ALTAI5tSGPPReBBE2wGcd9rkR",
"AccessKeySecret": "A73rBXpdV6dt4Q1kpeDiuj0kfT1lvJO",
"Endpoint": "oss-cn-shanghai.aliyuncs.com",
"BucketName": "inkqiyi",


我的建议(3 条路任选)
1、保持现状(推荐) — 按 CLAUDE.md 的设计:bucket 默认 private → 给 watermarked/ thumbnails/ 两个前缀单独配 public-read(在阿里云控制台 bucket → 权限管理 → bucket policy 加 2 条规则即可,5 分钟搞定),原图依然只走预签名。安全、便宜、性能好。

2、折中 — bucket 全 private,但水印/缩略图签 7 天长有效期 + 用 OSS 自身的 URL 鉴权(支持 CDN),DB 还是存 object_key,gateway 列表接口签好再返回。比方案 3 改动小一半。

3、彻底全签名 — 你坚持要的方案。我会给你一份具体的改造清单(预计 10 个文件,12 小时),你确认后再动手。   之前这三个我选2 
