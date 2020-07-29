/**
 * This Jenkinsfile builds Theia across the major OS platforms
 */

pipeline {
    agent none
    environment {
        BUILD_TIMEOUT = 180
    }
    stages {
        stage('Installers') {
            parallel {
                stage('Create Linux Installer') {
                    agent {
                        kubernetes {
                            label 'node-pod'
                            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: node
    image: thegecko/theia-dev
    command:
    - cat
    tty: true
    resources:
      limits:
        memory: "4Gi"
        cpu: "2"
      requests:
        memory: "4Gi"
        cpu: "2"
    volumeMounts:
    - name: yarn-cache
      mountPath: /.cache
    - name: electron-cache
      mountPath: /.electron-gyp
  - name: jnlp
    volumeMounts:
    - name: volume-known-hosts
      mountPath: /home/jenkins/.ssh
  volumes:
  - name: yarn-cache
    emptyDir: {}
  - name: electron-cache
    emptyDir: {}
  - name: volume-known-hosts
    configMap:
      name: known-hosts
"""
                        }
                    }
                    steps {
                        timeout(time: "${env.BUILD_TIMEOUT}") {
                            container('node') {
                                sh "printenv"
                                sh "yarn cache clean"
                                sh "yarn --frozen-lockfile --force"
                                sh "yarn package"
                            }
                        }
                        container('jnlp') {
                            sshagent(['projects-storage.eclipse.org-bot-ssh']) {
                                sh '''
                                    ssh genie.theia@projects-storage.eclipse.org rm -rf /home/data/httpd/download.eclipse.org/theia/snapshots/linux
                                    ssh genie.theia@projects-storage.eclipse.org mkdir -p /home/data/httpd/download.eclipse.org/theia/snapshots/linux
                                    scp -r dist/theia* genie.theia@projects-storage.eclipse.org:/home/data/httpd/download.eclipse.org/theia/snapshots/linux
                                '''
                            }
                        }
                    }
                }
                stage('Create Mac Installer') {
                    agent {
                        label 'macos'
                    }
                    steps {
                        timeout(time: "${env.BUILD_TIMEOUT}") {
                            sh "printenv"
                            sh "yarn cache clean"
                            sh "yarn --frozen-lockfile --force"
                            sh "yarn package"
                        }
                        sshagent(['projects-storage.eclipse.org-bot-ssh']) {
                            sh '''
                                ssh genie.theia@projects-storage.eclipse.org rm -rf /home/data/httpd/download.eclipse.org/theia/snapshots/macos
                                ssh genie.theia@projects-storage.eclipse.org mkdir -p /home/data/httpd/download.eclipse.org/theia/snapshots/macos
                                scp -r dist/theia* genie.theia@projects-storage.eclipse.org:/home/data/httpd/download.eclipse.org/theia/snapshots/macos
                            '''
                        }
                    }
                }
                stage('Create Windows Installer') {
                    agent {
                        label 'windows'
                    }
                    steps {
                        timeout(time: "${env.BUILD_TIMEOUT}") {
                            bat "set"
                            bat "yarn cache clean"
                            bat "yarn --frozen-lockfile --force"
                            bat "yarn package"
                        }
                        sshagent(['projects-storage.eclipse.org-bot-ssh']) {
                            bat '''
                                find dist -name *.msi -maxdepth 0 -exec curl -o signed.{} -F file=@{} http://build.eclipse.org:31338/winsign.php \;
                                ssh genie.theia@projects-storage.eclipse.org rm -rf /home/data/httpd/download.eclipse.org/theia/snapshots/windows
                                ssh genie.theia@projects-storage.eclipse.org mkdir -p /home/data/httpd/download.eclipse.org/theia/snapshots/windows
                                scp -r dist/theia* genie.theia@projects-storage.eclipse.org:/home/data/httpd/download.eclipse.org/theia/snapshots/windows
                            '''
                        }
                    }
                }
            }
        }
    }
}