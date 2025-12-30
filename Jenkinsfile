pipeline {
    agent any

    environment {
        // 定义测试仓库地址
        TEST_REPO_URL = 'https://github.com/Guyuechuqi/jenkins_interface.git'
    }

    stages {
        // 1. 拉取应用源码 (Pipeline script from SCM 默认会拉取，但我们可以显式打印信息)
        stage('Checkout App Source') {
            steps {
                echo 'Checking out Application Source Code...'
                checkout scm
                echo '拉取代码完成'
            }
        }

        // 2. 构建应用
        stage('Build Application') {
            steps {
                echo 'Building the application...'
                // 模拟构建，实际根据项目类型替换，如: sh 'mvn clean package' 或 sh 'npm run build'
                sh 'echo "Application Build Complete"'
                echo '构建完成'
            }
        }

        // 3. 拉取测试脚本 (关键步骤：多仓库拉取)
        stage('Checkout Test Scripts') {
            steps {
                // 在工作空间创建一个子目录存放测试脚本，避免文件冲突
                dir('automation_tests') {
                    git branch: 'master',
                        credentialsId: 'github_token', // 之前在Jenkins设置的凭据ID
                        url: "${TEST_REPO_URL}"
                }
            }
        }

        // 4. 执行测试
        stage('Execute Tests') {
            steps {
                dir('automation_tests') {
                    echo 'Installing dependencies and running tests...'
                    // 假设是 Python/Pytest 项目
                    sh '''
                        pip3 install -r requirements.txt
                        # 运行测试并生成 Junit XML 报告，以便Jenkins读取
                        python3 -m pytest --alluredir=allure-results
                    '''
                }
            }
        }
    }

    // 5. 构建后操作：生成报告与清理
    post {
        always {
            script {
                // 注意路径：因为我们在 'dir' 块之外执行 post，
                // 且之前的测试是在 'automation_tests' 目录下运行的，
                // 所以路径是 'automation_tests/allure-results'
                allure includeProperties: false, 
                       jdk: '', 
                       results: [[path: 'automation_tests/allure-results']]
            }
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}