# pg-mcp-server 로컬 셋업

**유형:** 하네스 메타 변경
**대상:** `repositories/pg-mcp-server` (ryan-papa/pg-mcp-server fork, stuzero/pg-mcp-server upstream)
**날짜:** 2026-05-22

## 변경 이유

Claude Code가 mac-mini-infra의 PostgreSQL(`museum_finder` DB)을 read-only로 조회할 수 있도록 SSE MCP 서버를 콜리마 단일 도커 컨테이너로 영속 운영.

| 요구 | 해결 |
|------|------|
| 클라이언트 자격증명이 MCP 대화·config에 남는 노출 | 서버 부팅 시 env(`PG_DSN`)로 자동 register → 클라이언트는 인자 없이 `connect()` 호출 |
| mac-mini PG가 mTLS 요구 (client cert + key 필수) | `_build_ssl_ctx_from_env` + named external volume `pg-mcp-certs`로 cert 마운트 |
| 평문 시크릿 디스크 보관 금지 (CLAUDE.md SSOT) | `sops+age` 암호문(`.env.enc`)만 보관, `scripts/up.sh`가 메모리로만 복호화 후 compose 주입 |
| Read-only 이중 방어 | DB role `rp_readonly` + asyncpg pool `default_transaction_read_only=on` |

## 영향 파일

**fork 레포 (`repositories/pg-mcp-server/`)**

| 파일 | 변경 |
|------|------|
| `server/tools/connection.py` | boot-time auto-register, `connect` 인자 빈값 시 `DEFAULT_DSN` fallback, `disconnect`로 default 끊김 방지 |
| `server/database.py` | `_build_ssl_ctx_from_env` 추가, `PG_SSL_MODE`(verify-full/verify-ca/require) 분기, `asyncpg.create_pool`에 ssl context 전달 |
| `docker-compose.yml` | env passthrough, named external volume, `host.docker.internal:host-gateway`, restart unless-stopped, log rotation 10m×5, healthcheck, port `${PG_MCP_BIND_IP:-0.0.0.0}:38000:8000` |
| `scripts/up.sh`, `scripts/down.sh` | sops 메모리 주입 후 compose up/down |
| `.sops.yaml`, `.env.enc` | sops+age 시크릿 (공개키 1개) |

**workflow-agent-harness 루트**

| 파일 | 변경 |
|------|------|
| `docs/security/recipients.local.md` | mac-main 공개키 등록 (gitignore) |
| `docs/prd/pg-mcp-server-setup/prd.md` | 본 문서 |

## 롤백 전략

| 단계 | 명령 |
|------|------|
| 컨테이너 정지·제거 | `repositories/pg-mcp-server/scripts/down.sh` |
| 볼륨·이미지 제거 | `docker volume rm pg-mcp-certs && docker rmi pg-mcp-server-pg-mcp` |
| fork 코드 되돌리기 | `cd repositories/pg-mcp-server && git checkout -- server/ docker-compose.yml` |
| sops 도구 제거 (다른 레포 미사용 시) | `brew uninstall sops age` |
| age key 폐기 | `~/.config/sops/age/keys.txt` 삭제 → `recipients.local.md`에서 제거 (재발급 시 `.env.enc` 재암호화 필요) |
| Claude Code MCP 등록 해제 | `claude mcp remove pg-mcp` 또는 `~/.claude.json`의 `mcpServers.pg-mcp` 항목 삭제 |

## 검증

| 검증 | 명령 / 결과 |
|------|------|
| 컨테이너 상태 | `docker ps --filter name=pg-mcp` → `Up … (healthy)` |
| 실 PG 접속 | `docker exec pg-mcp uv run python -c "…"` → user `rp_readonly`, db `museum_finder` |
| read-only 강제 | `SHOW default_transaction_read_only` → `on` |
| 쓰기 차단 | `CREATE TABLE _probe(id int)` → `ReadOnlySQLTransactionError` |
| 스키마 인식 | `SELECT count(*) FROM information_schema.tables WHERE table_schema='public'` → `50` |
| idle 메모리 | `docker stats --no-stream pg-mcp` → ≈ 85 MiB |
| LAN 접근 | `curl -o /dev/null -w '%{http_code}' http://192.168.0.31:38000/sse` → `200` |
| MCP 등록 | Claude Code에서 `Pg-mcp` 도구 목록(`connect`/`disconnect`/`pg_query`/`pg_explain`) 노출 + 인자 없이 `connect()` 호출 시 conn_id 반환 |

## Open Issues

- `LocalDevCA`에 BasicConstraints `CA:TRUE` 미박힘 → 현재 `PG_SSL_MODE=require`(서버 cert 검증 skip, mTLS는 유지). 정석은 `mac-mini-infra` CA 재발급 후 `verify-ca`로 환원. **별도 작업 분리.**
- 현재 `0.0.0.0:38000` 바인딩이라 LAN 모든 기기 접근 가능. 인증 없음. LAN 신뢰 환경 전제. 향후 nginx + IP allowlist / mTLS reverse proxy 검토 가능.
- 공개키 1개(`mac-main`)만 등록. 다른 기기 추가 시 `.sops.yaml` 동기화 + `.env.enc` 재암호화 필요.
