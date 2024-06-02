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
        FROM_EMAIL="bityk@163.com"
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
        stage("合并展示"){
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
                mail subject:"PipeLine '${JOB_NAME}'(${BUILD_NUMBER}) 执行失败",
                // body:"${currentRepoName}仓CI报告链接：\nPipeline '${JOB_NAME}'(${BUILD_NUMBER}) (${allureReportUrl})"
                body: """
<div id="content">
<h1>仓库${currentRepoName} CI报告</h1>
<div id="sum2">
  <h2>构建结果</h2>
  <ul>
  <li>Job URL : <a href='${BUILD_URL}'>${BUILD_URL}</a></li>
  <li>执行结果 : <a>执行失败</a></li>
  <li>Job名称 : <a id="url_1">${JOB_NAME} [${BUILD_NUMBER}]</a></li>
  <li>项目名称 : <a>${JOB_NAME}</a></li>
  </ul>
</div>
<div id="sum0">
<h2>GIT 信息</h2>
<ul>
<li>GIT项目地址 : <a>${GIT_URL}</a></li>
<li>GIT项目当前分支名 : ${GIT_BRANCH}</li>
<li>GIT最后一次提交CommitID : ${GIT_COMMIT}</li>
</ul>
</div>
</div>
                """,
        charset: 'utf-8',
        from: "${FROM_EMAIL}",
        mimeType: 'text/html',
		to: "${REPORT_EMAIL}"
            }
        }
        success{
            script{
                mail subject:"PipeLine '${JOB_NAME}'(${BUILD_NUMBER}) 执行成功",
                // body:"${currentRepoName}仓CI报告链接：\nPipeline '${JOB_NAME}'(${BUILD_NUMBER}) (${allureReportUrl})"
                body: """
<div id="content">
<h1>仓库${currentRepoName} CI报告</h1>
<div id="sum2">
  <h2>构建结果</h2>
  <ul>
  <li>报告URL : <a href='${allureReportUrl}'>${allureReportUrl}</a></li>
  <li>Job URL : <a href='${BUILD_URL}'>${BUILD_URL}</a></li>
  <li>执行结果 : <a>执行成功</a></li>
  <li>Job名称 : <a id="url_1">${JOB_NAME} [${BUILD_NUMBER}]</a></li>
  <li>项目名称 : <a>${JOB_NAME}</a></li>
  </ul>
</div>
<div id="sum0">
<h2>GIT 信息</h2>
<ul>
<li>GIT项目地址 : <a>${GIT_URL}</a></li>
<li>GIT项目当前分支名 : ${GIT_BRANCH}</li>
<li>GIT最后一次提交CommitID : ${GIT_COMMIT}</li>
</ul>
</div>
</div>
                """,
        charset: 'utf-8',
        from: "${FROM_EMAIL}",
        mimeType: 'text/html',
		to: "${REPORT_EMAIL}"
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
        stage(repo + "编译测试"){
            withEnv(["repoName=$repo"]) { // it can override any env variable
                echo "repoName = ${repoName}"
                echo "$repo 编译测试"
                sh 'printenv'
                sh "cp -r /home/jenkins_home/pytest $WORKSPACE/$repo"
                echo "--------------------------------------------$repo test start------------------------------------------------"
                if (repoName == mainRepoName){
                    sh 'export pywork=$WORKSPACE/${repoName} repoName=${repoName} && cd $pywork/pytest && python3 -m pytest -m mainrepo --cmdrepo=${repoName} -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                } else {
                    sh 'export pywork=$WORKSPACE/${repoName} repoName=${repoName} && cd $pywork/pytest && python3 -m pytest -m childrepo --cmdrepo=${repoName} -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                }
                echo "--------------------------------------------$repo test end  ------------------------------------------------"
            }
        }
        stage(repo + "报告生成"){
            withEnv(["repoName=$repo"]) { // it can override any env variable
                echo "repoName = ${repoName}"
                echo "$repo 报告生成"
                // 输出 Allure 报告地址
                echo "$repo Allure Report URL: ${allureReportUrl}"
                echo "-------------------------$repo allure report generating start---------------------------------------------------"
                sh 'export pywork=$WORKSPACE/${repoName} && cd $pywork/pytest && cp -r ./report $WORKSPACE'
                echo "-------------------------$repo allure report generating end ----------------------------------------------------"
            }
        }
    }
  }
  return jobs
}
