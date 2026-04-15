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

保持现状(推荐) — 按 CLAUDE.md 的设计:bucket 默认 private → 给 watermarked/ thumbnails/ 两个前缀单独配 public-read(在阿里云控制台 bucket → 权限管理 → bucket policy 加 2 条规则即可,5 分钟搞定),原图依然只走预签名。安全、便宜、性能好。
缩略图: https://inkqiyi.oss-cn-shanghai.aliyuncs.com/thumbnails/artworks/119/261815a350914d7aa1d9da77185f9817.webp
水印图: https://inkqiyi.oss-cn-shanghai.aliyuncs.com/watermarked/artworks/119/261815a350914d7aa1d9da77185f9817.webp


# ========== Gemini AI ==========
GEMINI_API_KEY=AIzaSyCEl5QxQUrheh3e-iCZrVtTPNxuAd2c1xM
GEMINI_API_URL=https://apigateway.iwzh.net/v1beta/models
GEMINI_MODEL=gemini-3.1-flash-image-preview
GEMINI_TIMEOUT=180
GEMINI_IMAGE_SIZE=1K
GEMINI_IMAGE_COST_POINTS=10

# ========== AI 存储 (对齐 gateway 的 /static 挂载) ==========
AI_STORAGE_BACKEND=local
AI_STORAGE_LOCAL_ROOT=./storage/ai-uploads
AI_STORAGE_PUBLIC_BASE_URL=http://localhost:8080/static/ai-uploads

# ========== gateway static base ==========
STORAGE_BASE_DIR=./storage

# ========== Admin DB（system_configs 等管理后台表所在库） ==========
ADMIN_DATABASE_URL=sqlite+aiosqlite:///./data/admin.db

# ========== 主密钥（用于加密 system_configs 中 is_encrypted=1 的字段） ==========
# 一旦设置后请勿再修改,否则历史密文将无法解密。
# 生成方式: python -c "import secrets; print(secrets.token_hex(32))"
MASTER_KEY=a1b2d292b79848f7d455507b666ec41b03e2b6723603359b70a3ff19b50c0d44

# ========== 存储后端选择 ==========
# auto = 优先 OSS,加载失败回退本地; oss = 强制 OSS; local = 强制本地
STORAGE_BACKEND=auto

# ========== 阿里云 OSS（首次引导值，由 seed_oss_config.py 写入 system_configs） ==========
ALIYUN_OSS_ENDPOINT=oss-cn-shanghai.aliyuncs.com
ALIYUN_OSS_BUCKET=inkqiyi
ALIYUN_OSS_ACCESS_KEY_ID=AALTAI5tSGPPReBBE2wGcd9rkRRY
ALIYUN_OSS_ACCESS_KEY_SECRET=UI73rBXpdV6dt4Q1kpeDiuj0kfT1lvJOGH
ALIYUN_OSS_CDN_DOMAIN=


INSERT INTO points_transactions
    (account_id, user_id, tx_type, amount, balance_before, balance_after, related_no, remark, created_at, updated_at)
SELECT
    id,
    user_id,
    'admin_adjust',
    10000,
    balance - 10000,
    balance,
    NULL,
    '管理员手动充值',
    NOW(),
    NOW()
FROM points_accounts
WHERE user_id = 5;


SELECT id, user_id, balance, total_earned, total_spent, updated_at
FROM points_accounts WHERE user_id = 5;
