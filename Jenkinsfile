node {
    def app

    stage('Clone repository') {
        checkout scm
        // Ensure we're on the correct branch (e.g., main)
        sh "git checkout main || git checkout -b main"  // Checkout main, or create it if it doesn't exist
    }

    stage('Update GIT') {
        script {
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                withCredentials([usernamePassword(credentialsId: 'git_token', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    // Configure Git user details
                    sh "git config user.email 'raj@cloudwithraj.com'"
                    sh "git config user.name 'RajSaha'"

                    // Print the deployment.yaml to check the current content
                    sh "cat deployment.yaml"

                    // Modify the deployment.yaml using sed (replacing docker image tag)
                    sh "sed -i 's+anandkraws/python_app.*+anandkraws/python_app:${DOCKERTAG}+g' deployment.yaml"

                    // Verify the change after sed
                    sh "cat deployment.yaml"

                    // Stage the changes and commit
                    sh "git add ."
                    sh "git commit -m 'Done by Jenkins Job changemanifest: ${env.BUILD_NUMBER}'"

                    // If there are no changes to commit, this will fail with exit code 1. Handle it:
                    script {
                        def gitStatus = sh(script: 'git status --porcelain', returnStdout: true).trim()
                        if (gitStatus) {
                            // If there are changes to commit, push them
                            sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/kubernetesmanifest.git HEAD:main"
                        } else {
                            echo "No changes to commit"
                        }
                    }
                }
            }
        }
    }
}
