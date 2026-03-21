Republic Dual-Node Setup Guide (Validator + GPU Worker)
This repository contains the necessary scripts to sync a local GPU worker with a Republic Validator node. This setup ensures that your GPU specifically processes the jobs submitted by your own validator, maximizing your RESULT count and success rate.

Prerequisites
OS: Windows 10/11 with WSL2 (Ubuntu 22.04 recommended).

Hardware: NVIDIA GPU (e.g., RTX 3060 12GB) with latest drivers.

Software: * Docker Desktop (Must be running with WSL2 integration enabled).

Python 3.10+ installed inside WSL.

Installation
Clone the repository:

Bash
git clone https://github.com/your-username/republic-miner-scripts
cd republic-miner-scripts
Prepare the environment:
Ensure your scripts have execution permissions:

Bash
chmod +x vps_worker_multi.sh
Directory Structure:
It is recommended to run this from a dedicated drive path to avoid permission issues:
/mnt/d/RepublicGPU/republic-miner-scripts/

Configuration (The "Handshake")
The most important part of this setup is ensuring the Manager (Bash script) and the Worker (Python script) are talking to each other.

Step 1: Start the GPU Worker
Open your first terminal and run:

Bash
python3 local_worker.py
Note the Cloudflare URL generated in the output (e.g., https://xxxx.trycloudflare.com).

This script is configured to "watch" the local directory for job_*.txt files.

Step 2: Configure the Manager
Open vps_worker_multi.sh in a second terminal:

Bash
nano vps_worker_multi.sh
TUNNEL_URL: Paste the Cloudflare URL from Step 1.

DATA_DIR: Set this to your absolute script path.

WALLET/VALIDATOR: Ensure your rai... and raivaloper... addresses are correct.

Running the Node
Terminal 1 (The Brain): Keep local_worker.py running. It will wait for jobs.

Terminal 2 (The Muscle): Start the manager with your desired slots (e.g., 5 slots for a 3060):

Bash
./vps_worker_multi.sh 5
How it works:
The Manager submits a job to the blockchain and creates a local job_ID.txt file.

The Worker detects the file, runs the AI model via Docker, and saves a result_ID.json.

The Manager sees the result and broadcasts the transaction hash (TX) to the network.

Troubleshooting
TabErrors: If you edit the Python script, ensure you use 4 spaces for indentation, not tabs.

Timeout/Result 0: This usually means the TUNNEL_URL has changed. Restart the Python worker and update the URL in the Bash script.

VRAM Issues: If Docker crashes, reduce the number of slots in the Bash script (e.g., from 5 down to 3).

Success Metrics
Once synced, your dashboard should reflect an increasing RESULT count. The transition from "Submitting" to "✅ Job Complete" should happen within 60-90 seconds per task.
