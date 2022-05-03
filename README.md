# Using-AWS-Secrets-Manager-and-Ansible-Vault-to-encrypt-an-ansible-yaml-file-and-keep-its-password
 

## Description

In this tutorial, we are going to discuss how to use "AWS Secrets Manager" and "Ansible vault" to encrypt an ansible yaml file which contains sensitive data like passwords. 


So, here im going to encrypt a file named main.yml. Lets get into it.

main.yml

~~~
---

- name: "create docker image "
  hosts: build
  become: true
  vars:
    packages:
      - git
      - pip
      - docker
    repo_url: "https://github.com/Jibincl/devops-flask.git"
    repo_dir: "/home/ec2-user/flaskapp/"
    docker_user: "jibincl"
    docker_password: "dEe5SH***B58ezt"
    image_name: "jibincl/flaskrep"

  tasks:
    - name: " installing packages"
      yum:
        .
        .
        .
        etc
        
        
~~~

### Encrypting the file using ansiblevault

~~~
# ansible-vault encrypt main.yml
New Vault password:
Confirm New Vault password:
Encryption successful
~~~

I have encrypted the file with password "jibin@123"

Now, the file will be encrypted and wont be able to visible.

~~~
cat main.yml
$ANSIBLE_VAULT;1.1;AES256
65633835316634636433306439306563666534383636393838666436633866623165313735646231
3436666238373733353635616435373639633463633961370a666139313333643365366166653961
63346131343065616133376335643330353938653665616231363662643338636561396265656239
3966613831323864330a353963653230386164316262653861393461353166653638616139633230
39343661313931376533636431336139303563303530396232336364353534386135613164376564
3832323439633638373264653831656332303536313037626665
~~~

We wont be able to run the ansible playbook without the password now.

~~~
ansible-playbook -i hosts main.yml
ERROR! Attempting to decrypt but no vault secrets found
~~~

Now, we need to keep the password safe with AWS Secrete Manager. Lets check it out.



Go to AWS Secrets Manager and select "Store a new secrete"

![image](https://user-images.githubusercontent.com/100774483/165912163-77d9f883-8e01-4ce4-8ab6-0c3649edb87f.png)




Fill as below

Secret type: Other type of secret
![image](https://user-images.githubusercontent.com/100774483/165914151-6322bf3f-d054-4963-8a7b-32b9d98800b5.png)


Key/value pairs : provide a name for the key and mention the password we used to encrypt the file

![image](https://user-images.githubusercontent.com/100774483/165913459-5d849209-a560-4773-afda-07f37dd68720.png)


Encryption key: AWS/secrets manager (default)
![image](https://user-images.githubusercontent.com/100774483/165913516-22f09440-f0b9-4133-b1f9-60e099c868f2.png)


Then click on "Next"


![image](https://user-images.githubusercontent.com/100774483/166187466-abc426f3-c331-48ad-97ad-94857ce0ec85.png)

On next step, provide the secret name, description and key value. Next page is for setting automatic secret rotation and configurations associated with.
On last page, review and click "store".


Now, the ansible vault password is succesfully stored in AWS secret manager.

We need to attach an IAM role which is having "SecretsManagerReadWrite" policy to the instance for using secretsmanager. So i  manually created an IAM role with the respective policy and attached to the imstance by following the below mentioned steps

~~~
. EC2 instance
. Actions
. Security
. Modify IAM role
. Selected the role i have created
. Click "save"
~~~

So, currently the playbook cant be executed without the password. By using this methode, we are creating a script to grep the vault password from AWS secretsmanager and using the script name as an argument while running the plabook.

~~~
argument : --vault-password filename
~~~

### Script to get the password

~~~
#! /bin/bash
password=$(aws secretsmanager get-secret-value --secret-id ansiblepassword --region us-east-2 | jq -r ".SecretString" | jq -r ".ansiblepassword")
echo ${password}

~~~

#### FYI : ansible password of the name we given on AWS secrets manager while storing the vault password.

## Installing json processor

~~~
sudo yum install jq -y
~~~

Now, the ansible file can be executed using the below sommand

~~~
ansible-playbook -i hosts --vault-password ans.sh
~~~

So the vault password required for executing the ansible playbook file will be fetched from the script "ans.sh".



## Conclusion

In this repo, we discussed about encrypting an ansible files having sensitive informations with "ansible-vault" and keeping the vault password in AWS secretsmanager. 
Feel free contact if you have any concerns about the subject.


