pipeline {
    agent any

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

  for (repo in repos()) {
    jobs[repo] = { 
        stage(repo) {
           echo "Step for $repo"
        }
    }
  }
  return jobs
}
