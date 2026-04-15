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


- 连接到 inkframe_points 库
USE inkframe_points;

-- 给 user_id = 5 添加 10000 点数（若已存在则累加；不存在则插入）
INSERT INTO points_accounts (user_id, balance, total_earned, total_spent, created_at, updated_at)
VALUES (5, 10000, 10000, 0, NOW(), NOW())
ON DUPLICATE KEY UPDATE
    balance       = balance + 10000,
    total_earned  = total_earned + 10000,
    updated_at    = NOW();

-- 补一条流水，便于后台对账
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

-- 验证
SELECT id, user_id, balance, total_earned, total_spent, updated_at
FROM points_accounts WHERE user_id = 5;
抓到真正的错误了,根本不是余额问题。

真实错误

StatusCode.UNAVAILABLE
failed to connect to all addresses; last error: UNKNOWN:
ipv4:127.0.0.1:60061: Failed to connect to remote host: Connection refused
ai_service 尝试用 gRPC 连 127.0.0.1:60061 调 SpendPoints,连不上。所以报错完全不是因为你点数不够——余额 10000 已确认到账。错误发生在预扣点数的 gRPC 调用阶段,被 ai_servicer_mixin.py:63-68 的 except Exception 吞掉后套了个误导性的"点数不足或扣减失败"前缀。

根因
ai_service 的默认 points 地址是 main.py:123:


os.environ.get("POINTS_SERVICE_GRPC_ADDR", "127.0.0.1:60061")
生产是 Docker 部署,ai_service 容器里的 127.0.0.1:60061 指向它自己,而不是 points_service 容器。正确做法是环境变量设成 points_service:60061(Docker compose 服务名)。

翻 docker-compose.yml 验证一下:

Grep "POINTS_SERVICE_GRPC_ADDR|ai_service:" (in /Users/bitshare/Desktop/panhx/xiyiart/xiyiart/docker-compose.yml)
16 lines of output
Grep "POINTS_SERVICE_GRPC_ADDR|ai_service:" (in /Users/bitshare/Desktop/panhx/xiyiart/xiyiart/docker-compose.prod.yml)
16 lines of output
Read docker-compose.yml (lines 412-461)
Read docker-compose.prod.yml (lines 271-350)
确认根因。 定位到 docker-compose.yml:419-430 和 docker-compose.prod.yml:278-293:

ai_service 容器的 environment 里没有 POINTS_SERVICE_GRPC_ADDR(和 ARTWORK_SERVICE_GRPC_ADDR),所以 main.py:123 走默认值 127.0.0.1:60061——在容器内就是 ai_service 自己,端口上根本没东西监听,于是 connection refused。

对比 gateway 的配置(docker-compose.yml:184-194),它把 11 个 *_SERVICE_GRPC_ADDR 都显式列出了,但 ai_service 被漏了。这是个配置遗漏的 bug,不是代码 bug。

修复
两份 compose 都改。docker-compose.yml 里 L419-L423:


  ai_service:
    ...
    environment:
      <<: *common-env
      DATABASE_URL: mysql+aiomysql://inkframe:inkframe_dev_2024@mysql:3306/inkframe_ai
      AI_SERVICE_GRPC_PORT: "60060"
      METRICS_PORT: "9100"
      POINTS_SERVICE_GRPC_ADDR: points_service:60061   # 新增
      ARTWORK_SERVICE_GRPC_ADDR: artwork_service:60053 # 新增(生图成功后要调 artwork 入库)
    depends_on:
      mysql: { condition: service_healthy }
      redis: { condition: service_healthy }
      points_service: { condition: service_started }   # 新增(避免启动顺序问题)
      artwork_service: { condition: service_started }  # 新增
docker-compose.prod.yml 里 L286-L293 同理补这两行 env。

应用
生产上:


# 1. 改 docker-compose.prod.yml(把上面两行加进 ai_service.environment)
# 2. 只重启 ai_service 即可,不需要动其它容器
docker compose -f docker-compose.prod.yml up -d --force-recreate ai_service

# 3. 验证 env 注入进去了
docker exec inkframe-ai-service env | grep -E 'POINTS|ARTWORK'
# 应看到:
#   POINTS_SERVICE_GRPC_ADDR=points_service:60061
#   ARTWORK_SERVICE_GRPC_ADDR=artwork_service:60053

# 4. 看启动日志
docker logs inkframe-ai-service 2>&1 | grep -E 'PointsService|ArtworkService'
# 应看到:
#   PointsService gRPC client 已初始化
#   ArtworkService gRPC client 已初始化
改完后用刚才这个 curl 再跑一遍 /api/v1/ai/generate,应该就能成功返回 generation_id + points_remaining=9990 了。

顺带建议(可不改,但值得知道)
ai_servicer_mixin.py:63-68 那段 except Exception as e 把所有异常都套成"点数不足或扣减失败",这次排查被误导了 20 分钟就是因为它。建议区分:

InsufficientBalance 类 gRPC FAILED_PRECONDITION → "点数不足"
其它异常(UNAVAILABLE / DEADLINE_EXCEEDED / ...) → "点数服务不可用,请稍后重试",日志层面 logger.exception(...)
但这是二期优化,当务之急是先把上面那两行 env 补上让你能跑通。
