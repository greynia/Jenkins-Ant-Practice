1. 必要的 Jenkins Plugins

  必裝 Plugins

  # 在 Jenkins > Manage Jenkins > Manage Plugins > Available 搜尋並安裝:
  - Ant Plugin
  - JUnit Plugin
  - HTML Publisher Plugin
  - Pipeline Plugin
  - Git Plugin (如果要用 Git)
  - Email Extension Plugin
  - Build Timeout Plugin
  - Workspace Cleanup Plugin

  Plugin 安裝指令 (Docker 啟動時)

  # 如果你用 Docker，可以預裝 plugins
  FROM jenkins/jenkins:lts
  COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
  RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt

  # plugins.txt 內容
  ant:475.vf34069fef73c
  junit:1189.v18a_8def8d9fb_
  htmlpublisher:1.36
  workflow-aggregator:596.v8c21c963d92d
  git:5.2.1
  email-ext:2.108
  build-timeout:1.32
  ws-cleanup:0.45

  2. 完整的 Jenkinsfile

⏺ Write(production_jenkinsfile.groovy)
  ⎿  Wrote 361 lines to production_jenkinsfile.groovy      
     pipeline {
         agent any
         options {
             // 建置超時設定
             timeout(time: 30, unit: 'MINUTES')
             // 保留建置記錄
             buildDiscarder(logRotator(numToKeepStr: '10'))
             // 避免並行執行
             disableConcurrentBuilds()
     … +351 lines (ctrl+r to expand)

⏺ 3. Jenkins 設定步驟

  🔧 Global Tool Configuration

  Jenkins > Manage Jenkins > Global Tool Configuration

  JDK installations:
  - Name: Default JDK
  - JAVA_HOME: /usr/lib/jvm/java-21-openjdk (Docker 內路徑)

  Ant installations:  
  - Name: Default Ant
  - ANT_HOME: /usr/share/ant (Docker 內路徑)

  🐳 Docker Jenkins 啟動指令

  # 啟動 Jenkins with 必要工具
  docker run -d \
    --name jenkins-practice \
    -p 8080:8080 \
    -p 50000:50000 \
    -v jenkins_home:/var/jenkins_home \
    -v /Users/greynia/Desktop/Job/Andes_interview:/workspace \
    jenkins/jenkins:lts

  # 進入容器安裝 Ant
  docker exec -u root jenkins-practice apt-get update
  docker exec -u root jenkins-practice apt-get install -y ant

  📝 創建 Jenkins Job

  1. Jenkins > New Item
  2. 輸入名稱: Practice01-Build
  3. 選擇: Pipeline
  4. Pipeline Definition: Pipeline script
  5. 複製貼上上面的 Jenkinsfile 內容
  6. Save

  4. 測試執行
------
1. 啟動 Jenkins
2. 安裝必要 plugins
3. 設定 Global Tools
4. 創建 Pipeline Job
5. 點擊 "Build Now"
6. 檢查 Console Output

Jenkins 是在容器內部建立臨時工作空間，不是在你的本地專案目錄執行。

  🏗️ 執行位置說明：

  實際執行位置：
```
  Jenkins 容器內部：
  
  /var/jenkins_home/workspace/Practice01-Build/
  ├── build.xml          ← 複製過來的
  ├── src/               ← generate-sources 建立的
  ├── build/             ← 編譯輸出
  ├── dist/              ← JAR/WAR 輸出
  └── logs/              ← 建置日誌

  你的本地專案：

  /Users/greynia/Desktop/Job/Andes_interview/interview/practice1/advanced/
  ├── build.xml          ← 原始檔案（不變）
  ├── Jenkinsfile        ← 原始檔案（不變）
  └── README.md          ← 原始檔案（不變）

  📁 檔案流向：

  本地檔案 → 複製到容器 → 容器內執行 → 產出檔案在容器內
```
  Jenkinsfile 中的這行：

  sh """
      cp '/Users/greynia/Desktop/Job/Andes_interview/interview/practice1/advanced/build.xml' .
  """

  這是把你的 build.xml 複製到 Jenkins 工作空間，然後在容器內執行建置。

  🔍 驗證方式：

  查看容器內的檔案：

  # 建置期間或完成後
  docker exec -it jenkins-practice bash
  cd /var/jenkins_home/workspace/Practice01-Build
  ls -la

  # 會看到建置產出：
  dist/Practice01-1.0.0.jar
  dist/Practice01-1.0.0.war
  logs/build.log

  查看本地專案：

  # 你的本地專案目錄不會有變化
  ls -la /Users/greynia/Desktop/Job/Andes_interview/interview/practice1/advanced/

  # 只會看到原始檔案：
  build.xml
  Jenkinsfile
  README.md

  💾 如果要取得建置產物：

  // 在 Jenkinsfile 中加入
  post {
      always {
          // 保存到 Jenkins Archives（可下載）
          archiveArtifacts artifacts: 'dist/*.jar,dist/*.war'

          // 或複製回本地（需要 volume 掛載）
          sh 'cp dist/*.jar /workspace/output/'
      }
  }

  總結：執行是在容器的隔離環境中，產出檔案也在容器內，需要透過 Jenkins GUI 下載或 volume 掛載來取得。
