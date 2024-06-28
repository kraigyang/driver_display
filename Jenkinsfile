pipeline {
    agent any
    environment {
        // 主仓名
        mainRepoName = "ComponentStarry"
        // 提交仓名
        // currentRepoName = "${GIT_URL.substring(GIT_URL.lastIndexOf('/')+1, GIT_URL.length()-4)}"
	currentRepoName = "${GIT_URL}"
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
        // 将GA_TOKEN(GA = Github Access)替换为在 Jenkins 中存储的 GitHub 访问令牌的凭据 ID
        GA_TOKEN = credentials("GithubAccessToken")
        GA_REPO_OWNER = 'kraigyang'
        GA_REPO_NAME = "${GIT_URL.substring(GIT_URL.lastIndexOf('/')+1, GIT_URL.length()-4)}"
        // 动态获取当前构建的提交 SHA
        GA_COMMIT_SHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
    }
    
    stages {
        stage("多仓CI") {
            steps {
                script {
		        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
	                    sh "rm -rf $WORKSPACE/report"
	                    parallel repoJobs()
			}
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
                updateGithubCommitStatus('failure', "Build failed")
            }
        }
        success{
            script{
                updateGithubCommitStatus('success', "Build succeeded")
            }
        }
	unstable{
            script{
                updateGithubCommitStatus('failure', "Build succeeded")
            }
        }
	always{
            script{
		sendResultMail()
            }
        }
    }
}

def sendResultMail(){
        mail subject:"PipeLine '${JOB_NAME}'(${BUILD_NUMBER}) Build ${currentBuild.currentResult}",
        body: """
<div id="content">
<h1>仓库${currentRepoName} CI报告</h1>
<div id="sum2">
  <h2>构建结果</h2>
  <ul>
  <li>报告URL : <a href='${allureReportUrl}'>${allureReportUrl}</a></li>
  <li>Job URL : <a href='${BUILD_URL}'>${BUILD_URL}</a></li>
  <li>执行结果 : <a>Build ${currentBuild.currentResult}</a></li>
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


import groovy.json.JsonSlurper
def getSecrets(jsonFilePath, var) {
    def fileContent = readFile "${jsonFilePath}"
    Map jsonContent = (Map) new JsonSlurper().parseText(fileContent)
    var = jsonContent.get(var)
    echo "var : ${var}"
    return var
}


def updateGithubCommitStatus(String state, String description) {
    def context = 'continuous-integration/jenkins'
    def target_url = "${env.JOB_URL}/${env.BUILD_NUMBER}"

    sh """
    curl -s -X POST -H "Authorization: token ${GA_TOKEN}" \
    -d '{\"state\": \"${state}\", \"target_url\": \"${target_url}\", \"description\": \"${description}\", \"context\": \"${context}\"}' \
    https://api.github.com/repos/${GA_REPO_OWNER}/${GA_REPO_NAME}/statuses/${GA_COMMIT_SHA}
    """
}

def repos() {
  return ["$currentRepoName"] +  getSecrets("./README.json", "test_repos")
}

def repoJobs() {
  jobs = [:]
  repos().each { repo ->
    jobs[repo] = {
	repoFolder = "${repo.substring(repo.lastIndexOf('/')+1, repo.length()-4)}"
        stage(repoFolder + "代码检出"){
           echo "$repoFolder 代码检出"
           sh "rm -rf  $repoFolder; git clone $repo; echo `pwd`;"
        }
        stage(repoFolder + "编译测试"){
                script {
		        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                           withEnv(["repoName=$repoFolder"]) { // it can override any env variable
                                echo "repoName = ${repoName}"
                                echo "$repoFolder 编译测试"
                                sh 'printenv'
                                sh "cp -r /home/jenkins_home/pytest $WORKSPACE/$repoFolder"
                                echo "--------------------------------------------$repoFolder test start------------------------------------------------"
                                if (repoName == mainRepoName){
                                  sh 'export pywork=$WORKSPACE/${repoName} repoName=${repoName} && cd $pywork/pytest && python3 -m pytest -m mainrepo --cmdrepo=${repoName} -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                                } else {
                                  sh 'export pywork=$WORKSPACE/${repoName} repoName=${repoName} && cd $pywork/pytest && python3 -m pytest -m childrepo --cmdrepo=${repoName} -sv --alluredir report/result testcase/test_arceos.py --clean-alluredir'
                                }
                                echo "--------------------------------------------$repoFolder test end  ------------------------------------------------"
                          }
		       }
               }
        }
        stage(repoFolder + "报告生成"){
            withEnv(["repoName=$repoFolder"]) { // it can override any env variable
                echo "repoName = ${repoName}"
                echo "$repoFolder 报告生成"
                // 输出 Allure 报告地址
                echo "$repoFolder Allure Report URL: ${allureReportUrl}"
                echo "-------------------------$repoFolder allure report generating start---------------------------------------------------"
                sh 'export pywork=$WORKSPACE/${repoName} && cd $pywork/pytest && cp -r ./report $WORKSPACE'
                echo "-------------------------$repoFolder allure report generating end ----------------------------------------------------"
            }
        }
    }
  }
  return jobs
}
