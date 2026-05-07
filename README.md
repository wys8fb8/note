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
