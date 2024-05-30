pipeline {
    agent any

    stages {
        stage('Parallel') {
            steps {
                script {
                    parallel parallelJobs()
                }
            }
        }
    }
}


def accounts() {
  return ["dynamic", "list"]
}

def parallelJobs() {
  jobs = [:]

  for (account in accounts()) {
    jobs[account] = { stage(account) {
       echo "Step for $account"
     }
    }
  }
  return jobs
}
