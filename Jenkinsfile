pipeline {
    agent {
        label "Slave"
    }
    triggers {
        pollSCM('0 0 * * *')
        githubPush()
    }

    tools{
        nodejs "nodejs"
    }

    stages {
        stage("Checkout"){
            steps{
                checkout scm
            }
        }
        stage("Preparing"){
            parallel {
                stage("JS preparing"){
                    steps{
                        sh 'cat www/js/* | uglifyjs -o www/min/merged-and-compressed.js --compress'
                    }
                }
                stage("CSS preparing"){
                    steps{
                        sh 'cat www/css/* | cleancss -o www/min/merged-and-minified.css'
                    }
                }
            }
        }
        stage("Archiving"){
            steps{
                sh "mkdir -p artifacts"
                sh "tar --exclude=.git --exclude=www/js --exclude=www/css --exclude=artifacts -czvf artifacts/result.tar.gz ."
                archiveArtifacts artifacts: "artifacts/result.tar.gz", fingerprint: true
            }
        }
        stage("Deploy"){
            steps{
                rtUpload (
                    serverId: "jfrog",
                    spec:
                        """{
                            "files": [
                                {
                                "pattern": "artifacts/result.tar.gz",
                                "target": "default-generic-local/result_v_${BUILD_ID}.tar.gz"
                                }
                            ]
                        }"""
                    )
            }
        }
  }
}
