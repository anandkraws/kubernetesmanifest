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

                    // Ensure pull strategy is merge (we'll use merge to handle divergent branches)
                    sh "git config pull.rebase false"  // Use merge when pulling

                    // Pull the latest changes from the remote repository and merge them
                    sh """
                        git fetch origin  // Fetch remote changes first
                        git pull origin main  // Merge remote changes into local main branch
                    """

                    // Check the git status to determine if we have changes to commit
                    def gitStatus = sh(script: 'git status --porcelain', returnStdout: true).trim()

                    if (gitStatus) {
                        // Stage and commit the changes if there are any
                        sh "git add ."
                        sh "git commit -m 'Done by Jenkins Job changemanifest: ${env.BUILD_NUMBER}'"
                    } else {
                        echo "No local changes to commit"
                    }

                    // Push changes to the remote repository
                    sh """
                        git config --global credential.helper 'cache --timeout=3600'
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/anandkraws/kubernetesmanifest.git HEAD:main
                    """
                }
            }
        }
    }
}
