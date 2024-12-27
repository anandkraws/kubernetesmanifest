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
                // Use Jenkins credentials securely
                withCredentials([usernamePassword(credentialsId: 'git_token', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    // Configure Git user details
                    sh "git config user.email 'anandkraws@gmail.com'"
                    sh "git config user.name 'anandkraws'"  // Correct GitHub username (not email)

                    // Print the deployment.yaml to check the current content
                    sh "cat deployment.yaml"

                    // Modify the deployment.yaml using sed (replacing docker image tag)
                    sh "sed -i 's+anandkraws/python_app.*+anandkraws/python_app:${DOCKERTAG}+g' deployment.yaml"

                    // Verify the change after sed
                    sh "cat deployment.yaml"

                    // Fetch the latest changes from the remote repository to avoid non-fast-forward errors
                    sh "git fetch origin"

                    // Attempt to merge the remote changes with your local branch (this will not overwrite local changes)
                    sh "git merge origin/main || true" // Use `|| true` to continue even if merge conflicts occur

                    // Check if there are any changes to commit
                    def gitStatus = sh(script: 'git status --porcelain', returnStdout: true).trim()

                    // If there are changes to commit, stage, commit, and push
                    if (gitStatus) {
                        sh "git add ."
                        sh "git commit -m 'Done by Jenkins Job changemanifest: ${env.BUILD_NUMBER}'"

                        // Ensure the repository URL uses credentials explicitly
                        sh """
                            git config --global credential.helper 'cache --timeout=3600'
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/anandkraws/kubernetesmanifest.git HEAD:main
                        """
                    } else {
                        echo "No changes to commit"
                    }
                }
            }
        }
    }
}
