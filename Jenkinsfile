pipeline {
    agent any

    parameters{
        string(name:'mainRepo',defaultValue:'https://github.com/kraigyang/ComponentStarry.git',description:'main repository')
        string(name:'relatedRepo1',defaultValue:'https://github.com/kraigyang/driver_display.git',description:'related repository')
        string(name:'relatedRepo2',defaultValue:'https://github.com/kraigyang/axtrap.git',description:'related repository')

        string(name:'mainRepoFolder',defaultValue:'ComponentStarry',description:'main repository')
        string(name:'relatedRepo1Folder',defaultValue:'driver_display',description:'related repository')
        string(name:'relatedRepo2Folder',defaultValue:'axtrap',description:'related repository')

        string(name:'email',defaultValue:'528198540@qq.com',description:'Email address to send the report to')
    }
    
    environment {
        name = "ComponentStarry"
        mainRepoName = "ComponentStarry"
        JENKINS_URL = "http://49.51.192.19:9095"
        JOB_PATH = "job/github_test_yk"
        REPORT_PATH = "allure"
        GITHUB_URL_PREFIX = "https://github.com/kraigyang/"
        GITHUB_URL_SUFFIX = ".git"
        //根据内置变量currentBuild获取构建号
        buildNumber = "${currentBuild.number}"
        // 构建 Allure 报告地址
        allureReportUrl = "${JENKINS_URL}/${JOB_PATH}/${buildNumber}/${REPORT_PATH}"
    }

    stages {
        stage("多仓CI"){
            steps {
                script {
                    def relatedRepos = ["ComponentStarry", "driver_display",  "axtrap"]
                    for (repo in relatedRepos) {
                        stage("$repo仓: 代码检出"){
                            steps{
                                sh"rm -rf  $repo; git clone $GITHUB_URL_PREFIX$repo$GITHUB_URL_SUFFIX; echo `pwd`;"
                            }
                        }
                        // stage('MainRepoTest'){
                        //     steps{
                        //        sh"rm -rf  ${params.mainRepoFolder}; git clone ${params.mainRepo}; cd ${params.mainRepoFolder}; echo `pwd`;"
                        //     }
                        // }
                
                        stage("$repo仓: pytest嵌入"){
                                    steps{
                                            sh 'echo $PATH'
                                            sh 'printenv'
                                            sh 'cp -r /home/jenkins_home/pytest $WORKSPACE/$repo'
                                    }
                                }
                        stage("$repo仓: 编译测试"){
                                    steps {
                                            echo "--------------------------------------------test start------------------------------------------------"
                                            // sh ' export pywork=$WORKSPACE/$mainRepoName && cd $pywork/pytest  && python3 -m pytest -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                                            echo "--------------------------------------------test end  ------------------------------------------------"
                
                                    }
                                }
                
                        stage("$repo仓: 报告生成") {
                            steps {
                                script {
                                    // 输出 Allure 报告地址
                                    echo "Allure Report URL: ${allureReportUrl}"
                                }
                            }
                        }
                
                        stage('$repo仓: 结果展示'){
                                    steps{
                                        echo "-------------------------allure report generating start---------------------------------------------------"
                                        // sh 'export pywork=$WORKSPACE/$mainRepoName && cd $pywork/pytest && allure generate ./report/result -o ./report/html --clean'
                                        // allure includeProperties: false, jdk: 'jdk17', report: "$mainRepoName/pytest/report/html", results: [[path: "$mainRepoName/pytest/report/result"]]
                                        echo "-------------------------allure report generating end ----------------------------------------------------"
                                    }
                                }
                    }
                } // Script
            } // Steps
        } // Stage
    } // Stages

    post {
        // always {
        //     junit'**/target/*.xml'
        // }
        failure{
            script{
                mail to:"${params.email}",
                subject:"the pipeline failed",
                body:"${env.name}\n pipeline '${JOB_NAME}'(${BUILD_NUMBER}) (${allureReportUrl})"
            }
        }
        success{
            script{
                mail to:"${params.email}",
                subject:"PipeLine'${JOB_NAME}'(${BUILD_NUMBER})result",
                body:"${env.name}\n pipeline '${JOB_NAME}'(${BUILD_NUMBER}) (${allureReportUrl})"
            }
        }
    }
}
