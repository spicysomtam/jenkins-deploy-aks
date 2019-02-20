pipeline {

   parameters {
    choice (name: 'action', choices: 'create\ndestroy', description: 'Create/update or destroy the aks cluster.')
    string (name: 'cluster', defaultValue : 'demo', description: "AKS cluster name;eg demo creates cluster named eks-demo.")
    string (name: 'vm_type', defaultValue : 'Standard_DS2_v2', description: "k8s worker node vm type.")
    string (name: 'num_workers', defaultValue : '1', description: "k8s number of worker instances.")

    credentials (name: 'principle', 
      credentialType: 'com.microsoft.azure.util.AzureCredentials', 
      description: 'Jenkins credential service principle that allows access to create AKS resources.', 
      defaultValue: '', 
      required: true)

    string (name: 'location', defaultValue : 'ukwest', description: "AZURE location/region.")
  }

  options {
    disableConcurrentBuilds()
    timeout(time: 1, unit: 'HOURS')
  }

  agent { label 'master' }

  stages {

    stage('Setup') {
      steps {
        script {
          cluster = params.cluster
          resGroup = 'aks' + cluster.capitalize()
          currentBuild.displayName = "#" + env.BUILD_NUMBER + " " + params.action + " aks" + cluster.capitalize()

          withCredentials([azureServicePrincipal(params.principle)]) {
            sh """
              az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
              az role assignment list
            """
          }

        }
      }
    }

    stage('Create') {
      when {
        expression { params.action == 'create' }
      }
      steps {
        script {
          withCredentials([azureServicePrincipal(params.principle)]) {
            sh """
              az group create --name ${resGroup} --location ${params.location}
              az aks create \
                --service-principal $AZURE_CLIENT_ID \
                --client-secret $AZURE_CLIENT_SECRET \
                --resource-group ${resGroup} \
                --name ${resGroup} \
                --node-count ${params.num_workers} \
                --node-vm-size ${params.vm_type} \
                --enable-addons monitoring \
                --generate-ssh-keys
            """

            println "Include this in your ~/.kube/config to access the cluster:"
            sh "az aks get-credentials --name ${resGroup} --resource-group ${resGroup} --file -"
          }
        }
      }
    }

    stage('Destroy') {
      when {
        expression { params.action == 'destroy' }
      }
      steps {
        script {
          // deleteing the resource group deletes everything in it; azure is so easy...
          sh "az group delete --name ${resGroup} --yes"
        }
      }
    }

  }

  post {
    always {
      sh " az logout "
    }
  }

}
