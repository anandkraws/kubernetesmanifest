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

                    // Pull the latest changes from the remote main branch
                    sh "git pull origin main"  // This will merge the remote changes into the local branch

                    // Modify the deployment.yaml file using sed (replacing the docker image tag)
                    sh "sed -i 's+anandkraws/python_app.*+anandkraws/python_app:${DOCKERTAG}+g' deployment.yaml"

                    // Stage the modified file for commit
                    sh "git add deployment.yaml"

                    // Commit the changes
                    sh "git commit -m 'Updated deployment.yaml to use python_app:${DOCKERTAG}'"

                    // Push the changes to the remote repository
                    sh """
                        git config --global credential.helper 'cache --timeout=3600'
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/anandkraws/kubernetesmanifest.git HEAD:main
                    """
                }
            }
        }
    }
}
