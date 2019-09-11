import groovy.json.*

node () {
   def mvnHome, commitId
    
   stage('Preparation') { // for display purposes
      checkout scm


   }
   stage('Kill the container') {
      // Run the maven build
      catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
      sh "/usr/local/bin/docker kill \$(/usr/local/bin/docker ps | grep hack | awk '{print \$1;}')"
      }

      }

   stage('Build') {
      // Run the maven build
      try{
        if (isUnix()) {
           sh "./mvnw  -B -Dmaven.test.failure.ignore clean package"
        } else {
           bat("mvnw.cmd -B -Dmaven.test.failure.ignore clean package")
        }
        
        currentBuild.result = 'SUCCESS'

      }catch(Exception err){
        currentBuild.result = 'FAILURE'
      
      }
      sh "cd /Users/ryansheldrake/.jenkins/jobs/struts2-rce/branches/master/workspace/ && /usr/local/bin/docker build -t hack ."
      sh "/usr/local/bin/docker run -d -p 9080:8080 hack"
      sh "echo current build status ${currentBuild.result}"
      
   }

   
   stage('Lifecycle Evaluation'){
    // postGitHub commitId, 'pending', 'analysis', 'Nexus Lifecycle Analysis is running'

      def policyEvaluationResult = nexusPolicyEvaluation failBuildOnNetworkError: false, iqApplication: 'ryan-cloudbees-live', iqStage: 'release', jobCredentialsId: ''
    /*  if (currentBuild.result == 'FAILURE'){
        postGitHub commitId, 'failure', 'analysis', 'Nexus Lifecycle Analysis failed',"${policyEvaluationResult.applicationCompositionReportUrl}"
        return
      } else {
        postGitHub commitId, 'success', 'analysis', 'Nexus Lifecycle Analysis succeeded',"${policyEvaluationResult.applicationCompositionReportUrl}"
      } */
   }
   
}

def postGitHub(commitId, state, context, description, target_url="http://localhost:8080") {
         def payload = JsonOutput.toJson(
           state: state,
           context: context,
           description: description,
           target_url: target_url
          )
        sh "curl -H \"Authorization: token ${gitHubApiToken}\" --request POST --data '${payload}' https://api.github.com/repos/${project}/statuses/${commitId}"
}
  

