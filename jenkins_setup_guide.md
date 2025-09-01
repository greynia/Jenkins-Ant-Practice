# Jenkins GUI 執行指南

## 🚀 在 Jenkins GUI 中執行 Jenkinsfile

### 1. 啟動 Jenkins Docker
```bash
# 啟動 Jenkins 容器
# docker run -d \
#   --name jenkins-practice \
#   -p 8080:8080 \
#   -p 50000:50000 \
#   -v jenkins_home:/var/jenkins_home \
#   -v /Users/greynia/Desktop/Job/Andes_interview:/workspace \
#   jenkins/jenkins:lts

docker run -d --name jenkins-practice -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts

# 獲取初始密碼
docker logs jenkins-practice

docker exec jenkins-practice cat /var/jenkins_home/secrets/initialAdminPassword

winpty docker exec -it jenkins-practice cat //var/jenkins_home/secrets/initialAdminPassword
```

### 2. 訪問 Jenkins
- 打開瀏覽器，訪問：`http://localhost:8080`
- 輸入初始密碼進行設定

### 3. 安裝必要 Plugins
```
Dashboard > Manage Jenkins > Manage Plugins > Available

搜尋並安裝：
✅ Ant Plugin
✅ JUnit Plugin  
✅ HTML Publisher Plugin
✅ Pipeline Plugin
✅ Email Extension Plugin
✅ Build Timeout Plugin
✅ Workspace Cleanup Plugin

點擊 "Install without restart"
```

### 4. 設定全域工具
```
Dashboard > Manage Jenkins > Global Tool Configuration

🔧 JDK:
- Name: Default JDK
- 勾選 "Install automatically"
- 選擇版本：Java 21

🔧 Ant:  
- Name: Default Ant
- 勾選 "Install automatically"
- 選擇版本：Ant 1.10.x
```

### 5. 創建新的 Pipeline Job

#### 🎯 方法一：直接複製 Pipeline 腳本
```
1. Dashboard > New Item
2. 輸入名稱：Practice01-Build  
3. 選擇：Pipeline
4. 點擊 OK

在 Pipeline 設定頁面：
5. Definition: Pipeline script
6. 複製你的 Jenkinsfile 內容到 Script 欄位
7. 點擊 Save
```

#### 🎯 方法二：從檔案系統讀取（推薦）
```
1. Dashboard > New Item
2. 輸入名稱：Practice01-Build
3. 選擇：Pipeline  
4. 點擊 OK

在 Pipeline 設定頁面：
5. Definition: Pipeline script from SCM
6. SCM: None (Filesystem)
7. Script Path: /workspace/interview/practice1/advanced/Jenkinsfile
8. 點擊 Save
```

### 6. 執行建置

#### 🏗️ 手動執行
```
1. 進入你的 Job：Practice01-Build
2. 點擊左側 "Build Now" 按鈕
3. 建置會立即開始
```

#### 📊 監控執行過程
```
1. 點擊建置編號 (如 #1, #2)
2. 點擊 "Console Output" 查看即時日誌
3. 點擊 "Pipeline Steps" 查看各階段狀態
4. 查看 "Build Artifacts" 下載產出檔案
```

### 7. 檢查執行結果

#### ✅ 成功狀態檢查
```
建置完成後會看到：
- 🟢 Build Status: SUCCESS
- 📦 Artifacts: JAR, WAR 檔案
- 📋 Test Results: JUnit 報告
- 📚 HTML Reports: API 文檔、建置報告
```

#### ❌ 失敗排除
```
常見問題：
1. Java/Ant 未正確安裝
   解決：檢查 Global Tool Configuration

2. build.xml 找不到
   解決：檢查檔案路徑是否正確

3. 權限問題
   解決：確保 Jenkins 有讀取檔案權限

4. 網路問題
   解決：檢查依賴下載是否正常
```

## 🔧 執行流程圖

```
用戶點擊 "Build Now"
        ↓
Jenkins 讀取 Jenkinsfile
        ↓  
執行各個 stage 順序：
        ↓
🧹 Workspace Cleanup
        ↓
📋 Environment Info  
        ↓
📁 Copy Build Files
        ↓
🏗️ Initialize Project
        ↓
📦 Download Dependencies
        ↓
🔨 Generate & Compile
        ↓
🧪 Run Tests
        ↓
📦 Package (JAR/WAR 並行)
        ↓
📚 Generate Documentation
        ↓
📋 Generate Reports
        ↓
✅ 完成並顯示結果
```

## 🎯 重要提醒

### 檔案位置確認
```
確保這些檔案在正確位置：
✅ /workspace/interview/practice1/advanced/Jenkinsfile
✅ /workspace/interview/practice1/advanced/build.xml
```

### Docker 掛載確認
```bash
# 確保 Docker 掛載正確
docker exec jenkins-practice ls -la /workspace/interview/practice1/advanced/

應該看到：
- Jenkinsfile
- build.xml  
- README.md
```

### 手動測試
```bash
# 進入容器手動測試
docker exec -it jenkins-practice bash
cd /workspace/interview/practice1/advanced/
ant -projecthelp  # 查看可用 targets
```

這樣設定完後，點擊 "Build Now" 就能執行你的完整 CI/CD 流程了！



必要插件：
  - Docker Pipeline - 用於 Docker 相關的 CI/CD
  - Kubernetes - 如果有容器編排需求
  - SonarQube Scanner - 代碼品質檢查
  - Checkmarx - 安全掃描

  實用插件：
  - Blue Ocean - 現代化的 Pipeline 視覺化界面
  - Slack Notification - 通知整合
  - Role-based Authorization Strategy - 權限管理
  - Generic Webhook Trigger - 彈性的 webhook 觸發

  DevOps 相關：
  - Ansible - 自動化部署
  - Terraform - 基礎設施即代碼
  - AWS Steps - 如果使用 AWS