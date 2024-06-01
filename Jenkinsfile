pipeline {
    agent any

    environment {
        // 主仓名
        mainRepoName = "ComponentStarry"
        // 提交仓名
        currentRepoName = "${GIT_URL.substring(GIT_URL.lastIndexOf('/')+1, GIT_URL.length()-4)}"
        NODE_BASE_NAME = "ui-node-${GIT_COMMIT.substring(0, 6)}"
        JENKINS_URL = "http://49.51.192.19:9095"
        JOB_PATH = "job/github_test_yk"
        REPORT_PATH = "allure"
        GITHUB_URL_PREFIX = "https://github.com/kraigyang/"
        GITHUB_URL_SUFFIX = ".git"
        //根据内置变量currentBuild获取构建号
        buildNumber = "${currentBuild.number}"
        // 构建 Allure 报告地址
        allureReportUrl = "${JENKINS_URL}/${JOB_PATH}/${buildNumber}/${REPORT_PATH}"
        REPORT_EMAIL="528198540@qq.com"
    }
    
    stages {
        stage("多仓CI") {
            steps {
                script {
                    sh "rm -rf $WORKSPACE/report"
                    parallel repoJobs()
                }
            }
        }
        stage("报告合并"){
            steps {
                script {
                    echo "-------------------------allure report generating start---------------------------------------------------"
                    sh 'cd $WORKSPACE && allure generate ./report/result -o ./report/html --clean'
                    allure includeProperties: false, jdk: 'jdk17', report: "report/html", results: [[path: "report/result"]]
                    echo "-------------------------allure report generating end ----------------------------------------------------"
                }
            }
        } 
    }

    post {
        failure{
            script{
                mail to:"${REPORT_EMAIL}",
                subject:"PipeLine'${JOB_NAME}'(${BUILD_NUMBER})执行失败",
                body:"${currentRepoName}仓CI报告链接：\nPipeline '${JOB_NAME}'(${BUILD_NUMBER}) (${allureReportUrl})"
            }
        }
        success{
            script{
                mail to:"${REPORT_EMAIL}",
                subject:"PipeLine'${JOB_NAME}'(${BUILD_NUMBER})执行成功",
                body:"${currentRepoName}仓CI报告链接：\nPipeline '${JOB_NAME}'(${BUILD_NUMBER}) (${allureReportUrl})"
            }
        }
    }
}

def repos() {
  return ["$currentRepoName", "$mainRepoName"]
}

def repoJobs() {
  jobs = [:]
  repos().each { repo ->
    jobs[repo] = { 
        stage(repo + "代码检出"){
           echo "$repo 代码检出"
           sh "rm -rf  $repo; git clone $GITHUB_URL_PREFIX$repo$GITHUB_URL_SUFFIX; echo `pwd`;"
        }
        stage(repo + "自动化嵌入"){
            echo "$repo pytest嵌入"
            // sh 'echo $PATH'
            // sh 'printenv'
            sh """
                cp -r /home/jenkins_home/pytest $WORKSPACE/$repo;
                sed -i "s#\"仓库名\": .*#\"仓库名\": \"${currentRepoName}\",#" $WORKSPACE/$repo/pytest/config.py
               """
        }
        stage(repo + "编译测试"){
            withEnv(["repoName=$repo"]) { // it can override any env variable
                echo "repoName = ${repoName}"
                echo "$repo 编译测试"
                sh 'printenv'
                echo "--------------------------------------------$repo test start------------------------------------------------"
                if (repoName == mainRepoName){
                    sh 'export pywork=$WORKSPACE/${repoName} repoName=${repoName} && cd $pywork/pytest && python3 -m pytest -m mainrepo -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                } else {
                    sh 'export pywork=$WORKSPACE/${repoName} repoName=${repoName} && cd $pywork/pytest && python3 -m pytest -m childrepo -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                }
                echo "--------------------------------------------$repo test end  ------------------------------------------------"
            }
        }
        stage(repo + "报告生成") {
            echo "$repo 报告生成"
            // 输出 Allure 报告地址
            echo "$repo Allure Report URL: ${allureReportUrl}"
        }
        stage(repo + "结果展示"){
            withEnv(["repoName=$repo"]) { // it can override any env variable
                echo "repoName = ${repoName}"
                echo "$repo 结果展示"
                sh 'printenv'
                echo "-------------------------$repo allure report generating start---------------------------------------------------"
                sh 'export pywork=$WORKSPACE/${repoName} && cd $pywork/pytest && cp -r ./report $WORKSPACE'
                echo "-------------------------$repo allure report generating end ----------------------------------------------------"
            }
        }
    }
  }
  return jobs
}
