pipeline {
    agent any

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
        stage("多仓CI") {
            steps {
                script {
                    parallel repoJobs()
                }
            }
        }
    }
}


def repos() {
  return ["ComponentStarry", "driver_display",  "axtrap"]
}

def repoJobs() {
  jobs = [:]
  repos().each { repo ->
    jobs[repo] = { 
        stage(repo) {
           echo "Step for $repo"
        }

        stage("代码检出"){
            steps{
               echo "$repo 代码检出"
               sh"rm -rf  $mainRepoName; git clone $GITHUB_URL_PREFIX$mainRepoName$GITHUB_URL_SUFFIX; echo `pwd`;"
            }
        }
        stage("pytest嵌入"){
                    steps{
                            echo "$repo pytest嵌入"
                            sh 'echo $PATH'
                            sh 'printenv'
                            sh 'cp -r /home/jenkins_home/pytest $WORKSPACE/$mainRepoName'
                    }
                }
        stage("编译测试"){
                    steps {
                            echo "$repo 编译测试"
                            echo "--------------------------------------------test start------------------------------------------------"
                            // sh ' export pywork=$WORKSPACE/$mainRepoName && cd $pywork/pytest  && python3 -m pytest -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                            echo "--------------------------------------------test end  ------------------------------------------------"

                    }
                }

        stage("报告生成") {
            steps {
                script {
                    echo "$repo 报告生成"
                    // 输出 Allure 报告地址
                    echo "Allure Report URL: ${allureReportUrl}"
                }
            }
        }

        stage("结果展示"){
                    steps{
                        echo "$repo 结果展示"
                        echo "-------------------------allure report generating start---------------------------------------------------"
                        // sh 'export pywork=$WORKSPACE/$mainRepoName && cd $pywork/pytest && allure generate ./report/result -o ./report/html --clean'
                        // allure includeProperties: false, jdk: 'jdk17', report: "$mainRepoName/pytest/report/html", results: [[path: "$mainRepoName/pytest/report/result"]]
                        echo "-------------------------allure report generating end ----------------------------------------------------"
                    }
        }
    }
  }
  return jobs
}
