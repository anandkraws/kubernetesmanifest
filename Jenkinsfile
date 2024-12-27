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

                    // Fetch the latest changes from the remote repository
                    sh "git fetch origin"

                    // Merge the latest changes into your local branch
                    // If you want to merge, use:
                    sh "git merge origin/main || true"

                    // Alternatively, you can use `git pull` instead of `fetch + merge`
                    // sh "git pull origin main || true"

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
