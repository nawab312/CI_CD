Create a Pipeline with 3 Parameters:
- Get input for our custom deployment name
- Select which AZ to Deploy
- Checkbox to Confirm Deployments

```groovy
pipeline {
    agent any
    parameters {
        string(name: "deploymentName", defaultValue: "", description: "Deployment Name")
        choice(name: "azDeploy", choices: ["EU-WEST-2A", "EU-WEST-2B", "EU-WEST-2C"], description: "Which AZ")
        booleanParam(name: "confirmDep", defaultValue: false, description: "CONFIRM DEPLOYMENT")
    }
    stages {
        stage("Deploy") {
            steps {
                echo "String set to ${deploymentName} \n"
                echo "Choice set to ${azDeploy}"
            }
        }
    }
}
```
