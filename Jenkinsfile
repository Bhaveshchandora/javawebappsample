import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=52662c76-5baf-488a-980e-1f93d29907ae',
            'AZURE_CLIENT_ID=4ace2e1f-9bda-473f-b8b3-453054e35f6e',
           'AZURE_CLIENT_SECRET=irWkQ2_WyDGbQ349GnpEHHRk.Ub5-zqWsm',
          'AZURE_TENANT_ID=0adb040b-ca22-4ca6-9447-ab7b049a22ff']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'QuickstartJenkins-rg'
      def webAppName = 'newjenkinss-app-bhavesh'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'irWkQ2_WyDGbQ349GnpEHHRk.Ub5-zqWsm', usernameVariable: '4a5ac69f-9294-42c0-8dc4-a8175bec85e4')]) {
       sh '''
          az login --service-principal -u 4a5ac69f-9294-42c0-8dc4-a8175bec85e4 -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
