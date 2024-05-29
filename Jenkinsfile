pipeline {
    agent any

    parameters{
        string(name:'mainRepo',defaultValue:'https://github.com/henshing/Starry.git',description:'main repository')
        string(name:'relatedRepo1',defaultValue:'https://github.com/henshing/driver_display.git',description:'related repository')
        string(name:'relatedRepo2',defaultValue:'https://github.com/henshing/axtrap.git',description:'related repository')
        string(name:'email',defaultValue:'1445323887@qq.com',description:'Email address to send the report to')
    }

    environment {
        name = "StarryOS"
        JENKINS_URL = "http://49.51.192.19:9095"
        JOB_PATH = "job/github_test_yk"
        REPORT_PATH = "allure"
        //根据内置变量currentBuild获取构建号
        buildNumber = currentBuild.number
        // 构建 Allure 报告地址
        allureReportUrl = "${JENKINS_URL}/${JOB_PATH}/${buildNumber}/${REPORT_PATH}"
    }
    
    stages {
        stage('MainRepoTest'){
            steps{
                sh"git clone ${params.mainRepo}; cd `basename ${params.mainRepo}`; echo `pwd`; cd .."
            }
        }
        
        stage('RelatedRepoTest1'){
            steps{
                sh"git clone ${params.relatedRepo1}; cd `basename ${params.relatedRepo1}`; echo `pwd`;"
            }
        }
        
        stage('pytest嵌入'){
                    steps{
                            sh 'echo $PATH'
                            sh 'printenv'
                            sh 'cp -r /home/jenkins_home/pytest $WORKSPACE'
                    }
                }

        stage('编译测试'){
                    steps {
                            echo "--------------------------------------------test start------------------------------------------------"
                            sh ' export pywork=$WORKSPACE && cd $pywork/pytest  && python3 -m pytest -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                            echo "--------------------------------------------test end  ------------------------------------------------"
                        
                    }
                }
        
        stage('Report') {
            steps {
                script {
                    // 输出 Allure 报告地址
                    echo "Allure Report URL: ${allureReportUrl}"
                }
            }
        }

        stage('结果展示'){
                    steps{
                        echo "-------------------------allure report generating start---------------------------------------------------"
                        sh 'export pywork=$WORKSPACE && cd $pywork/pytest && allure generate ./report/result -o ./report/html --clean'
                        allure includeProperties: false, jdk: 'jdk21', report: 'pytest/report/html', results: [[path: 'pytest/report/result']]
                        echo "-------------------------allure report generating end ----------------------------------------------------"
                    }
                }
    }

    post {
        // always {
        //     junit'**/target/*.xml'
        // }
        failure{
            mail to:"${params.email}",subject:"the pipeline failed"，
            body:"${env.name}\n pipeline '${JOB_NAME}'(${BUILD_NUMBER}) (${allureReportUrl})"
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
