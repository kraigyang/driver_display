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
  repos().each { repo ->
    jobs[repo] = { 
        stage(repo) {
           echo "Step for $repo"
        }
    }
  }
  return jobs
}
