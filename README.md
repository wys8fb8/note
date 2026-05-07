# 1. 看现状
python scripts/detect_schema_state.py --db-prefix inkframe_dev_

# 2. 假设诊断说 V001-V088 已存在 → 标记 applied（不跑 SQL）
python migrate_manual_mysql.py --db-prefix inkframe_dev_ \
    --mark-applied V001-V088 --dry-run     # 先 dry-run 确认
python migrate_manual_mysql.py --db-prefix inkframe_dev_ \
    --mark-applied V001-V088               # 实际写

# 3. 之后增量自动安全
python migrate_manual_mysql.py --db-prefix inkframe_dev_


[1/2] 扫描 sqlmigrate_mysql/ 提取 schema 标志…
      99 个迁移文件，共 155 个 schema 标志
[2/2] 诊断 12 个库（前缀=inkframe_dev_）…

======================================================================
  inkframe_dev_user  (logical=user)
======================================================================
  [✓] _schema_migrations 表已存在，11 个版本被记录。
      Applied: V001, V002, V003, V004, V012, V013, V014, V015, V016, V060 ... (共 11 个)

  Schema 推断结果（基于 sqlmigrate_mysql/ 标志检测）：
    [✓] 已应用 (9): V001, V002, V003, V004, V012, V013, V015, V016, V060
    [✗] 未应用/部分应用 (1):
        - V098: users.phone_verified_at
    [?] 无法判断 (1): V014 (纯数据迁移，无 schema 标志)
    [⚠️ ] V072 BIGINT 修复未应用（id 列仍为 INT）→ 需手工跑：
          mysql -h $MYSQL_HOST -u $MYSQL_USER -p < sqlmigrate_mysql/V072__fix_int_to_bigint.sql

======================================================================
  inkframe_dev_artist  (logical=artist)
======================================================================
  [✓] _schema_migrations 表已存在，8 个版本被记录。
      Applied: V005, V006, V017, V018, V019, V020, V061, V067

  Schema 推断结果（基于 sqlmigrate_mysql/ 标志检测）：
    [✓] 已应用 (7): V005, V006, V017, V018, V019, V020, V067
    [✗] 未应用/部分应用 (1):
        - V093: artist_kyc_records.id_card_number
    [⚠️ ] V072 BIGINT 修复未应用（id 列仍为 INT）→ 需手工跑：
          mysql -h $MYSQL_HOST -u $MYSQL_USER -p < sqlmigrate_mysql/V072__fix_int_to_bigint.sql

======================================================================
  inkframe_dev_artwork  (logical=artwork)
======================================================================
  [✓] _schema_migrations 表已存在，8 个版本被记录。
      Applied: V007, V008, V009, V021, V022, V053, V057, V061

  Schema 推断结果（基于 sqlmigrate_mysql/ 标志检测）：
    [✓] 已应用 (9): V007, V008, V009, V021, V022, V053, V057, V082, V083
    [✗] 未应用/部分应用 (3):
        - V091: 表 admin_gallery_artworks; 表 tmp_old_artwork_to_admin_gallery
        - V092: artworks.review_status; artworks.listing_status; artwork_review_records.change_type
        - V095: 表 user_library_new
    [?] 无法判断 (7): V079 (纯数据迁移，无 schema 标志), V087 (纯数据迁移，无 schema 标志), V089 (纯数据迁移，无 schema 标志), V090 (纯数据迁移，无 schema 标志), V094 (纯数据迁移，无 schema 标志) ...
    [⚠️ ] V072 BIGINT 修复未应用（id 列仍为 INT）→ 需手工跑：
          mysql -h $MYSQL_HOST -u $MYSQL_USER -p < sqlmigrate_mysql/V072__fix_int_to_bigint.sql

======================================================================
  inkframe_dev_copyright  (logical=copyright)
======================================================================
  [✓] _schema_migrations 表已存在，4 个版本被记录。
      Applied: V023, V024, V025, V061

  Schema 推断结果（基于 sqlmigrate_mysql/ 标志检测）：
    [✓] 已应用 (4): V023, V024, V025, V056
    [⚠️ ] V072 BIGINT 修复未应用（id 列仍为 INT）→ 需手工跑：
          mysql -h $MYSQL_HOST -u $MYSQL_USER -p < sqlmigrate_mysql/V072__fix_int_to_bigint.sql

======================================================================
  inkframe_dev_product  (logical=product)
======================================================================
  [✓] _schema_migrations 表已存在，5 个版本被记录。
      Applied: V026, V027, V028, V029, V061

  Schema 推断结果（基于 sqlmigrate_mysql/ 标志检测）：
    [✓] 已应用 (4): V026, V027, V028, V029
    [⚠️ ] V072 BIGINT 修复未应用（id 列仍为 INT）→ 需手工跑：
          mysql -h $MYSQL_HOST -u $MYSQL_USER -p < sqlmigrate_mysql/V072__fix_int_to_bigint.sql

======================================================================
  inkframe_dev_order  (logical=order)
======================================================================
  [✓] _schema_migrations 表已存在，8 个版本被记录。
      Applied: V030, V031, V032, V033, V034, V035, V058, V061

  Schema 推断结果（基于 sqlmigrate_mysql/ 标志检测）：
    [✓] 已应用 (8): V030, V031, V032, V033, V034, V035, V058, V078
    [⚠️ ] V072 BIGINT 修复未应用（id 列仍为 INT）→ 需手工跑：
          mysql -h $MYSQL_HOST -u $MYSQL_USER -p < sqlmigrate_mysql/V072__fix_int_to_bigint.sql

======================================================================
  inkframe_dev_payment  (logical=payment)
======================================================================
  [✓] _schema_migrations 表已存在，3 个版本被记录。
      Applied: V036, V037, V061

  Schema 推断结果（基于 sqlmigrate_mysql/ 标志检测）：
    [✓] 已应用 (2): V036, V037
    [✗] 未应用/部分应用 (1):
        - V099: payment_transactions.pay_channel

======================================================================
  inkframe_dev_device  (logical=device)
======================================================================
  [✓] _schema_migrations 表已存在，10 个版本被记录。
      Applied: V040, V041, V042, V043, V059, V061, V063, V064, V068, V071

  Schema 推断结果（基于 sqlmigrate_mysql/ 标志检测）：
    [✓] 已应用 (13): V040, V041, V042, V043, V059, V063, V064, V071, V073, V076, V077, V085, V086
    [✗] 未应用/部分应用 (1):
        - V068: 部分命中 9/10: devices.token_version
    [?] 无法判断 (1): V091 (纯数据迁移，无 schema 标志)
    [⚠️ ] V072 BIGINT 修复未应用（id 列仍为 INT）→ 需手工跑：
          mysql -h $MYSQL_HOST -u $MYSQL_USER -p < sqlmigrate_mysql/V072__fix_int_to_bigint.sql

======================================================================
  inkframe_dev_notification  (logical=notification)
======================================================================
  [✓] _schema_migrations 表已存在，5 个版本被记录。
      Applied: V044, V045, V046, V047, V061

  Schema 推断结果（基于 sqlmigrate_mysql/ 标志检测）：
    [✓] 已应用 (4): V044, V045, V046, V047
    [⚠️ ] V072 BIGINT 修复未应用（id 列仍为 INT）→ 需手工跑：
          mysql -h $MYSQL_HOST -u $MYSQL_USER -p < sqlmigrate_mysql/V072__fix_int_to_bigint.sql

======================================================================
  inkframe_dev_ai  (logical=ai)
======================================================================
  [✓] _schema_migrations 表已存在，6 个版本被记录。
      Applied: V048, V049, V061, V062, V065, V069

  Schema 推断结果（基于 sqlmigrate_mysql/ 标志检测）：
    [✓] 已应用 (10): V048, V049, V062, V065, V069, V074, V075, V081, V084, V088
    [⚠️ ] V072 BIGINT 修复未应用（id 列仍为 INT）→ 需手工跑：
          mysql -h $MYSQL_HOST -u $MYSQL_USER -p < sqlmigrate_mysql/V072__fix_int_to_bigint.sql

======================================================================
  inkframe_dev_points  (logical=points)
======================================================================
  [✓] _schema_migrations 表已存在，5 个版本被记录。
      Applied: V050, V051, V052, V056, V061

  Schema 推断结果（基于 sqlmigrate_mysql/ 标志检测）：
    [✓] 已应用 (4): V050, V051, V052, V056
    [⚠️ ] V072 BIGINT 修复未应用（id 列仍为 INT）→ 需手工跑：
          mysql -h $MYSQL_HOST -u $MYSQL_USER -p < sqlmigrate_mysql/V072__fix_int_to_bigint.sql

======================================================================
  inkframe_dev_admin  (logical=admin)
======================================================================
  [✓] _schema_migrations 表已存在，5 个版本被记录。
      Applied: V010, V011, V061, V066, V070

  Schema 推断结果（基于 sqlmigrate_mysql/ 标志检测）：
    [✓] 已应用 (1): V010
    [?] 无法判断 (6): V011 (纯数据迁移，无 schema 标志), V066 (纯数据迁移，无 schema 标志), V070 (纯数据迁移，无 schema 标志), V074 (纯数据迁移，无 schema 标志), V080 (纯数据迁移，无 schema 标志) ...
    [⚠️ ] V072 BIGINT 修复未应用（id 列仍为 INT）→ 需手工跑：
          mysql -h $MYSQL_HOST -u $MYSQL_USER -p < sqlmigrate_mysql/V072__fix_int_to_bigint.sql

======================================================================
  推荐操作
======================================================================

  • 部分库已有 _schema_migrations 表 → 这些库直接跑迁移即可：
      python migrate_manual_mysql.py --db-prefix inkframe_dev_ --dry-run

      **************

      
rabbitmqctl add_vhost inkframe_dev
rabbitmqctl set_permissions -p inkframe_dev inkframe ".*" ".*" ".*"


 docker logs  inkframe-dev-gateway-1 2>&1|grep -E "ERROR|Traceback|500" |tail -50
INFO:     180.165.11.223:2311 - "POST /api/v1/user/auth/login HTTP/1.1" 500 Internal Server Error
INFO:     180.165.11.223:2313 - "POST /api/v1/user/auth/login HTTP/1.1" 500 Internal Server Error
INFO:     180.165.11.223:2314 - "POST /api/v1/user/auth/login HTTP/1.1" 500 Internal Server Error
INFO:     180.165.11.223:2316 - "POST /api/v1/user/auth/login HTTP/1.1" 500 Internal Server Error
INFO:     180.165.11.223:2317 - "POST /api/v1/user/auth/login HTTP/1.1" 500 Internal Server Error
INFO:     180.165.11.223:2318 - "GET /api/v1/artwork/gallery/list?page=1&page_size=20&sort=latest HTTP/1.1" 500 Internal Server Error
INFO:     180.165.11.223:2320 - "GET /api/v1/artwork/gallery/list?page=1&page_size=20&sort=latest HTTP/1.1" 500 Internal Server Error
INFO:     180.165.11.223:2321 - "POST /api/v1/user/auth/login HTTP/1.1" 500 Internal Server Error
[root@iZuf62khg8ourx3stkboshZ xiyiart-pre]# docker logs --tail 200  inkframe-dev-user_service-1
2026-05-07 13:27:03,671 | INFO     | user_service | user_service | 正在启动 UserService gRPC (port=60051)...
2026-05-07 13:27:03,671 | INFO     | user_service | user_service | DATABASE_URL=mysql+aiomysql://inkframe:www.71AD.comxiyi@host.docker.internal:3306/inkframe_dev_user
2026-05-07 13:27:03,699 | DEBUG    | user_service | aiomysql | caching sha2: succeeded by fast path.
2026-05-07 13:27:03,800 | INFO     | user_service | user_service | RabbitMQ 发布器连接成功
2026-05-07 13:27:03,894 | INFO     | user_service | user_service | RabbitMQ 消费器已启动，监听 chain_account.created / artist.kyc_approved / artist.kyc_rejected
2026-05-07 13:27:03,996 | INFO     | user_service | user_service | Redis 已连接 url=redis://:www.71AD.comxiyi@host.docker.internal:6379/1
2026-05-07 13:27:04,000 | INFO     | user_service | user_service | notification_service stub 创建成功 addr=notification_service:60059
2026-05-07 13:27:04,005 | INFO     | user_service | user_service | UserService gRPC 已启动，监听 [::]:60051

