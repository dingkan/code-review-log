# 小傅哥项目： OpenAi 代码评审.
### 😀代码评分：90
#### 😀代码逻辑与目的：
该代码片段包含两个GitHub Actions工作流程文件，用于构建和运行OpenAi代码评审，以及发布JAR包。`main-remote-jar.yml`用于在push到`master-close`分支或pull request时构建和运行代码评审。`main-upload-jar.yml`用于在推送带有`v`前缀的tag时构建、发布和上传JAR包。

#### 🤔问题点：
1. **环境变量使用**：在`main-remote-jar.yml`中，环境变量`GITHUB_REVIEW_LOG_URI`和`CODE_TOKEN`被错误地设置为`CODE_REVIEW_LOG_URI`和`CODE_TOKEN`。这可能导致配置错误。
2. **资源下载**：在`main-remote-jar.yml`中，使用`wget`下载JAR包没有错误处理，如果下载失败，工作流程将失败。
3. **依赖管理**：在`openAI-code-review-sdk/dependency-reduced-pom.xml`中，所有依赖项都被标记为`provided`，这意味着它们不会包含在最终构建的JAR包中。如果主应用程序需要这些依赖项，则需要重新考虑这一点。

#### 🎯修改建议：
1. 修正环境变量名称。
2. 在下载资源时添加错误处理。
3. 根据需要调整依赖项的`scope`。

#### 💻修改后的代码：
```yaml
# main-remote-jar.yml
name: Build and Run OpenAiCodeReview By Main Remote Jar

on:
  push:
    branches:
      - master-close
  pull_request:
    branches:
      - master-close

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
        with:
          fetch-depth: 2

      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          distribution: 'adopt'
          java-version: '11'

      - name: Create libs directory
        run: mkdir -p ./libs

      - name: Download openai-code-review-sdk JAR
        run: |
          wget -O ./libs/openai-code-review-sdk-1.0.jar https://github.com/fuzhengwei/openai-code-review-log/releases/download/v1.0/openai-code-review-sdk-1.0.jar
          if [ $? -ne 0 ]; then
            echo "Failed to download JAR file."
            exit 1
          fi

      - name: Get repository name
        id: repo-name
        run: echo "REPO_NAME=${GITHUB_REPOSITORY##*/}" >> $GITHUB_ENV

      - name: Get branch name
        id: branch-name
        run: echo "BRANCH_NAME=${GITHUB_REF#refs/heads/}" >> $GITHUB_ENV

      - name: Get commit author
        id: commit-author
        run: echo "COMMIT_AUTHOR=$(git log -1 --pretty=format:'%an <%ae>')" >> $GITHUB_ENV

      - name: Get commit message
        id: commit-message
        run: echo "COMMIT_MESSAGE=$(git log -1 --pretty=format:'%s')" >> $GITHUB_ENV

      - name: Print repository, branch name, commit author, and commit message
        run: |
          echo "Repository name is ${{ env.REPO_NAME }}"
          echo "Branch name is ${{ env.BRANCH_NAME }}"
          echo "Commit author is ${{ env.COMMIT_AUTHOR }}"
          echo "Commit message is ${{ env.COMMIT_MESSAGE }}"

      - name: Run Code Review
        run: java -jar ./libs/openai-code-review-sdk-1.0.jar
        env:
          GITHUB_REVIEW_LOG_URI: ${{ secrets.CODE_REVIEW_LOG_URI }}
          GITHUB_TOKEN: ${{ secrets.CODE_TOKEN }}
          COMMIT_PROJECT: ${{ env.REPO_NAME }}
          COMMIT_BRANCH: ${{ env.BRANCH_NAME }}
          COMMIT_AUTHOR: ${{ env.COMMIT_AUTHOR }}
          COMMIT_MESSAGE: ${{ env.COMMIT_MESSAGE }}
          # ... other environment variables ...

# main-upload-jar.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v3
        with:
          distribution: 'adopt'
          java-version: '11'

      - name: Build with Maven
        run: mvn -B clean package

      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          draft: false
          prerelease: false

      - name: Debug Output
        run:
          echo "Upload URL: ${{ steps.create_release.outputs.upload_url }}"

      - name: Upload Release Asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: target/*.jar
          asset_name: my-app.jar
          asset_content_type: application/java-archive
```

#### 🎯修改后的pom.xml（openAI-code-review-sdk/dependency-reduced-pom.xml）：
```xml
<dependencies>
  <!-- ... other dependencies ... -->
  <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.6</version>
    <scope>runtime</scope>
  </dependency>
  <!-- ... other dependencies with scope 'runtime' ... -->
</dependencies>
```

#### 🎯代码优点：
- 工作流程设计清晰，易于理解。
- 使用了GitHub Actions进行自动化构建和发布。
- 环境变量管理良好。

#### 🎯代码的逻辑和目的：
该代码用于自动化构建、运行代码评审和发布JAR包。它通过GitHub Actions在特定事件（如push或tag推送）触发工作流程，并执行相应的任务。