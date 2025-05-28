pipeline {
    agent any

    parameters {
        string(name: 'DEST_DIR', defaultValue: '/root/Project', description: 'Destination directory to deploy files')
        string(name: 'BACKUP_DIR', defaultValue: '/root/backup', description: 'Directory to store backup files')
        string(name: 'RELEASE_BRANCH', defaultValue: 'main', description: 'GitHub branch to checkout from')
    }

    environment {
        DEST_DIR = "${params.DEST_DIR}"
        BACKUP_DIR = "${params.BACKUP_DIR}"
        RELEASE_BRANCH = "${params.RELEASE_BRANCH}"
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                script {
                    echo "Checking out from branch: ${RELEASE_BRANCH}"
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "refs/heads/${RELEASE_BRANCH}"]],
                        extensions: [],
                        userRemoteConfigs: [[
                            url: 'https://github.com/sandeep-r-mishra/release.git',
                            credentialsId: 'Github'
                        ]]
                    ])
                    sh 'ls -la'
                }
            }
        }

        stage('Create Backup') {
            steps {
                script {
                    echo "Creating backup of files in ${DEST_DIR}"
                    def timestamp = sh(returnStdout: true, script: 'date +"%Y%m%d_%H%M%S"').trim()
                    def versionedBackupDir = "${BACKUP_DIR}/backup_${timestamp}"

                    sh "mkdir -p ${versionedBackupDir}"

                    sh """
                        for file in ${DEST_DIR}/*.txt; do
                            if [ -f "\$file" ]; then
                                filename=\$(basename "\$file")
                                cp -v "\$file" "${versionedBackupDir}/\${filename}"
                            fi
                        done
                    """

                    echo "Backup created at: ${versionedBackupDir}"
                    sh "ls -l ${versionedBackupDir}"
                }
            }
        }

        stage('Deploy Files') {
            steps {
                script {
                    echo "Deploying .txt files from workspace to ${DEST_DIR}"
                    sh """
                        find . -name '*.txt' -exec cp -v {} ${DEST_DIR} \\;
                        echo "Files after deployment:"
                        ls -l ${DEST_DIR}/*.txt || echo '⚠️ No .txt files deployed.'
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    echo "Verifying files in ${DEST_DIR}"
                    sh """
                        if compgen -G "${DEST_DIR}/*.txt" > /dev/null; then
                            ls -l ${DEST_DIR}/*.txt
                            echo "Total files deployed: \$(ls -1 ${DEST_DIR}/*.txt | wc -l)"
                        else
                            echo "⚠️ No .txt files found in ${DEST_DIR}"
                        fi
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Files from '${RELEASE_BRANCH}' branch have been deployed to '${DEST_DIR}'."
        }
        failure {
            echo "❌ Deployment failed. Check the logs for details."
        }
    }
}
