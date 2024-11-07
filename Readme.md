# Introduction to Cloud Computing for Geospatial Processing with AWS
This lecture is part of elective course in Faculty of ITC, University of Twente.
## Lecture Overview
### Morning Session: Introduction to Cloud Computing with AWS
This session covers various cloud-based service models and an overview of AWS basic services. Students will gain insights into how AWS technologies can enhance geospatial processing workflows and analysis.

### Afternoon Session: Hands-on AWS Geospatial Processing Demo
The next session focuses on practical application, demonstrating how to set up a distributed processing cluster using AWS services. Students will learn to run a sample geospatial processing workflow using Jupyter Lab as an interface, providing hands-on experience with cloud-based geospatial analysis.

## Setting-up Dask Cluster on top of AWS EC2
Setting up a Dask cluster with Jupyter Lab on AWS EC2 involves several steps. Below is a detailed tutorial to guide you through the process using `uv` as the Python package manager.

### Prerequisites

- AWS account with permissions to create EC2 instances.
- Basic knowledge of SSH and terminal commands.
- Terminal apps with SSH client installed on your computer.
- Python installed on your local machine.

### Section 1: Create EC2 Virtual Machines

1. **Log in to AWS Console:**
   - Navigate to the [AWS Management Console](https://aws.amazon.com/console/).
   - Select "EC2" from the services menu.

2. **Launch EC2 Instances:**
   - Click on "Launch Instance."
   - Choose an Amazon Machine Image (AMI), such as "Amazon Linux 2" or "Ubuntu Server."
   - Select an instance type (e.g., `t2.micro` for free tier or `t2.medium` for better performance).
   - Configure instance details (default settings are generally fine for a basic setup).
   - Add storage (default settings are usually sufficient).
   - Add tags (optional, but tagging helps in managing instances).
   - Configure security group:
     - Create a new security group or select an existing one.
     - Add rules to allow SSH (port 22) and Jupyter Lab (port 8888) access.
   - Review and launch the instance.
   - Select or create a new key pair for SSH access and download it.

3. **Access EC2 Instances with SSH:**
   - Open a terminal on your local machine.
   - Change permissions of the downloaded key pair file:
     ```bash
     chmod 400 your-key-pair.pem
     ```
   - Connect to your EC2 instance using SSH:
     ```bash
     ssh -i "your-key-pair.pem" ec2-user@your-ec2-public-dns
     ```

   Repeat the above steps to create and access the second EC2 instance.

### Section 2: Setup Dask Cluster

1. **Install `uv` and Dask on Each EC2 Instance:**
   - Update the package manager and install `uvx`:
     ```bash
     sudo yum update -y  # For Amazon Linux
     sudo apt update -y  # For Ubuntu
     curl -LsSf https://astral.sh/uv/install.sh | sh
     ```
   - Install Dask and distributed using `uvx`:
     ```bash
     uvx install dask distributed
     ```

2. **Start Dask Scheduler and Workers:**
   - On one of the instances, start the Dask scheduler:
     ```bash
     dask-scheduler
     ```
   - Note the IP address and port (usually `8786`) where the scheduler is running.
   - On both instances, start Dask workers and connect them to the scheduler:
     ```bash
     dask-worker your-scheduler-ip:8786
     ```

3. Note: to run the `dask` scheduler and worker in background, you can use following approaches:
   1. **Using `nohup`**: You can use `nohup` to run the Dask worker in the background, which allows it to continue running even if you log out of the session:

      ```bash
      nohup dask-worker tcp://192.168.1.100:8786 &
      ```

      This command will start the worker in the background and redirect its output to a file named `nohup.out` by default.

   2. **Using `&`**: Simply appending `&` to the command runs it in the background:

      ```bash
      dask-worker tcp://192.168.1.100:8786 &
      ```

   3. **Using `screen` or `tmux`**: These tools allow you to run processes in a detachable session. You can start a session, run the worker, and then detach:

      ```bash
      screen -S dask_worker
      dask-worker tcp://192.168.1.100:8786
      # Press Ctrl+A, then D to detach
      ```

      To reattach, use `screen -r dask_worker`.

4. In order to kill the existing `dask` worker process, you can find the process ID (PID) of the Dask worker using `ps` or `pgrep`, then kill it:

   ```bash
   pgrep -f 'dask-worker'
   kill <PID>
   ```

   Or directly:

   ```bash
   pkill -f 'dask-worker'

### Section 3: Setup Jupyter Lab

1. **Prepare the project:**
   - Clone the repository:
     ```bash
     git clone https://github.com/bhawiyuga/aws-geospatial-itc.git
     cd aws-geospatial-itc 
     ```
   - Syncronize the dependency:
     ```bash
     uv sync 
     ```
   - Activate the virtual environment:
     ```bash
     source .venv/bin/activate
     ```

2. **Configure Jupyter Lab:**
   - Generate a Jupyter configuration file:
     ```bash
     jupyter lab --generate-config
     ```
   - Edit the configuration file (usually located at `~/.jupyter/jupyter_lab_config.py`) to allow remote access:
     ```python
     c.ServerApp.ip = '0.0.0.0'
     c.ServerApp.open_browser = False
     c.ServerApp.port = 8888
     ```

3. **Start Jupyter Lab:**
   - Start Jupyter Lab:
     ```bash
     jupyter lab
     ```
   - Access Jupyter Lab from your local machine by navigating to `http://your-ec2-public-dns:8888` in a web browser. You might need to provide a token, which is displayed in the terminal when Jupyter Lab starts.

4. **Connect Jupyter Lab to Dask Cluster:**
   - In Jupyter Lab, install the Dask Labextension for better integration:
     ```bash
     jupyter labextension install dask-labextension
     ```
   - Use the Dask dashboard to monitor and manage your cluster directly from Jupyter Lab.

## Resources
- [AWS Free Tier](https://aws.amazon.com/free/)
- [Jupyter Lab Documentation](https://jupyterlab.readthedocs.io/)

## Contact
For any questions or further information, please contact me through email or Teams (a.bhawiyuga@utwente.nl).
