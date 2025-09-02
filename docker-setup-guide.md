# 🐳 Jenkins + Ant Docker Compose 使用指南

## 📋 專案概述

這是你原有 Jenkins-Ant-Practice 專案的 Docker Compose 升級版本，專注於：
- ✅ Jenkins CI/CD 自動化
- ✅ Apache Ant 建置工具整合
- ✅ 簡潔、實用的容器化架構

## 🚀 快速開始

### 1. 停止現有的 Jenkins 容器
```bash
# 停止並移除舊的容器
docker stop jenkins-practice
docker rm jenkins-practice
```

### 2. 使用 Docker Compose 啟動
```bash
# 啟動服務
docker-compose up -d

# 查看服務狀態
docker-compose ps

# 查看 Jenkins 日誌
docker-compose logs -f jenkins
```

### 3. 訪問服務
- **Jenkins Web UI**: http://localhost:8080
- **初始密碼獲取**: 
  ```bash
  docker-compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
  ```

## 📁 目錄結構

```
Jenkins-Ant-Practice/
├── docker-compose.yml          # 主要服務配置
├── docker-compose.override.yml # 開發環境覆蓋配置
├── build.xml                   # 你的 Ant 建置腳本
├── Jenkinsfile                 # 你的 Pipeline 定義
├── README.md                   # 原有文檔
└── docker-setup-guide.md      # 本文檔
```

## 🔧 與你原有專案的整合

### Jenkins Pipeline 更新
在你的 `Jenkinsfile` 中，build.xml 現在可以直接存取：
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                // build.xml 在 /workspace 目錄下
                sh 'cd /workspace && ant all'
            }
        }
    }
}
```

### Ant 工具安裝
Jenkins 容器啟動後，需要安裝 Ant：
```bash
# 進入 Jenkins 容器
docker-compose exec jenkins bash

# 安裝 Ant
apt-get update && apt-get install -y ant

# 驗證安裝
ant -version
```

## 🛠️ 進階配置

### 啟用 Jenkins Agent (可選)
如果需要分散式建置：
```bash
# 啟動包含 Agent 的完整配置
docker-compose --profile agents up -d
```

### 資料備份
```bash
# 備份 Jenkins 資料
docker run --rm -v jenkins-ant-practice_jenkins_home:/data -v $(pwd):/backup ubuntu tar czf /backup/jenkins-backup.tar.gz -C /data .

# 恢復 Jenkins 資料
docker run --rm -v jenkins-ant-practice_jenkins_home:/data -v $(pwd):/backup ubuntu tar xzf /backup/jenkins-backup.tar.gz -C /data
```

## 🎯 相比單一容器的優勢

| 特性 | 單一容器 | Docker Compose |
|------|----------|----------------|
| **服務管理** | 手動 | 自動化 |
| **網路配置** | 手動端口 | 自動網路 |
| **資料持久化** | 手動Volume | 聲明式管理 |
| **擴展性** | 困難 | 簡單添加服務 |
| **環境一致性** | 中等 | 高 |

## 📝 常用指令

```bash
# 啟動所有服務
docker-compose up -d

# 停止所有服務
docker-compose down

# 重建並啟動
docker-compose up --build -d

# 查看服務狀態
docker-compose ps

# 查看特定服務日誌
docker-compose logs jenkins

# 進入 Jenkins 容器
docker-compose exec jenkins bash

# 清理所有資料（謹慎使用）
docker-compose down -v
```

## 🔍 疑難排解

### Jenkins 無法啟動
```bash
# 檢查端口是否被占用
netstat -tulpn | grep 8080

# 查看詳細錯誤日誌
docker-compose logs jenkins
```

### 權限問題
```bash
# 修復 Jenkins 資料目錄權限
sudo chown -R 1000:1000 jenkins_home/
```

## 🚀 下一步計劃

1. ✅ **當前階段**: Jenkins + Ant 基礎架構
2. 🔄 **下階段**: 加入程式碼品質檢查 (SonarQube)
3. 📦 **未來**: 整合套件管理 (Nexus)
4. ☁️ **長期**: Kubernetes 遷移

---

**注意**: 這個配置專注於你的核心需求，沒有添加過多複雜性。隨著需求成長，可以逐步添加其他服務。