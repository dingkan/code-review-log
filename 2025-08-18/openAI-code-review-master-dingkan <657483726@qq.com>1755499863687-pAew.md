# 小傅哥项目： OpenAi 代码评审.
### 😀代码评分：85
#### 😀代码逻辑与目的：
该代码片段是一个GitHub Actions工作流配置文件，用于在合并PR到master分支时构建和部署项目。工作流包括两个任务：`build`和`deploy`。`build`任务负责检查代码和构建项目，而`deploy`任务则依赖于`build`任务的结果，用于部署构建的jar包。

#### 🤔问题点：
1. `deploy`任务中缺少对`build`任务的明确依赖声明。
2. `deploy`任务中下载的jar包路径设置不正确。
3. `deploy`任务中的步骤不够详细，缺乏必要的配置。
4. 代码注释缺失，不利于其他开发者理解。

#### 🎯修改建议：
1. 明确`deploy`任务依赖`build`任务。
2. 修正下载的jar包路径。
3. 添加必要的步骤，如设置JDK版本、下载SDK包等。
4. 添加必要的注释，提高代码可读性。

#### 💻修改后的代码：
```yaml
diff --git a/.github/workflows/main-remote-jar.yml b/.github/workflows/main-remote-jar.yml
index 4dd78a9..d099d2f 100644
--- a/.github/workflows/main-remote-jar.yml
+++ b/.github/workflows/main-remote-jar.yml
@@ -7,11 +7,12 @@ on:
   pull_request:
     branches:
       - master
+      -
 
 jobs:
-  build:
+  build:
     runs-on: ubuntu-latest
-
     steps:
       - name: Checkout repository
         uses: actions/checkout@v2
         with:
@@ -27,12 +28,36 @@ jobs:
       - name: Create libs directory
         run: mkdir -p ./libs
 
-
-      # 下载sdk包
+      - name: Download Artifact
         uses: actions/download-artifact@v3
         with:
-          name: openAI-code-review-sdk-1.0-SNAPSHOT   # 必须与下载时的名称一致
-          path: target/*.jar        # 上传的文件路径
+          name: openAI-code-review-sdk-1.0-SNAPSHOT  # 必须与上传时的名称一致
+          path: ./libs
+
+      - name: List files
+        run: ls -R ./libs
+
+
+
+  deploy:
+    needs: build
+    runs-on: ubuntu-latest
+
+    steps:
+      - name: Checkout repository
+        uses: actions/checkout@v2
+        with:
+          fetch-depth: 2
+
+      - name: Set up JDK 11
+        uses: actions/setup-java@v2
+        with:
+          distribution: 'adopt'
+          java-version: '11'
+
+      # 下载sdk包
 #      - name: Download openai-code-review-sdk JAR
 #        env:
 #          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```