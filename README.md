# 1. 直接看最近 500 行原始日志（traceback 是 25+ 行）
docker logs --tail 500 inkframe-dev-gateway-1 2>&1 | tail -200

# 2. 抓 Traceback 前后 30 行
docker logs --tail 1000 inkframe-dev-gateway-1 2>&1 | grep -B 3 -A 30 "Traceback"

# 3. 抓所有 ERROR 级别日志（exception_handler 用 logger.exception 写）
docker logs --tail 1000 inkframe-dev-gateway-1 2>&1 | grep -B1 -A 25 "ERROR"

# 4. 复现的同时实时看（最直观）
docker logs -f inkframe-dev-gateway-1 &
curl -i -X POST http://localhost:8009/api/v1/user/auth/login \
  -H "Content-Type: application/json" \
  -d '{"identifier":"13800000000","password":"abc123456"}'
一个关键观察
你的日志里 /artwork/gallery/list 也 500 了——和登录是两个完全不同的下游服务（user_service vs artwork_service），却都 500。这说明问题大概率不在某个具体下游，而在 gateway 全局。

排查方向（按命中率）：

gRPC channel 还没建 — gateway 启动时各 client 是 lazy connect，但 connect() 由 lifespan 调用。如果 lifespan 里某一步失败，channel 是 None，每个路由调用都会抛 RuntimeError("XxxClient 未连接") → 落到 unhandled handler → 500
gateway 连不通某个共享依赖 —— 比如 rate_limiter 用 Redis、admin_db 用 MySQL；启动时连接失败但应用没崩，每个请求过中间件就抛
Consul / 环境变量未注入 —— 但 gateway 用的是固定 DNS xxx_service:60051（见上一轮分析），不依赖 Consul
顺手查这两件事

# 1. gateway 启动阶段是否报错（startup/lifespan）
docker logs inkframe-dev-gateway-1 2>&1 | head -100

# 2. gateway 容器里能否解析下游服务 DNS
docker exec inkframe-dev-gateway-1 getent hosts user_service
docker exec inkframe-dev-gateway-1 getent hosts artwork_service

# 3. gateway 容器里能否 ping 通 host.docker.internal 的 Redis/MySQL
docker exec inkframe-dev-gateway-1 sh -c 'getent hosts host.docker.internal'

INFO:     172.24.0.8:33828 - "GET /metrics HTTP/1.1" 401 Unauthorized
INFO:     31.94.24.193:57304 - "POST /api/v1/user/auth/login HTTP/1.1" 500 Internal Server Error
INFO:     127.0.0.1:34066 - "GET /health HTTP/1.1" 200 OK
docker logs --tail 1000 inkframe-dev-gateway-1 2>&1 | grep -B 3 -A 30 "Traceback"
输出空
docker logs --tail 1000 inkframe-dev-gateway-1 2>&1 | grep -B1 -A 25 "ERROR"
输出空
  HTTP/1.1 500 Internal Server Error
date: Thu, 07 May 2026 14:00:26 GMT
server: uvicorn
content-length: 183
content-type: application/json

{"success":false,"code":500,"message":"服务内部错误: internal error: OperationalError","data":null,"timestamp":"2026-05-07T14:00:27.213895+00:00","requestId":"req_b0e3cdfd39c7"}%     
  
  docker exec inkframe-dev-gateway-1 getent hosts user_service
172.24.0.10     user_service
docker exec inkframe-dev-gateway-1 sh -c 'getent hosts host.docker.internal'
 docker exec inkframe-dev-gateway-1 sh -c 'getent hosts host.docker.internal'
172.17.0.1      host.docker.internal

# 1. 看 inkframe_dev_user 库是否存在 + 表数量
docker exec -it $(docker ps -qf name=mysql) \
  mysql -uinkframe -p'www.71AD.comxiyi' -e \
  "SHOW DATABASES LIKE 'inkframe_dev%'; USE inkframe_dev_user; SHOW TABLES;"

# 如果你 MySQL 不是跑在容器里，直接用宿主机 client：
mysql -uinkframe -p'www.71AD.comxiyi' -h 127.0.0.1 -e \
  "SHOW DATABASES LIKE 'inkframe_dev%'; USE inkframe_dev_user; SHOW TABLES;"

  [root@iZuf62khg8ourx3stkboshZ ~]# mysql -uinkframe -p'www.71AD.comxiyi' -h 127.0.0.1 -e \
>   "SHOW DATABASES LIKE 'inkframe_dev%'; USE inkframe_dev_user; SHOW TABLES;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------------------+
| Database (inkframe_dev%)  |
+---------------------------+
| inkframe_dev_admin        |
| inkframe_dev_ai           |
| inkframe_dev_artist       |
| inkframe_dev_artwork      |
| inkframe_dev_copyright    |
| inkframe_dev_device       |
| inkframe_dev_notification |
| inkframe_dev_order        |
| inkframe_dev_payment      |
| inkframe_dev_points       |
| inkframe_dev_product      |
| inkframe_dev_user         |
| inkframe_device           |
+---------------------------+
+-----------------------------+
| Tables_in_inkframe_dev_user |
+-----------------------------+
| _schema_migrations          |
| membership_plans            |
| user_addresses              |
| user_follows                |
| user_memberships            |
| user_notification_prefs     |
| user_oauth                  |
| user_oauth_bindings         |
| user_sessions               |
| users                       |
| verification_codes          |
+-----------------------------+
>
> 库和表都建好了，那问题在具体 SQL 执行环节。OperationalError 一定有 traceback，刚才你只看了 user_service 的开头日志。现在按下面这一步精准抓：

复现并实时抓 user_service 的 traceback

# 终端 A：跟 user_service 日志
docker logs -f --tail 0 inkframe-dev-user_service-1

# 终端 B：发请求触发
curl -i -X POST http://localhost:8009/api/v1/user/auth/login \
  -H "Content-Type: application/json" \
  -d '{"identifier":"13800000000","password":"abc123456"}'
或事后捞：


docker logs --since 10m inkframe-dev-user_service-1 2>&1 | grep -B 3 -A 40 -iE "traceback|operationalerror|sqlalchemy"
同时检查这几个高命中点
1. 表结构是否到位（迁移有没有跑全）
users 表里有几个登录流程必需的字段，看一下：


mysql -uinkframe -p'www.71AD.comxiyi' -h 127.0.0.1 inkframe_dev_user -e \
  "DESC users; SELECT version FROM _schema_migrations ORDER BY version;"
把 users 表结构和已应用的版本号贴出来。重点看是否有 phone、email、username、password_hash、status 这几列。

2. 登录账户是否存在

mysql -uinkframe -p'www.71AD.comxiyi' -h 127.0.0.1 inkframe_dev_user -e \
  "SELECT id, username, email, phone, status FROM users LIMIT 5;"
如果一行没有，那登录查不到用户，但应该是 UNAUTHENTICATED 而不是 OperationalError——除非查询本身就报错。

3. user_service 实际跑的是哪条 SQL
Bash Find login query in user_service
IN
grep -rnE "def.*login|by_identifier|find.*user" /Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/infrastructure/persistence/ 2>/dev/null | head -10

OUT
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/infrastructure/persistence/sql_oauth_repository.py:27:    async def find_by_user_id(
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/infrastructure/persistence/sql_oauth_repository.py:41:    async def find_by_user_and_provider(
Bash Find login command handler
IN
grep -rnE "Login|login|identifier" /Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/application/ 2>/dev/null | head -10

OUT
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/application/queries/admin_stats_query.py:6:- active_today:   今日活跃用户数（UserModel 暂无 last_login_at 列，返回 0 占位）
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/application/queries/admin_stats_query.py:7:                  TODO: 待 last_login_at 列就绪后替换占位逻辑
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/application/queries/admin_stats_query.py:26:    active_today: int  # 暂固定为 0，等 last_login_at 列就绪后实现
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/application/queries/admin_stats_query.py:62:            # 今日活跃用户数：UserModel 当前无 last_login_at 列，返回 0 占位
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/application/queries/admin_stats_query.py:63:            # TODO: 待 last_login_at 列添加后，替换为：
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/application/queries/admin_stats_query.py:65:            #       UserModel.last_login_at >= today_start)
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/application/queries/authenticate_user.py:16:    identifier: str  # username / email / phone 之一
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/application/queries/authenticate_user.py:21:    """按 identifier 查用户并校验密码。
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/application/queries/authenticate_user.py:33:        user = await self._repo.get_by_username(query.identifier)
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/application/queries/authenticate_user.py:35:            user = await self._repo.get_by_email(query.identifier)
Read authenticate_user.py (lines 26-75)
Bash Inspect UserModel columns
IN
grep -nE "class UserModel|Column|__tablename__" /Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/infrastructure/persistence/sql_user_repository.py 2>/dev/null | head -30

OUT
(Bash completed with no output)
Bash Locate UserModel
IN
find /Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/ -name "*.py" | xargs grep -ln "class UserModel" 2>/dev/null

OUT
/Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/infrastructure/persistence/user_model.py
Bash Show UserModel columns
IN
grep -nE "Column|mapped_column|__tablename__" /Users/bitshare/Desktop/panhx/xiyiart/xiyiart/services/user_service/user_service/infrastructure/persistence/user_model.py

OUT
11:from sqlalchemy.orm import Mapped, mapped_column
17:    __tablename__ = "users"
19:    username: Mapped[Optional[str]] = mapped_column(
22:    email: Mapped[Optional[str]] = mapped_column(
25:    phone: Mapped[Optional[str]] = mapped_column(
29:    phone_verified_at: Mapped[Optional[object]] = mapped_column(
33:    password_hash: Mapped[str] = mapped_column(String(128), nullable=False)
35:    nickname: Mapped[Optional[str]] = mapped_column(String(50), nullable=True)
36:    avatar_url: Mapped[Optional[str]] = mapped_column(
39:    avatar_width: Mapped[Optional[int]] = mapped_column(Integer, nullable=True)
40:    avatar_height: Mapped[Optional[int]] = mapped_column(Integer, nullable=True)
41:    bio: Mapped[Optional[str]] = mapped_column(Text, nullable=True)
43:    role: Mapped[str] = mapped_column(String(20), nullable=False, default="user")
44:    status: Mapped[str] = mapped_column(
47:    membership_level: Mapped[str] = mapped_column(
51:    chain_address: Mapped[Optional[str]] = mapped_column(
login 查询会按顺序查 username / email / phone（authenticate_user.py:33-37），SELECT 出 UserModel 的全部列。最常见的 OperationalError 是 某个迁移漏跑导致列缺失。UserModel 期望以下列：


username  email  phone  phone_verified_at  password_hash
nickname  avatar_url  avatar_width  avatar_height  bio
role  status  membership_level  chain_address
这两条命令一起跑，把输出贴回来

# 1. users 表实际结构 + 已应用的迁移版本
mysql -uinkframe -p'www.71AD.comxiyi' -h 127.0.0.1 inkframe_dev_user -e \
  "DESC users; SELECT version FROM _schema_migrations ORDER BY version;"

# 2. user_service 最近的 traceback（复现一次再抓）
docker logs --since 5m inkframe-dev-user_service-1 2>&1 | grep -B 3 -A 40 -iE "traceback|operationalerror|unknown column|sqlalchemy"
第 1 条会告诉我"哪一列缺了"，第 2 条会直接告诉我"哪一行 SQL 炸了"。两个加起来基本就能一锤定音。

注：你刚才贴的 dev_user 表已有 users / user_addresses / user_oauth / user_oauth_bindings / user_sessions / user_memberships / user_notification_prefs / user_follows / verification_codes / membership_plans 共 10 张业务表，少了 V003 对应的 user_oauth 不在我列表里但已有——大致都到位，但具体列差异要 DESC 才能看出来。




[root@iZuf62khg8ourx3stkboshZ ~]# mysql -uinkframe -p'www.71AD.comxiyi' -h 127.0.0.1 inkframe_dev_user -e \
>   "DESC users; SELECT version FROM _schema_migrations ORDER BY version;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+-----------------------+--------------+------+-----+-------------------+-------------------+
| Field                 | Type         | Null | Key | Default           | Extra             |
+-----------------------+--------------+------+-----+-------------------+-------------------+
| id                    | int          | NO   | PRI | NULL              | auto_increment    |
| username              | varchar(50)  | YES  | UNI | NULL              |                   |
| email                 | varchar(100) | YES  | UNI | NULL              |                   |
| phone                 | varchar(20)  | YES  | UNI | NULL              |                   |
| password_hash         | varchar(128) | NO   |     | NULL              |                   |
| nickname              | varchar(50)  | YES  |     | NULL              |                   |
| avatar_url            | varchar(500) | YES  |     | NULL              |                   |
| bio                   | text         | YES  |     | NULL              |                   |
| gender                | varchar(10)  | YES  |     | NULL              |                   |
| birthday              | date         | YES  |     | NULL              |                   |
| city                  | varchar(100) | YES  |     | NULL              |                   |
| role                  | varchar(20)  | NO   | MUL | user              |                   |
| status                | varchar(20)  | NO   | MUL | active            |                   |
| membership_level      | varchar(20)  | NO   |     | free              |                   |
| membership_expires_at | datetime     | YES  | MUL | NULL              |                   |
| chain_address         | varchar(100) | YES  | UNI | NULL              |                   |
| chain_private_key_enc | varchar(500) | YES  |     | NULL              |                   |
| chain_created_at      | datetime     | YES  |     | NULL              |                   |
| created_at            | datetime     | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
| updated_at            | datetime     | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
| avatar_width          | int          | YES  |     | NULL              |                   |
| avatar_height         | int          | YES  |     | NULL              |                   |
+-----------------------+--------------+------+-----+-------------------+-------------------+
+---------+
| version |
+---------+
| V001    |
| V002    |
| V003    |
| V004    |
| V012    |
| V013    |
| V014    |
| V015    |
| V016    |
| V060    |
| V061    |
+---------+
[root@iZuf62khg8ourx3stkboshZ ~]# docker logs --since 5m inkframe-dev-user_service-1 2>&1 | grep -B 3 -A 40 -iE "traceback|operationalerror|unknown column|sqlalchemy"
>
> 
> 
# 切到项目目录
cd /path/to/xiyiart-pre   # 你的项目根目录

# 1. 把 dev 库一次性补全
DB_PREFIX=inkframe_dev_ \
DB_USER=inkframe \
DB_PASS='www.71AD.comxiyi' \
DB_HOST=127.0.0.1 \
  bash scripts/migrate_mysql.sh

# 2. 重启 dev 微服务（迁移期间最好先停服避免连接报错）
docker compose -f docker-compose.dev.yml restart

# 3. 验证 user_service
mysql -uinkframe -p'www.71AD.comxiyi' -h 127.0.0.1 inkframe_dev_user -e \
  "SELECT version FROM _schema_migrations ORDER BY version;"
# 期望看到：V001 V002 V003 V004 V012 V013 V014 V015 V016 V055 V060 V061 V098

# 4. 复测登录
curl -i -X POST http://localhost:8009/api/v1/user/auth/login \
  -H "Content-Type: application/json" \
  -d '{"identifier":"13800000000","password":"abc123456"}'



