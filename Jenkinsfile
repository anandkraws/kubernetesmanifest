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

                    // Discard any local changes (including merge conflicts)
                    sh "git reset --hard"  // This will reset the working directory to the latest commit and discard all changes.
                    
                    // Pull the latest changes from the remote main branch
                    sh "git pull origin main"  // This will merge the remote changes into the local branch

                    // Check for conflicts
                    def gitStatus = sh(script: 'git status --porcelain', returnStdout: true).trim()

                    if (gitStatus) {
                        echo "There were no conflicts and changes were successfully merged."

                        // Modify the deployment.yaml using sed (replacing docker image tag)
                        sh "sed -i 's+anandkraws/python_app.*+anandkraws/python_app:${DOCKERTAG}+g' deployment.yaml"

                        // Commit the changes
                        sh "git add ."
                        sh "git commit -m 'Done by Jenkins Job changemanifest: ${env.BUILD_NUMBER}'"

                        // Push changes to the remote repository
                        sh """
                            git config --global credential.helper 'cache --timeout=3600'
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/anandkraws/kubernetesmanifest.git HEAD:main
                        """
                    } else {
                        echo "No local changes to commit"
                    }
                }
            }
        }
    }
}
