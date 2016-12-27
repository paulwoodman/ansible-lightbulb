Ansible AWS training provisioner
================================

This is an automated lab setup for ansible training. It creates 5 nodes per user in the `users` list.

* One control node from which ansible will be executed from
* Three web nodes that coincide with the three nodes in lightbulb's original design
* And one node where haproxy is installed (via lightbulb lesson)

Usage
-----

## AWS Setup

To set up the lab for Ansible training, follow these steps.

1. Create an Amazon AWS account.

2. Create an ssh key pair called 'ansible' (Network & Security->Key Pairs->Create Key Pair). Download the private key to your .ssh directory, e.g. to .ssh/ansible.pem. Alternatively, you can upload your own public key into AWS.

   If using an AWS generated key add it to the ssh-agent:

   ```bash
   ssh-add ~/.ssh/ansible.pem
   ```

3. Login to the [EC2 dashboard](https://eu-west-1.console.aws.amazon.com/ec2/v2/home?region=eu-west-1#) (this is for eu-west) and create a VPC with a single public subnet. Make note of both the VPC and Subnet IDs.

4. Create an AWS [Access Key ID and Secret Access Key](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSGettingStartedGuide/AWSCredentials.html).

5. Create a free Sendgrid account if you don't have one at [sendgrid.com](http://sendgrid.com) and record your credentials.

6. Install the `sendgrid` python library: (Currently Not Working)

   ```bash
   pip install sendgrid
   ```

7. Install `boto`.

   ```bash
   pip install boto
   ```

8. Create a `boto` configuration file containing your AWS access key ID and secret access key.

    ```bash
    mkdir ~/.aws
    touch ~/.aws/credentials
    chmod 600 ~/.aws/credentials

    # The file should contain the following:
    [default]
    aws_access_key_id = [access key ID]
    aws_secret_access_key = [secret key]
    ```

9. Clone the lightbulb repo:

   ```bash
   git clone https://github.com/gdykeman/lightbulbv2
   cd lightbulb/aws_lab_setup
   ```

10. Define the following variables, either in a file passed in using `-e @extra_vars.yml` or directly in a `vars` section in `aws_lab_setup\infra-aws.yml`:

    ```yaml
    aws_key_name: ansible      # SSH key you created earlier
    aws_region: us-west-1     # region where the nodes will live
    name_prefix: TRAINING-LAB  # name prefix for all the VMs
    sendgrid_user: username   # username for the Sendgrid module
    sendgrid_pass: 'passwordgoeshere'     # sendgrid accound password
    instructor_email: 'Ansible Instructor <helloworld@redhat.com>'  # address you want the emails to arrive from
    admin_password: changeme123           # set this to something better if you'd like
    vpc_id: vpc-2a994a4c                  # VPC ID of the VPC you created earlier
    vpc_subnet_id: subnet-b70447fe        # VPC Subnet ID of the VPC you created earlier
    current_ssh_port: 22
    ssh_port: 22
    ```

11. Create a `users.yml` by copying sample-users.yml and adding all your
students:

   ```yaml
   users:
      - username: student1
        email: student1@redhat.com

      - username: student2
        email: student2@redhat.com
   ```

12. You should now be able to run the playbook:

   ```bash
   $ ssh-add -l     # ensure that your ansible key is loaded into the agent
   $ ansible-playbook infra-aws.yml -e @extra_vars.yml -e @users.yml
   ```

13. Check on the EC2 console and you should see instances being created like:

   ```bash
   TRAINING-LAB-<student_username>-node1|2|control
   ```

If successful all your students will be emailed the details of their hosts including addresses and credentials, and an instructor_inventory file will be created listing all the student machines.

The playbook will start the instances, configure them for password auth, and dump an ansible inventory file for each user with their IPs and credentials into the current directory, then it will email every user their respective inventory file. This will also generate an instructor inventory file in the current directory which will let the instructor access the nodes of any student by simply targeting the the username as a host group.

Ensure you have boto installed and configured, and that your public key is installed in the target region. The lab will get created in us-west-1 by default.
