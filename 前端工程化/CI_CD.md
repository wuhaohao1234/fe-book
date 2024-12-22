CI/CD 代表持续集成（Continuous Integration）和持续交付/持续部署（Continuous Delivery/Continuous Deployment），是现代软件开发中非常重要的实践和流程，通过这两者的无缝协作，开发团队可以更快速、更可靠地交付软件，极大地提高开发效率和产品质量。这些实践强调自动化，从代码提交到生产环境发布，涵盖了软件的整个生命周期

## 基础概念
### 持续集成（Continuous Integration，CI）
持续集成通过自动化构建和测试过程来实现多次代码集成和验证，从而尽早发现问题

#### 特点
1. 频繁集成：每次代码提交都会触发自动构建和测试，确保代码库始终保持可工作状态
2. 自动化构建：使用自动化工具进行编译、打包和构建，减少人为错误
3. 自动化测试：集成单元测试、集成测试和回归测试，确保每次代码更改不会破坏现有功能
4. 快速反馈：开发者可以在几分钟内得到自动化测试结果，从而快速修复可能存在的问题

#### 示例流程
1. 开发者将代码提交到版本控制系统（如 Git）
2. 触发 CI 工具进行自动化构建和测试
3. 构建成功后生成可部署的内容
4. 发送构建和测试结果的通知

### 持续交付（Continuous Delivery，CD）
持续交付是持续集成的扩展，目的是将构建后的内容自动化部署到生产环境的环中，确保每个版本都可随时发布

#### 特点
1. 可控发布：代码始终保持可发布状态，具备按需发布的能力。发布步骤仍需要手动批准，以确保合规性和稳定性
2. 自动化部署：自动化将构建的工件部署到不同的测试环境或预生产环境，减少部署过程中的人为干预
3. 环境一致性：确保在不同环境中部署的一致性，提前发现潜在问题，减少发布风险

#### 示例流程
1. 从 CI 工具生成的内容开始
2. 部署工件到测试环境，运行集成测试和验收测试
3. 部署到预生产环境，模拟生产环境的运行条件
4. 人工批准，将版本发布到生产环境

### 持续部署（Continuous Deployment，CD）
持续部署进一步扩展了持续交付，通过完全自动化的流程，将每次通过 CI 和 CD 流程的变更直接部署到生产环境。它完全消除手动发布步骤，提供了一种完全自动化的发布方式

#### 特点
1. 全自动发布：在持续交付基础上，移除人工批准步骤，实现代码更改自动部署到生产环境
2. 快速发布：每次通过测试的变更都会快速部署到生产环境，显著减少发布时间
3. 高效反馈：实时监控和快速响应生产环境中的问题，提高对用户反馈的响应速度

#### 示例流程
1. 从 CI 工具生成的工件开始
2. 部署工件到测试环境，运行自动化测试
3. 部署到预生产环境，运行自动化验收测试
4. 自动化审批并部署到生产环境

## CI/CD 工具
现代 CI/CD 工具通常包含以下功能：

+ 版本控制集成：与 Git、Subversion 等版本控制系统集成
+ 构建自动化：支持编译、测试、打包等构建任务的自动化执行
+ 测试自动化：集成单元测试、集成测试、验收测试等各类测试的执行和报告
+ 部署自动化：支持自动化部署工件到不同的环境（如开发、测试、生产环境）
+ 监控和报告：提供构建、测试和部署的日志和报告，帮助团队快速定位问题



有几个常见的 CI/CD 工具

+ GitHub Actions：GitHub 提供的 CI/CD 工具，直接集成在 GitHub 平台中
+ Jenkins：开源的自动化服务器，具有强大的插件生态系统，适用于各种 CI/CD 流程
+ Travis CI：专注于 GitHub 仓库的持续集成服务，支持多种编程语言
+ CircleCI：云端或本地安装的持续集成和交付平台，支持并行测试和工作流配置
+ GitLab CI：集成在 GitLab 环境中的 CI/CD 工具，适合结合 GitLab 仓库使用

## Github Actions
Github Actions 使用起来非常简单，创建 `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD # 当前工作流的名称

on: [push, pull_request] # 触发工作流的事件

jobs: # 定义具体工作
  build-and-test:
    runs-on: ubuntu-latest # 指定工作运行环境
    steps: # 定义每一步需要执行的操作
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'

    - name: Install Dependencies
      run: npm install

    - name: Run Tests
      run: npm test

  deploy:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Code
      uses: actions/checkout@v2

    - name: Deploy to Production
      env:
        API_KEY: ${{secrets.PRODUCTION_API_KEY}}
      run: echo "Deploying to production with API_KEY=${API_KEY}..."
      # 在这里添加自定义的部署脚本
```

在这个 CI/CD 配置中：

1. 触发条件：当有代码提交（push）或拉取请求（pull request）时，触发 CI/CD 流程
2. 构建和测试：检出代码、设置 Node.js 环境、安装依赖和运行测试。如果测试通过，才进行下一步的部署
3. 部署：如果代码被推送到 `main` 分支，则执行部署步骤

如果需要使用 API 密钥、SSH 密钥等敏感信息，可以在 GitHub 仓库的 "Settings" > "Secrets" 页面中添加这些敏感信息，然后在工作流中使用

提交代码后，GitHub Actions 会自动触发配置的工作流，可以在 GitHub 仓库的 "Actions" 标签页下查看工作流的执行情况

## Jenkins
[https://www.jenkins.io/zh/](https://www.jenkins.io/zh/)

相对于 Github Actions Jenkins 使用起来要复杂非常多，但功能也很强大，是业界最流行的 CI/CD 工具，关于起安装、基本操作可以参考 [Jenkins 官方文档](https://www.jenkins.io/zh/doc/book/installing/)  


在 Jenkins 中，有两种主要的项目类型：Freestyle 项目和 Pipeline 项目

+ Freestyle 项目是 Jenkins 中的一种基础项目类型，用于配置简单的构建、测试和部署任务。它通过图形用户界面（GUI）配置，适用于不需要复杂逻辑的项目
+ Pipeline 项目是 Jenkins 中灵活性最强的项目类型，允许使用代码定义整个构建、测试和部署的自动化流程。它支持更复杂的逻辑和多步骤的工作流，非常适合复杂项目和 DevOps 实践



大部分时候应用会选择 Pipeline 项目

```yaml
pipeline {
    agent any

    # 定义全局变量
    environment {
        NODE_VERSION = '14'
        DEPLOY_SERVER = "user@remote-server:/path/to/deploy"
    }
    # 定义项目的主要工作流程，包括检出代码、设置环境、安装依赖、测试、构建和部署
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Set up Node.js') {
            steps {
                sh '''
                # 安装 Node.js
                curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
                . ~/.nvm/nvm.sh
                nvm install $NODE_VERSION
                node -v
                npm -v
                '''
            }
        }
        stage('Install Dependencies') {
            steps {
                sh '''
                . ~/.nvm/nvm.sh
                nvm use $NODE_VERSION
                npm install
                '''
            }
        }
        stage('Test') {
            steps {
                sh '''
                . ~/.nvm/nvm.sh
                nvm use $NODE_VERSION
                npm test
                '''
            }
        }
        stage('Build') {
            steps {
                sh '''
                . ~/.nvm/nvm.sh
                nvm use $NODE_VERSION
                npm run build
                '''
            }
        }
        stage('Deploy') {
            steps {
                sshagent(['your-credentials-id']) {
                    sh '''
                    echo 'Deploying...'
                    scp -r ./dist $DEPLOY_SERVER
                    '''
                }
            }
        }
    }

    # 定义构建完成后的操作
    post {
        # 无论构建是否成功，总是执行这些动作
        always {
            archiveArtifacts artifacts: 'dist/**', onlyIfSuccessful: true
            mail to: 'team@example.com',
                 subject: "Jenkins Build [${currentBuild.fullDisplayName}]",
                 body: "Build details:\n\n${currentBuild.fullDisplayName}\nResult: ${currentBuild.currentResult}\n\nLogs: ${env.BUILD_URL}console"
        }
    }
}
```

