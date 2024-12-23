# Ansible Project: AWS EC2 Automation

This project involves using Ansible to automate the creation, management, and shutdown of AWS EC2 instances. Below is an overview of the tasks performed and the setup required.

---

## Prerequisites

1. **Install boto3**:
   ```bash
   pip install boto3
   ```

2. **Install AWS Collection for Ansible**:
   ```bash
   ansible-galaxy collection install amazon.aws
   ```

3. **Setup AWS Authentication**:
   - Obtain AWS Access Key and Secret Access Key from the AWS Console.
   - Secure credentials using Ansible Vault (see below).

---

## Project Tasks

### Task 1: Create EC2 Instances Using Ansible Loops

- **Objective**: Create three EC2 instances:
  - 2 instances with Ubuntu distribution.
  - 1 instance with CentOS distribution (For simplicity, I have used the AWS linux AMI).
- **Hints**:
  - Use `connection: local` on the Ansible control node for configuration.
- **Steps**:
  1. Define an Ansible role:
     ```bash
     ansible-galaxy role create ec2
     ```
  2. Generate a password for the Ansible Vault:
     ```bash
     openssl rand -base64 2048 > vault.pass
     ```
  3. Encrypt AWS credentials using the vault:
     ```bash
     ansible-vault create ec2/defaults/main.yaml --vault-password-file vault.pass
     ```
     Add your AWS keys inside the file:
     ```yaml
     aws_access_key: YOUR_ACCESS_KEY
     aws_secret_key: YOUR_SECRET_KEY
     ```
  4. Use Ansible playbooks to provision the EC2 instances.

### Task 2: Set Up Passwordless Authentication

- **Objective**: Enable passwordless SSH between the Ansible control node and the created instances.
- **Steps**:
  1. Copy the SSH key to the instances:
     ```bash
     ssh-copy-id -f "-o IdentityFile <PATH TO PEM FILE>" ubuntu@<INSTANCE_PUBLIC_IP>
     ```
  2. List the managed nodes in the inventory file (`inventory.ini`):
     ```ini
     [ubuntu]
     ubuntu@<ubuntu-instance-ip>
     
     [centos]
     centos@<centos-instance-ip>
     ```

### Task 3: Automate Shutdown of Ubuntu Instances Using Conditionals

- **Objective**: Automate the shutdown of Ubuntu instances only, based on the gathered facts.
- **Hints**:
  - Use the `when` condition in your playbook to filter Ubuntu instances.
- **Steps**:
  1. Create a playbook `ec2-shutdown.yaml` to perform the shutdown operation:
     ```yaml
     ---
     - name: Shutdown Ubuntu Instances
       hosts: all
       gather_facts: yes
       tasks:
         - name: Shutdown Ubuntu Machines
           command: /sbin/shutdown -h now
           when: ansible_os_family == 'Debian'
     ```
  2. Execute the playbook:
     ```bash
     ansible-playbook -i inventory.ini ec2-shutdown.yaml --vault-password-file vault.pass
     ```

---

## File Structure

```plaintext
|-- ec2/
    |-- defaults/
        |-- main.yaml  # Contains encrypted AWS credentials.
    |-- tasks/
        |-- main.yaml  # Defines playbooks for EC2 creation and management.
|-- inventory.ini       # Lists managed nodes.
|-- vault.pass          # Password file for Ansible Vault.
|-- ec2-shutdown.yaml   # Playbook to shut down Ubuntu instances.
```

---

## Usage Instructions

1. Clone the repository:
   ```bash
   git clone <repository-url>
   ```

2. Navigate to the project directory:
   ```bash
   cd <project-folder>
   ```

3. Run the playbooks:
   - To provision EC2 instances:
     ```bash
     ansible-playbook -i inventory.ini ec2/tasks/main.yaml --vault-password-file vault.pass
     ```
   - To shut down Ubuntu instances:
     ```bash
     ansible-playbook -i inventory.ini ec2-shutdown.yaml --vault-password-file vault.pass
     ```

4. Ensure passwordless SSH access is set up before running tasks.

---

## Notes

- Always protect your `vault.pass` file and encrypted credential files.
- Use proper IAM roles and permissions for your AWS account to enhance security.
- This project is for my personal learning purpose.

