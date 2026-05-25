pipeline {
    agent any

    // ======================== 环境变量 ========================
    environment {
        JAVA_HOME = '/Users/lvzichong/Library/Java/JavaVirtualMachines/ms-25.0.0/Contents/Home'
        PATH = "$JAVA_HOME/bin:/opt/homebrew/bin:$PATH"
    }

    // ======================== 构建参数 ========================
    parameters {
        booleanParam(
            name: 'SKIP_JAVADOC',
            defaultValue: true,
            description: '跳过 Javadoc 生成（可选阶段）'
        )
        booleanParam(
            name: 'SKIP_TESTS_IN_PACKAGE',
            defaultValue: true,
            description: 'Package 阶段跳过测试（测试已在 Test Stage 完成）'
        )
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
                script {
                    echo "当前分支: ${env.BRANCH_NAME}"
                    echo "当前提交: ${env.GIT_COMMIT.take(8)}"
                }
            }
        }

        stage('Clean') {
            steps {
                sh 'mvn clean'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test -pl docs-core,docs-web-common -Dtest=!com.sismics.util.TestResourceUtil'
            }
            post {
                always {
                    junit(
                        testResults: '**/target/surefire-reports/*.xml',
                        allowEmptyResults: true
                    )
                }
            }
        }

        stage('PMD') {
            steps {
                sh 'mvn pmd:pmd pmd:cpd'
            }
            post {
                always {
                    echo 'PMD 报告已生成至各模块 target/site/pmd.html'
                }
            }
        }

        stage('JaCoCo') {
            steps {
                sh 'mvn jacoco:report-aggregate'
            }
            post {
                always {
                    echo 'JaCoCo 聚合报告已生成至 target/site/jacoco-aggregate/'
                }
            }
        }

        stage('Javadoc') {
            when {
                expression { params.SKIP_JAVADOC == false }
            }
            steps {
                catchError(
                    buildResult: 'SUCCESS',
                    stageResult: 'UNSTABLE',
                    message: 'Javadoc 生成失败（可选阶段，不阻塞流水线）'
                ) {
                    sh 'mvn javadoc:javadoc'
                }
            }
        }

        stage('Site') {
            steps {
                sh 'mvn site'
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'target/site',
                        reportFiles: 'index.html',
                        reportName: 'Maven Site',
                        reportTitles: 'Teedy - Maven Site'
                    ])
                }
            }
        }

        stage('Package') {
            steps {
                script {
                    def skipTestsFlag = params.SKIP_TESTS_IN_PACKAGE ? '-DskipTests' : ''
                    sh "mvn package ${skipTestsFlag}"
                }
            }
        }
    }

    post {
        always {
            echo '=== 流水线完成，开始收集构建产物 ==='
            archiveArtifacts artifacts: 'docs-core/target/*.jar', allowEmptyArchive: true
            archiveArtifacts artifacts: 'docs-web-common/target/*.jar', allowEmptyArchive: true
            archiveArtifacts artifacts: 'docs-web/target/*.war', allowEmptyArchive: true
            archiveArtifacts artifacts: 'target/site/**/*', allowEmptyArchive: true
        }

        success {
            echo '=== 所有阶段成功完成 ==='
        }

        failure {
            echo '=== 流水线失败，请检查日志 ==='
        }
    }
}
