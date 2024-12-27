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
                    // Configure Git user details with updated email and username
                    sh "git config user.email 'anandkraws@gmail.com'"  // Use your GitHub email
                    sh "git config user.name 'anandkraws'"  // Use your GitHub username (not the email)

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

                        // Use GitHub's HTTPS method for authentication, securely handling the credentials
                        sh """
                            git config --global credential.helper 'cache --timeout=3600' 
                            git push https://github.com/anandkraws/kubernetesmanifest.git HEAD:main
                        """
                    } else {
                        echo "No changes to commit"
                    }
                }
            }
        }
    }
}
