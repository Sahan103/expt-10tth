trigger:
- main

pool:
  vmImage: 'windows-latest'

variables:
  buildConfiguration: 'Release'
  azureSubscription: 'YourAzureSubscription'
  resourceGroupName: 'YourResourceGroup'
  location: 'YourLocation'
  appServiceName: 'YourAppService'
  servicePrincipalId: 'YourServicePrincipalId'
  servicePrincipalKey: 'YourServicePrincipalKey'

stages:
- stage: Build
  displayName: 'Build Stage'
  jobs:
  - job: Build
    displayName: 'Build Job'
    steps:
    - task: UseDotNet@2
      inputs:
        packageType: 'sdk'
        version: '6.x'
        installationPath: $(Agent.ToolsDirectory)/dotnet

    - script: |
        echo "Building the project..."
        dotnet build --configuration $(buildConfiguration)
      displayName: 'Build Project'

- stage: Test
  displayName: 'Test Stage'
  dependsOn: Build
  jobs:
  - job: Test
    displayName: 'Test Job'
    steps:
    - script: |
        echo "Running tests..."
        dotnet test --configuration $(buildConfiguration)
      displayName: 'Run Tests'

- stage: Publish
  displayName: 'Publish Stage'
  dependsOn: Test
  jobs:
  - job: Publish
    displayName: 'Publish Job'
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Publish Application'
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
        zipAfterPublish: true

    - task: PublishBuildArtifacts@1
      displayName: 'Publish Artifacts'
      inputs:
        pathtoPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'drop'

- stage: Deploy
  displayName: 'Deploy Stage'
  dependsOn: Publish
  jobs:
  - job: Deploy
    displayName: 'Deploy Job'
    environment: 'staging'
    steps:
    - task: AzureWebApp@1
      displayName: 'Deploy to Azure App Service'
      inputs:
        azureSubscription: $(azureSubscription)
        appType: 'webAppWindows'
        appName: $(appServiceName)
        package: '$(System.DefaultWorkingDirectory)/**/*.zip'
        deployToSlotOrASE: true
        slotName: 'staging'
        resourceGroupName: $(resourceGroupName)
        location: $(location)

    - task: AzureCLI@2
      displayName: 'Swap Slots'
      inputs:
        azureSubscription: $(azureSubscription)
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          az login --service-principal -u $(servicePrincipalId) -p $(servicePrincipalKey) --tenant $(azureSubscription)
          az webapp deployment slot swap --resource-group $(resourceGroupName) --name $(appServiceName) --slot staging --target-slot production
