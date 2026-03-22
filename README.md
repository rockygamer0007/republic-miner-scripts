🚀 Republic Dual-Node Master Guide: Validator + GPU Worker
This guide provides a comprehensive, step-by-step walkthrough for setting up a Republic Validator and a Synced GPU Worker on a single machine using WSL2 and an NVIDIA GPU.

🛠 Prerequisites
Before starting, ensure your system meets these requirements:

OS: Windows 10/11 with WSL2 (Ubuntu 22.04) installed.

GPU: NVIDIA (RTX 3060 12GB or higher recommended).

Software: * Docker Desktop (Enable "WSL Integration" in Settings).

NVIDIA Container Toolkit (To allow Docker to access your GPU).

Python 3.10+ and pip installed inside WSL.

🏗 Phase 1: Validator Node Setup (The Blockchain)
The Validator handles consensus and submits your work to the network.

1. Install Republic Binary
Bash
# Create a directory for Republic
mkdir -p ~/republic && cd ~/republic

# Download and install the binary
wget https://republic-binary-link.com/republicd
chmod +x republicd
sudo mv republicd /usr/local/bin/
2. Initialize the Node
Replace ROCKYREPUBLICNODE with your own moniker.

Bash
republicd init ROCKYREPUBLICNODE --chain-id republic-testnet-1
3. Create or Recover Wallet
Bash
# To create a NEW wallet:
republicd keys add mywallet

# To RECOVER your existing wallet:
republicd keys add mywallet --recover
⚠️ WARNING: Save your 24-word seed phrase in a safe place. Never share it.

4. Syncing the Node
Instead of waiting days, use a snapshot:

Bash
# Clean old data
republicd tendermint unsafe-reset-all --home ~/.republicd

# Download and extract snapshot (Check Discord for the latest URL)
wget -O snapshot.tar.lz4 https://snapshots.republic.ai/latest.tar.lz4
lz4 -d snapshot.tar.lz4 | tar -x -C ~/.republicd
5. Launch the Node
Bash
republicd start
🧠 Phase 2: GPU Worker Setup (The AI Engine)
The GPU Worker executes the heavy AI tasks.

1. Install Python Dependencies
Bash
sudo apt update && sudo apt install python3-pip -y
pip3 install requests flask glob2
2. Prepare the Docker Inference Image
Ensure Docker Desktop is running on Windows, then pull the Republic inference image:

Bash
docker pull republic-llm-inference:latest
🤝 Phase 3: The "Handshake" (Syncing Both Nodes)
This is the secret to getting a high RESULT count. We connect the Manager to the Worker.

1. Start the GPU Worker (Brain)
Run the local_worker.py script (code provided below). It will generate a Cloudflare URL.

Bash
python3 local_worker.py
Copy the link that looks like: https://lesson-maiden.trycloudflare.com

2. Start the Manager (Muscle)
Open vps_worker_multi.sh and update the TUNNEL_URL with your new link. Then start mining:

Bash
./vps_worker_multi.sh 3
Note: For an RTX 3060, 3 slots is the "Sweet Spot" to avoid timeouts.

📂 Required Scripts
File 1: local_worker.py
Copy this code into a file named local_worker.py.

Python
import os
import time
import subprocess
import glob

# CONFIGURATION
VALIDATOR = "YOUR_VALOPER_ADDRESS"
BASE_DIR = os.getcwd()

def run_mining():
    print(f"--- Republic Synced GPU Worker Started ---")
    while True:
        # Check for job files from Manager
        job_files = glob.glob(os.path.join(BASE_DIR, "job_*.txt"))
        if not job_files:
            time.sleep(2)
            continue

        for job_file in job_files:
            try:
                with open(job_file, 'r') as f:
                    job_id = f.read().strip()
                
                print(f"🚀 Processing Job: {job_id}")
                result_file = os.path.join(BASE_DIR, f"result_{job_id}.json")
                
                # Execute AI Task via Docker
                cmd = ["docker", "run", "--rm", "--gpus", "all", "republic-llm-inference:latest"]
                result = subprocess.run(cmd, capture_output=True, text=True)
                
                with open(result_file, "w") as rf:
                    rf.write(result.stdout)

                print(f"✅ Job {job_id} Complete!")
                if os.path.exists(job_file):
                    os.remove(job_file)

            except Exception as e:
                print(f"❌ Error: {e}")
        time.sleep(1)

if __name__ == "__main__":
    run_mining()
File 2: vps_worker_multi.sh
Copy your existing multi-slot script here. Ensure you remove your private keys or seeds before sharing!

🌡️ Maintenance & Optimization
Thermal Control: If your GPU hits 80°C, reduce the number of slots.

URL Update: If you restart your PC, you must update the TUNNEL_URL in the Bash script with the new one from the Python worker.

VRAM Clearing: If jobs get stuck, run:
docker rm -f $(docker ps -aq)

Created by ROCKYREPUBLICNODE | Join the Republic on Discord
