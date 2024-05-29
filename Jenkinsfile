pipeline {
    agent any
    stages {
        // stage('检出代码') {
        //   steps {
        //         script{
        //                 // 打印所有环境变量
        //                 sh 'echo $PATH'
        //                 sh 'printenv'
        //                 // updateGitlabCommitStatus name: 'build', state: 'pending'
        //                 // 判断当前分支名
        //                 // if(env.gitlabSourceBranch !=null && !env.gitlabSourceBranch.contains('tags')){
        //                 //     checkout scmGit(branches: [[name: "origin/${env.gitlabSourceBranch}"]], extensions: [], userRemoteConfigs: [[credentialsId: '37b752c1-0023-42b9-b159-13e18fe85fea', url: 'git@10.14.93.219:cicv/kernel.git']])
        //                 //     echo "Current branch now is: env.gitlabSourceBranch=${env.gitlabSourceBranch}"
                            
        //                 // }else{
        //                 //     checkout scmGit(branches: [[name: "${params.GIT_BRANCH}"]], extensions: [], userRemoteConfigs: [[credentialsId: '37b752c1-0023-42b9-b159-13e18fe85fea', url: 'git@10.14.93.219:cicv/kernel.git']])
        //                 //     echo "Current branch now is: params.GIT_BRANCH=${params.GIT_BRANCH}"
        //                 // }
        //             }
        //     }
        // }
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
                stage('结果展示'){
                    steps{
                        echo "-------------------------allure report generating start---------------------------------------------------"
                        sh 'export pywork=$WORKSPACE && cd $pywork/pytest && allure generate ./report/result -o ./report/html --clean'
                        allure includeProperties: false, jdk: 'jdk21', report: 'pytest/report/html', results: [[path: 'pytest/report/result']]
                        echo "-------------------------allure report generating end ----------------------------------------------------"
                    }
                }
        //     stage('远程部署') {
        //     steps {
        //         echo '======开始自动部署======'
        //         sh 'cd /mnt/cicv && ./Commit_154A_changekernel.sh build-rust'
        //         echo '******自动部署完成******'
        //     }
        // }
        
        //     stage('自动化测试') {
        //     steps {
        //         echo '======开始自动化测试======'
        //         sh 'cd /mnt/cicv/jenkins/workspace/CommitKernelCI/KernelTA && pytest -sv --alluredir report/result testcase/test_driver_function.py --clean-alluredir && allure generate ./report/result -o ./report/html --clean'
        //         allure includeProperties: false, jdk: 'jdk17', report: 'KernelTA/report/html', results: [[path: 'KernelTA/report/result']]
        //         echo '******自动化测试完成******'
        //     }
        // }
    
      }
    
      //updateGitlabCommitStatus会将构建状态传到gitlab
    // post {
    //       failure {
    //         updateGitlabCommitStatus name: 'build', state: 'failed'
    //         script{
    //                 def triggerUser = env.gitlabMergedByUser ?: 'adm'
    //                 def targetBranch = env.gitlabTargetBranch ?: env.GIT_BRANCH
    //                 def sourceBranch=env.gitlabSourceBranch ?: env.GIT_BRANCH
    //                 def duration = currentBuild.durationString.contains("and counting") ? currentBuild.durationString.split("and counting")[0] : currentBuild.durationString
    //                 dingtalk(
    //                           robot: '1',
    //                           type: 'MARKDOWN',
    //                           title: "构建失败: ${JOB_NAME}",
    //                           text:["###  **流水线 ${JOB_NAME}** \n- **任务ID** : ${currentBuild.displayName} \n- **状态** :  <font color=#FF0000>失败</font> \n- **提交人** : ${triggerUser} \n- **源分支** : ${sourceBranch} \n- **目标分支** : ${targetBranch} \n- **持续时间** :  ${duration} \n- **URL** : ${JOB_URL} \n"]
    //                 )
    //             }
          
    //         }
    //       success {
    //         updateGitlabCommitStatus name: 'build', state: 'success'
    //         script{
    //                 def triggerUser = env.gitlabMergedByUser ?: 'adm'
    //                 def targetBranch = env.gitlabTargetBranch ?: env.GIT_BRANCH
    //                 def sourceBranch=env.gitlabSourceBranch ?: env.GIT_BRANCH
    //                 def duration = currentBuild.durationString.contains("and counting") ? currentBuild.durationString.split("and counting")[0] : currentBuild.durationString
    //                 dingtalk(
    //                           robot: '1',
    //                           type: 'MARKDOWN',
    //                           title: "构建成功: ${JOB_NAME}",
    //                           text:["###  **流水线 ${JOB_NAME}** \n- **任务ID** : ${currentBuild.displayName} \n- **状态** :  <font color=#00FF00>成功</font> \n- **提交人** : ${triggerUser} \n- **源分支** : ${sourceBranch} \n- **目标分支** : ${targetBranch} \n- **持续时间** :  ${duration} \n- **URL** : ${JOB_URL} \n"]
    //                 )
    //             }
    //     }
    // }

}
