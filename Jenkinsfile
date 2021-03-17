
def rtServer, buildInfo, rtMaven, xrayConfig, xrayResults

pipeline {
    agent { label 'master' }
    tools {
        jdk 'JDK8' //Make sure this tool in configured in Manage Jenkins -> Global Tool Configuration
        maven 'mvn360' //Make sure this tool in configured in Manage Jenkins -> Global Tool Configuration
    }
    parameters {
        string (name: 'ART_URL', defaultValue: 'http://10.132.0.77:8081/artifactory', description: 'Artifactory where artifacts will be deployed/resolved')
        string (name: 'ART_USER', defaultValue: 'admin', description: 'Artifactory user for deploy/resolve artifacts')
        string (name: 'ART_PASSWORD', defaultValue: 'Password1', description: 'Artifactory password for deploy/resolve artifacts')
        string (name: 'ART_RELEASE_REPO', defaultValue: 'mvn-libs-release', description: 'Virtual Repository where artifacts will be deployed/resolved (Releases)')
        string (name: 'ART_SNAPSHOT_REPO', defaultValue: 'mvn-libs-snapshot', description: 'Virtual Repository where artifacts will be deployed/resolved (Snapshots)')
        booleanParam (name: 'XRAY_SCAN', defaultValue: true, description: 'Scan artifacts using Xray')
        booleanParam (name: 'FAIL_BUILD', defaultValue: true, description: 'Fail build if any violation is found in Xray')
    }
    stages {
        stage('Init properties'){
            steps {
                script {
                    rtServer = Artifactory.newServer url: "${params.ART_URL}", username: "${params.ART_USER}", password: "${params.ART_PASSWORD}"
                    rtMaven = Artifactory.newMavenBuild()
                    buildInfo = Artifactory.newBuildInfo()
                    // These variables should be PARAMETERIZED
                    rtMaven.deployer releaseRepo: 'mvn-libs-release', snapshotRepo: 'mvn-libs-snapshot', server: rtServer
                    rtMaven.resolver releaseRepo: 'mvn-maven-jcenter', snapshotRepo: 'mvn-maven-jcenter', server: rtServer
                }
            }
        }
        stage('Checkout'){
            steps {
                git url: 'https://github.com/jfrogdev/project-examples.git' //JFrog main project examples repository
            }
        }
        stage('Install'){
            steps {
                script {
                    rtMaven.run pom: 'maven-example/pom.xml', goals: 'clean install -U', buildInfo: buildInfo
                }
            }
        }
        stage('Publish'){
            steps {
                script {
                    rtServer.publishBuildInfo buildInfo
                }
            }
        }
        stage('Xray Scan'){
    
            steps {
                script {
                    xrayConfig = [
                        'buildName'     : buildInfo.name,
                        'buildNumber'    : buildInfo.number,
                        'failBuild'     : true
                    ]
                    xrayResults = rtServer.xrayScan xrayConfig
                    echo xrayResults as String
                }
            }
        }
    }
}
