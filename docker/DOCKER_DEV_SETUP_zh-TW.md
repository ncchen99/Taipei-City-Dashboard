# Taipei City Dashboard Docker 開發環境設定指南

這份文件是依據專案目前倉庫中的設定檔整理，包含：
- 根目錄 README
- `docker/.env.template`
- `docker/docker-compose-db.yaml`
- `docker/docker-compose-init.yaml`
- `docker/docker-compose.yaml`
- `docker/qdrant-upgrade/README.md`
- FE/BE 程式碼中的實際環境變數讀取

## 1. 你現在可直接使用的設定檔

已建立可用的環境檔：
- `docker/.env`

你至少要調整下列值再啟動：
- `JWT_SECRET`
- `IDNO_SALT`
- `DASHBOARD_DEFAULT_PASSWORD`
- `QDRANT_API_KEY`
- `TWCC_API_KEY`（若要啟用 TWCC）
- `VITE_MAPBOXTOKEN`、`VITE_MAPBOXTILE`（若要完整地圖底圖）

## 2. 啟動順序（Docker 開發模式）

在專案根目錄執行：

```bash
cd docker

# 1) 建立外部 network（compose 檔要求 external network）
docker network create --driver=bridge --subnet=192.168.128.0/24 --gateway=192.168.128.1 br_dashboard

# 2) 啟動 DB/Cache/Qdrant
docker compose -f docker-compose-db.yaml up -d

# 3) 初始化前後端依賴與 DB sample data
docker compose -f docker-compose-init.yaml up --abort-on-container-exit

# 4) 啟動 Nginx + FE + BE
docker compose -f docker-compose.yaml up -d
```

檢查服務狀態：

```bash
docker compose -f docker-compose-db.yaml ps
docker compose -f docker-compose.yaml ps
```

檢查日誌：

```bash
docker compose -f docker-compose.yaml logs -f dashboard-be
docker compose -f docker-compose.yaml logs -f dashboard-fe
docker compose -f docker-compose-db.yaml logs -f qdrant
```

## 3. 預設對外連接埠

- FE (Vite): `http://localhost:8080`
- BE (Gin): `http://localhost:8088`
- Nginx: `http://localhost:80` / `https://localhost:443`
- pgAdmin: `http://localhost:8889`
- Postgres manager: `localhost:5432`
- Qdrant: `http://localhost:6333`

## 4. 環境變數清單（詳細）

以下分類為：
- 必填：不填通常無法正常啟動或功能會壞
- 建議填：可啟動但建議設定
- 可留空：外部開發可先留空

### A. 映像與執行基本設定

- `NGINX_IMAGE_TAG`：建議填，Nginx image tag（預設 latest）
- `NODE_IMAGE_TAG`：建議填，Node image tag
- `GOLANG_IMAGE_TAG`：建議填，初始化容器用 Go image tag
- `NODE_ENV`：建議填，FE 執行模式（development）
- `GIN_MODE`：建議填，BE 模式（debug/release/test）
- `GIN_DOMAIN`：建議填，開發建議 `0.0.0.0`
- `GIN_PORT`：建議填，BE 容器內 port（預設 8080）
- `PORT`：建議填，compose 對映使用（預設 8080）

### B. FE 變數

- `VITE_API_URL`：必填，前端 API base（docker 模式通常 `/api/dev`）
- `VITE_APP_TITLE`：建議填，前端標題
- `VITE_APP_VERSION`：建議填，前端版本字串
- `VITE_MAPBOXTOKEN`：建議填，Mapbox token
- `VITE_MAPBOXTILE`：建議填，Mapbox tile source URL
- `VITE_TAIPEIPASS_URL`：可留空，台北通 OAuth URL
- `VITE_TAIPEIPASS_CLIENT_ID`：可留空，台北通 client id
- `VITE_TAIPEIPASS_SCOPE`：可留空，台北通 scope
- `VITE_PERSONAL_BOARD_UPDATE`：可留空，個人看板刷新設定

### C. BE 安全與登入

- `JWT_SECRET`：必填，JWT 簽章密鑰
- `IDNO_SALT`：必填，身分資料 hash salt
- `DASHBOARD_DEFAULT_USERNAME`：建議填，初始化管理員帳號
- `DASHBOARD_DEFAULT_Email`：建議填，初始化管理員 email
- `DASHBOARD_DEFAULT_PASSWORD`：建議填，初始化管理員密碼
- `ISSO_URL`：可留空，外部 SSO URL
- `TAIPEIPASS_URL`：可留空，外部 TaipeiPass URL
- `ISSO_CLIENT_ID`：可留空，SSO client id
- `ISSO_CLIENT_SECRET`：可留空，SSO secret
- `ISSO_CLIENT_SCOPE`：可留空，SSO scope

### D. 資料庫（Dashboard）

- `DB_DASHBOARD_HOST`：必填（docker 內預設 `postgres-data`）
- `DB_DASHBOARD_PORT`：必填（通常 5432）
- `DB_DASHBOARD_USER`：必填
- `DB_DASHBOARD_PASSWORD`：必填
- `DB_DASHBOARD_DBNAME`：必填
- `DB_DASHBOARD_SSLMODE`：建議填（docker 內通常 `disable`）
- `DASHBOARD_SAMPLE_FILE`：建議填（預設 `dashboard-demo.sql`）

### E. 資料庫（Manager）

- `DB_MANAGER_HOST`：必填（docker 內預設 `postgres-manager`）
- `DB_MANAGER_PORT`：必填（通常 5432）
- `DB_MANAGER_USER`：必填
- `DB_MANAGER_PASSWORD`：必填
- `DB_MANAGER_DBNAME`：必填
- `DB_MANAGER_SSLMODE`：建議填（docker 內通常 `disable`）
- `MANAGER_SAMPLE_FILE`：建議填（預設 `dashboardmanager-demo.sql`）

### F. Redis

- `REDIS_HOST`：必填（docker 內預設 `redis`）
- `REDIS_PORT`：必填（通常 6379）
- `REDIS_DB`：建議填（通常 0）
- `REDIS_PASSWORD`：可留空（目前 compose 未強制）

### G. pgAdmin

- `PGADMIN_DEFAULT_EMAIL`：建議填
- `PGADMIN_DEFAULT_PASSWORD`：建議填
- `PGADMIN_LISTEN_PORT`：建議填（通常 80）

### H. Qdrant / 向量搜尋

- `QDRANT_URL`：必填（docker 內通常 `http://qdrant:6333`）
- `QDRANT_API_KEY`：必填（`docker-compose-db.yaml` 有啟用 API key）
- `QDRANT_COLLECTION`：建議填（BE 全域設定使用）
- `QDRANT_COLLECTION_NAME`：建議填（`app/services/qdrant.go` 使用）

### I. 模型與 TWCC

- `LM_MODEL_PATH`：建議填，ONNX 模型在容器內的路徑。Dockerfile 將模型固定 COPY 至 `/opt/lm_model/onnx-e5`，因此此值應設為 `/opt/lm_model/onnx-e5/`（BE 程式碼預設值與此一致）。
- `TWCC_API_URL`：可留空（有預設）
- `TWCC_API_KEY`：若使用 TWCC 則必填
- `TWCC_MODEL`：建議填
- `TWCC_TIMEOUT`：建議填
- `TWCC_MAX_RETRY`：建議填
- `TWCC_MAX_CONCURRENT`：建議填

## 5. 目前文件/程式中的命名差異（已在 docker/.env 一併兼容）

- `docker/.env.template` 內是 `NGINX_IMAGE_tag`，但 compose 使用 `NGINX_IMAGE_TAG`
- BE 全域設定使用 `QDRANT_COLLECTION`，`app/services/qdrant.go` 使用 `QDRANT_COLLECTION_NAME`
- `LM_MODEL_PATH` 的路徑由 `Dockerfile` 決定（`COPY --from=model_export /out/onnx-e5 /opt/lm_model/onnx-e5`），正確值為 `/opt/lm_model/onnx-e5/`。舊版文件或 `.env` 若使用 `/opt/Taipei-City-Dashboard-BE/lm_model/onnx-e5/` 等其他路徑，容器啟動時向量搜尋功能將因找不到模型而失敗。

為避免踩雷，`docker/.env` 兩組都已放入。

## 6. 常用除錯指令

```bash
# 查看所有容器
docker ps -a

# 僅看 dashboard 服務
docker compose -f docker-compose.yaml ps

# 重新初始化資料（會重跑 init 容器）
docker compose -f docker-compose-init.yaml down
docker compose -f docker-compose-init.yaml up --abort-on-container-exit

# 關閉主服務
docker compose -f docker-compose.yaml down

# 關閉 DB 服務
docker compose -f docker-compose-db.yaml down
```
