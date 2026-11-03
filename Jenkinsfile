import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=c44a3d95-3a59-464b-94a7-824e4127d0ec',
        'AZURE_TENANT_ID=e0d9dd6b-e6ad-4c81-99dd-686842a46f7c']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'zemingazureapp'
      
      // 使用 Service Principal 凭据登录 Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      
      // 使用 Azure CLI 直接将 WAR 包部署到 Web App（自动处理解压和发布）
      sh "az webapp deploy --resource-group ${resourceGroup} --name ${webAppName} --src-path target/calculator-1.0.war --type war"
      
      // 登出 Azure（安全收尾）
      sh 'az logout'
    }
  }
}
