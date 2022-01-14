# *A bite at making a CI/CD pipeline

## Needed repositories
- [Terra repository](https://github.com/GNUKalashnikov/terra)
- [Jenkins repository](https://github.com/GNUKalashnikov/jenkins)
- [Ansible + docker](https://github.com/GNUKalashnikov/docker-ansible)

Utilising:
- Jenkins
- terraform
- AWS
- Ansible

## Building Jenkins

### AWS

Building a new instance with jenkins.

Launch instance:

- Ubuntu 18.04
	- t2.micro
	- vpc (Use the *respective* vpc)
	- auto-assign public ip
	- Tags: Key: Name | Value: *anything*
	- Security Group: 8080 for Jenkins and 22 for SSH

---

Creating jenkins 

#### Pre-install Java
1. `java -version` Check that if it's installed
2. `sudo apt install default-jre`
3. `sudo apt install default-jdk`

#### Ubuntu 18.04 Install
1. `wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -`
2. `sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'`
3. `sudo apt update`
4. `sudo apt install jenkins -y`

#### System D starting 
1. `sudo systemctl start jenkins`
2. `sudo systemctl status jenkins`

### PLUGINS!
- Git
- SSH
- Terraform
    - Rather than installing define the */usr/bin/* location
- AWS plugins
    - This needs for the AWS_ACCESS/SECRET_key
    - Also required input of the eng99.pem file
- EC2 Plugins
- Ansible
    - Needed for the hosts.inv as an alternative way to interact with the hosts file

#### Plugin Manager (essential)
- [ ] Ansible
- [ ] Terraform
- [ ] AWS

### Pre-requisites
1. [Ansible install](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
A *quick* version is:
`sudo apt-add-repository ppa:ansible/ansible`
`sudo apt update`
`sudo apt install ansible`
2. [terraform install](https://www.terraform.io/cli/install/apt)
`curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -`
`sudo apt-add-repository "deb [arch=$(dpkg --print-architecture)] https://apt.releases.hashicorp.com $(lsb_release -cs) main"`
`sudo apt install terraform`
#### Credentials
Manage jenkins -> Manage credentials -> global credentials -> add 

**This** will only work if the AWS credentials have been added.
A access and secret would need to be inputted so that I can interact with aws.




### Pipeline 1 | 2
Pick a name and pick *pipeline*
This would be the pipeline or *job* that would

Make sure to [follow](https://github.com/GNUKalashnikov/iac) and use the command to install ansible

```
pipeline {
    agent any
    stages {
        stage('Empty exisiting files'){
            steps{
                sh 'rm -rf terra'
            }
        }
        stage('checkout'){
            steps{
                sh 'git clone https://github.com/GNUKalashnikov/terra.git'
            }
        }
        stage('Terraform'){
            steps{
                dir('terra'){
                    sh 'terraform init'
                    sh 'terraform apply --auto-approve'
                }
            }
        }
    }
}
```

### Pipeline 2 | 2

```
pipeline {
    agent any
    stages{
        stage('execute playbooks'){
            steps {
                dir('terra'){
                ansiblePlaybook credentialsId: 'eng99', disableHostKeyChecking: true, installation: 'ansible2', inventory: 'hosts.inv', playbook: 'playbookofplaybooks.yml'
                }

            }

        }
    }
}

```
