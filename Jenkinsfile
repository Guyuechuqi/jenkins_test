pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "开发代码拉取完毕"
            }
        }

        stage('Prepare Tests') {
            steps {
                dir('autotest') {
                    // 这里的 credentialsId 记得替换成你实际可用的，或者去掉如果不需要
                    git branch: 'master', url: 'https://github.com/Guyuechuqi/jenkins_interface.git', credentialsId: 'git'
                    echo "测试代码拉取完毕"
                }
            }
        }

        stage('Run Test') {
            agent {
                docker {
                    image 'python:3.13'
                    // 关键点1：映射工作目录，保证容器内外路径一致（虽然Jenkins默认会做，但显式声明更稳妥）
                    args '-u root -v /var/jenkins_home:/var/jenkins_home' 
                }
            }
            steps {
                dir('autotest') {
                    script {
                        try {
                            sh '''
                                echo "当前执行目录: $(pwd)"
                                pip install -r requirements.txt
                                # 关键点2：确保 allure-results 目录存在
                                mkdir -p allure-results
                                # 运行测试，无论成功失败都继续
                                pytest -v -s testcases/ --alluredir=./allure-results || true
                            '''
                        } finally {
                            // 关键点3：【非常重要】修复权限！
                            // 因为容器里是 root，生成的文件 jenkins 没权限读，导致报告生成失败
                            sh 'chmod -R 777 ./allure-results'
                        }
                    }
                }
            }
        }
        
        // 关键点4：增加一个步骤，在 Docker 销毁后，检查宿主机上文件是否存在
        stage('Debug Files') {
            steps {
                dir('autotest') {
                    sh 'ls -la allure-results' // 如果这里报错，说明文件没带出来
                }
            }
        }
    }

    // 关键点5：在 post 中明确指定子目录
    post {
        always {
            script {
                echo "开始生成 Allure 报告..."
                // 这里的 path 必须匹配上面 dir('autotest') 里的相对路径
                allure includeProperties: false, 
                       jdk: '', 
                       results: [[path: 'autotest/allure-results']]
            }
        }
    }
}