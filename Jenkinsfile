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
                    // Configure Git user details with updated email and username
                    sh "git config user.email 'anandkraws@gmail.com'"
                    sh "git config user.name 'anandkraws'"

                    // Print the deployment.yaml to check the current content
                    sh "cat deployment.yaml"

                    // Modify the deployment.yaml using sed (replacing docker image tag)
                    sh "sed -i 's+anandkraws/python_app.*+anandkraws/python_app:${DOCKERTAG}+g' deployment.yaml"

                    // Verify the change after sed
                    sh "cat deployment.yaml"

                    // Check if there are any changes to commit
                    def gitStatus = sh(script: 'git status --porcelain', returnStdout: true).trim()

                    // If there are changes to commit, stage, commit, and push
                    if (gitStatus) {
                        sh "git add ."
                        sh "git commit -m 'Done by Jenkins Job changemanifest: ${env.BUILD_NUMBER}'"
                        // Push the changes to GitHub
                        sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/kubernetesmanifest.git HEAD:main"
                    } else {
                        echo "No changes to commit"
                    }
                }
            }
        }
    }
}
