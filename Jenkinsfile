pipeline {
    agent any

    stages {
        stage('Checkout App') {
            steps {
                // 1. 默认拉取开发代码 (App Repo)
                checkout scm
                echo "开发代码拉取完毕"
            }
        }

        stage('Prepare Tests') {
            steps {
                // 2. 拉取测试代码 (Test Repo) 到子目录 'autotest'
                dir('autotest') {
                    git branch: 'main', url: 'https://github.com/Guyuechuqi/jenkins_interface.git', credentialsId: 'jenkins_interface'
                }
                echo "测试脚本拉取完毕"
            }
        }

        stage('Run Test') {
            agent {
                docker {
                    image 'python:3.13'
                    args '-u root'
                }
            }
            steps {
                // 3. 进入测试目录执行
                dir('autotest') {
                    sh '''
                        pip install -r requirements.txt
                        pytest -v -s testcases/ --alluredir=../allure-results
                    '''
                }
            }
        }
    }
}