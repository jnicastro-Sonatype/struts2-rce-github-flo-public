import groovy.json.*

node () {
   def policyEvaluation

   stage('Preparation') {
      checkout scm
   }
    stage('Build') {
      // Run the maven build
         sh "./mvnw -Dmaven.test.failure.ignore clean install"
   }

   stage('IQ Policy Check') {
         env.GIT_REPO_NAME = env.GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')
         policyEvaluation = nexusPolicyEvaluation failBuildOnNetworkError: false, iqScanPatterns: [[scanPattern: '**/target/struts2-rest-showcase.war'], [scanPattern: '**/terraform/aws/aws.large.tfplan']], iqApplication: "${env.GIT_REPO_NAME}", iqStage: 'build', jobCredentialsId: ''
        //policyEvaluation = nexusPolicyEvaluation advancedProperties: '', enableDebugLogging: true, failBuildOnNetworkError: false, iqApplication: selectedApplication('struts2-rce-github-flo'), iqScanPatterns: [[scanPattern: 'target/struts2-rest-showcase.war'], [scanPattern: 'terraform/aws/aws.large.tfplan']], iqStage: 'release', jobCredentialsId: ''
        sh "echo ${policyEvaluation.applicationCompositionReportUrl}"
   }

    stage('Create TAG') {
        createTag nexusInstanceId: 'nxrm3', tagAttributesJson: "{\"IQScan\": \"${policyEvaluation.applicationCompositionReportUrl}\", \"JenkinsBuild\": \"${BUILD_URL}\"}", tagName: "IQ-Policy-Evaluation_${currentBuild.number}"
    }

    stage('Publish') {
        nexusPublisher nexusInstanceId: 'nxrm3', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: 'war', filePath: './target/struts2-rest-showcase.war']], mavenCoordinate: [artifactId: 'struts2-rest-showcase', groupId: 'org.apache.struts', packaging: 'war', version: '2.5.10']]], tagName: "IQ-Policy-Evaluation_${currentBuild.number}"
    }
}  

