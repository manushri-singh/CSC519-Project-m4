# CSC519-Project - Milestone 4
Shared repo for CSC519 Devops project

## Screencast videos
[Special Milestone - Code review through GerritHub + Patch verification through Bot](https://www.youtube.com/watch?v=zYVdxdbDWFw)

[Slides](patch-verification-bot.pptx)

[Screenshots](Screenshots/)

## Project Description
The main goal of this milestone is to setup a bot which performs automatic patch verification. We tried to implement an industry standard to our project pipeline (i.e.) add peer code-review mechanism. 

Whenever a patch is submitted by a developer, the reviewer takes a look and approves it. However, there is caveat here; the reviewer might not identify a buggy code. So, we developed a bot which verifies the patch whenever it is submitted. The reviewer can now confidently review the code and approve/deny the code merge.

We designed this milestone in a way that it fits our project pipeline. Whenever the patch is approved and merged, it automatically triggers the `iTrustJob` which performs the rolling update that we designed as a part of M3. This milestone thus fits the pipeline very smoothly.

![Alt text](arch.jpg?raw=true "Architecture")

The above figure depicts a basic architecture of our milestone.
1. Developer submits patch to GerritHub
2. Jenkins listens for patches and triggers the bot job
3. Jenkins job runs all the 1164 unit test cases in iTrust and updates the patch with `Verified +1` label if all tests pass. (if not `Verified -1` is added)
4. Reviewer(s) review the code and mark success (Code-Review +1 or +2) or failure (return to step 1)
5. Developer now has the permission to click on submit (merge the code with the repo) as shown in the screenshot
6. The Gerrithub pushes the code automatically to the linked Github repo's `master` branch
7. This triggers another Jenkins job that was created in M3 for rolling update of iTrust
8. The job does rolling update on all 5 instances of iTrust

## Gerrithub - Hosted Code Review Tool
GerritHub is a hosted code review tool which works with GitHub. It provides an interface for managing and creating code reviews, which can be initiated from a GitHub pull request. By using Gerrit Code Review we can improve the security, collaboration and visibility of changes for the development team. The benefits is provides are:
1. Provides a common dashboard per project showing all pending changes.
2. Defines validation rules which needs to be passed for each commit.
3. Limit damages caused by accidental force pushes.
4. Have fine grained permissions on the branch level.

## Gerrit Trigger plugin for Jenkins
This plugin integrates Jenkins to Gerrit code review for triggering builds whenever a patch set is created. Using this plugin, we can interact with the Gerrit repository to perform our automated code review.

The link to the exact plugin is: [https://wiki.jenkins.io/display/JENKINS/Gerrit+Trigger](https://wiki.jenkins.io/display/JENKINS/Gerrit+Trigger)

## One time manual Setup:

1. Create 2 accounts in GerritHub.io - one for the developer and one for the bot
2. Once signed in to the required account, it shows all the Github repos linked to the account automatically. Import the desired project. In our case, [iTrust](https://review.gerrithub.io/#/q/project:SilleBille/iTrust)
3. Create keypairs and upload your public keys to the corresponding accounts. Place your private keys in `~/.ssh` folder (this is done by default when key is generated using `ssh-keygen` command)
4. Each developer has to clone the repo and install an additional package `sudo apt install git-review`
5. Once cloned, the developer has to add a new git remote as follows: `git remote add gerrit	ssh://<username>@review.gerrithub.io:29418/SilleBille/iTrust`
6. Do a `git review -s` to ensure that authentication works fine.


## Submitting a patch:
1. The developer modifies code as usual
2. As usual, the developer does `git add <files modified>` and `git commit` with message
3. Now, instead of usual `git push`, we do `git review`. Submits a patch to gerrithub. The output is something similar to shown below:

        dinesh@dinesh-HP:~/temp/iTrust$ git remote -v
        gerrit	ssh://SilleBille@review.gerrithub.io:29418/SilleBille/iTrust (fetch)
        gerrit	ssh://SilleBille@review.gerrithub.io:29418/SilleBille/iTrust (push)
        origin	https://github.com/SilleBille/iTrust.git (fetch)
        origin	https://github.com/SilleBille/iTrust.git (push)

        dinesh@dinesh-HP:~/temp/iTrust$ git review
        remote: Processing changes: new: 1, refs: 1, done            
        remote: 
        remote: New Changes:        
        remote:   https://review.gerrithub.io/390145 Testing        
        remote: 
        To ssh://SilleBille@review.gerrithub.io:29418/SilleBille/iTrust
        * [new branch]      HEAD -> refs/publish/master

## Environment required to run the project
    - Ubuntu 16.04 x64 (Desktop Edition) – running natively
    - Ansible 2.4.0.0 installed
    - Cloned Github repo – m4 branch
    
## Instructions to setup the Bot for automated patch verification
    git clone https://github.ncsu.edu/dmolugu/CSC519-Project.git
    cd CSC519-Project
    git checkout m4
    cd productionDeployment

    # Export GIT_ID and GIT_TOKEN
    export GIT_ID_P=<Unity ID>
    export GIT_TOKEN_P=<Personal access token>
    export AWS_KEY=<AWS KEY>
    export AWS_SECRET_KEY=<AWS SECRET KEY>

    ansible-playbook --ask-vault-pass deploy.yml -K # -K ensures installation of boto and boto3 in localhost

***Cloned repo for iTrust used in this project***
    https://github.com/SilleBille/iTrust.git


### Explanation of roles used:
- createEC2Instance = Creates the required EC2 instances. It creates 5 instances for rolling deployment of iTrust + 1 instance for installing Jenkins and shared MySQL database

- installJenkins = Installs jenkins in the corresponding instance

- installAnsible = Installs Ansible in the instance with Jenkins

- iTrustSetup = Sets up the environment required for iTrust deployment in all 5 instances

- iTrustJenkinsJob = Sets up the Jenkins job to performing rolling update. Rolling update is performed one by one using Ansible script. (ie) the ansible script is configured to bring down only one instance and perform update and bring it back.

- createBotBuildJob = Sets up the Jenkins job (bot) which is triggered whenever there is a patch submitted to Gerrithub

- fixMySqlIp = Fixes the MySQL IP in the official repo since the EC2 instance is dynamically created through the Ansible script.

#### Jenkins Credentials (Used internally)
    Username: mkd_test1
    Password: mkd_test_passwd_1

#### Vault Password (need to be provided while running the ansible-playbook)
    jenkins

### Team Members and Their Contributions

- Vishal Murugan (vmuruga) and Dinesh Prasanth M K (dmolugu) - Pair programming:
    - Researched about the Gerrit Trigger plugin and how to integrate with Jenkins 
    - Researched about how to trigger jobs for the code that is not pushed to Github and trigger correct job against the latest SHA1
    - Created Ansible Playbook for implement the bot

- Manushri (manush) and Mukundram Muraliram (mmurali5):
    - Researched about the GerritHub Tool, Code review procedure and how it interacts with Github
    - Prepared slides for final presentation
    - Recorded screencast for final presentation
