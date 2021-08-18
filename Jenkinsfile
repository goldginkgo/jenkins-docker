@Library("jenkins-library")

def tools = new org.devops.v1.tools()
def sendEmail = new org.devops.v1.sendEmail()
def build = new org.devops.v1.build()
def semVer = new org.devops.v1.semVer()

pipeline {
    agent { label "maven38" }

    options {
        // timestamps()    // 日志会有时间
        skipDefaultCheckout()   // 删除隐式checkout scm语句
        disableConcurrentBuilds()   //禁止并行
        timeout(time:10, unit:'MINUTES') //设置流水线超时时间
    }

    environment {
        GIT_BRANCH = "master"
        GIT_URL = "https://github.com/goldginkgo/jenkins-docker.git"
        IMAGE = "goldginkgo/jenkins"
        TO_EMAIL_USER = "yongjindai09@gmail.com"
    }

    stages {
        stage('GetCode') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "${GIT_BRANCH}"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    submoduleCfg: [],
                    userRemoteConfigs: [[credentialsId: 'gitlab_username_pass', url: "${GIT_URL}"]]])

                script{
                    env.IMAGE_TAG = semVer.getNextMinorVersion()
                    sh """
                        echo ${env.IMAGE_TAG}
                    """
                }
            }
        }

        stage('BuildImage') {
            steps {
                container("kaniko") {
                    sh "/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --destination=${IMAGE}:${IMAGE_TAG} --cache=true --cache-dir=/kanikocache --skip-tls-verify"
                }
            }
        }

        stage('TagGitRepo') {
            steps {
                script{
                    sh """
                        git remote set-url origin ${GIT_URL}
                        git config --global user.email "yongjindai09@gmail.com"
                        git config --global user.name "Frank Dai"
                        git status
                        git tag -a ${IMAGE_TAG} -m "version ${IMAGE_TAG}"
                        git push origin ${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            script{
                tools.PrintMes("success:只有构建成功才会执行","green")
                currentBuild.description += "\n构建成功!"
                sendEmail.SendEmail("构建成功",TO_EMAIL_USER)
            }
        }
        failure {
            script{
                tools.PrintMes("failure:只有构建失败才会执行","red")
                currentBuild.description += "\n构建失败!"
                sendEmail.SendEmail("构建失败",TO_EMAIL_USER)
            }
        }
        aborted {
            script{
                tools.PrintMes("aborted:只有取消构建才会执行","red")
                currentBuild.description += "\n构建取消!"
                sendEmail.SendEmail("取消构建",TO_EMAIL_USER)
            }
        }
    }
}