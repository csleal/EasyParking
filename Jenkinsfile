node {

  def resourceGroupName = 'rg-global'
  def resourceGroupLocation = 'brazilsouth'
  def appServicePlanName = 'globalPlan'
  def appServicePlanTier = 'FREE'
  def webAppName = 'global-rm89052'
  def webAppRuntime = '"java:11:Java SE:11"'
  def packagePath = 'target/epictaskapi-0.0.1-SNAPSHOT.jar'

  stage('Extrair Codigo Fonte') {
    echo 'Obtendo o Código Fonte ...'
    checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/csleal/EasyParking.git']]])
  }

  stage('Build') {
    echo 'Empacotando o projeto...'
    sh '/opt/maven/bin/mvn clean package'
  }

  stage('Credenciais Azure') {
    echo 'Obtendo credenciais...'
    withCredentials([usernamePassword(credentialsId: 'AzureService', 
      passwordVariable: 'AZURE_CLIENT_SECRET',
      usernameVariable: 'AZURE_CLIENT_ID')]) {
      echo 'Logando na Azure...'
      sh 'az login -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET'
    }
  }

  stage('Criar Infra') {
    echo 'Criando o Grupo de Recursos...'
    sh "az group create --name $resourceGroupName --location $resourceGroupLocation"
    echo 'Criando Plano de Serviço...'
    sh "az appservice plan create --name $appServicePlanName --resource-group $resourceGroupName --sku $appServicePlanTier"
    echo 'Criando o Web App...'
    sh "az webapp create --name $webAppName --plan $appServicePlanName --resource-group $resourceGroupName --runtime $webAppRuntime"
  }

  stage('Deploy') {
     echo 'Realizando o Deploy na Azure...'
     sh "az webapp deploy --resource-group $resourceGroupName --name $webAppName --src-path $packagePath --type jar"
  }

}
