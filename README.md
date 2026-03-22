# Republic Dual-Node Master Guide: Validator + GPU Worker 🚀

This guide provides a step-by-step walkthrough for setting up a **Republic Validator** and a **Synced GPU Worker** on a single machine using WSL2 and an NVIDIA GPU.

---

## 🛠 Prerequisites
* **OS:** Windows 10/11 with [WSL2 (Ubuntu 22.04)](https://apps.microsoft.com/store/detail/ubuntu-22043-lts/9PN20MSR04DW).
* **GPU:** NVIDIA (RTX 3060 12GB or higher recommended).
* **Software:** * [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Enable "WSL Integration" in Settings).
    * [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).

---

## 🏗 Phase 1: Validator Setup (The Blockchain Node)

### 1. Install Republic Binary
```bash
wget https://republic-binary-link.com/republicd
chmod +x republicd
sudo mv republicd /usr/local/bin/


### 2. Initialize Node
Replace YourMoniker with your node name (e.g., ROCKYREPUBLICNODE).

```Bash
republicd init YourMoniker --chain-id republic-testnet-1
3. Create/Recover Wallet
Keep your seed phrase safe!

Bash
republicd keys add mywallet
4. Syncing
Download the latest snapshot to skip days of syncing:

Bash
# Example snapshot command (Check official Discord for latest link)
wget https://snapshots.republic.ai/latest.tar.lz4 | lz4 -d | tar -x -C ~/.republicd
5. Create the Validator
Once synced, register your node:

Bash
republicd tx staking create-validator \
  --amount=1000000arai \
  --pubkey=$(republicd tendermint show-validator) \
  --moniker="YourMoniker" \
  --chain-id=republic-testnet-1 \
  --commission-rate="0.10" \
  --from=mywallet
🧠 Phase 2: GPU Worker Setup (The AI Engine)
The GPU Worker executes AI tasks assigned to your validator.

1. Install Python Dependencies
Bash
sudo apt update && sudo apt install python3-pip -y
pip3 install requests flask
2. Configure the Worker
The local_worker.py script waits for your Validator to drop a job file, then uses your GPU via Docker to finish it.

🤝 Phase 3: The Handshake (Syncing Validator & GPU)
To earn points, your Manager (Bash) and Worker (Python) must talk to each other.

1. Start the Worker (Brain)
Bash
python3 local_worker.py
Copy the Cloudflare URL (e.g., https://lesson-maiden.trycloudflare.com).

2. Start the Manager (Muscle)
Edit your vps_worker_multi.sh and paste that Cloudflare URL into the TUNNEL_URL section. Then run:

Bash
./vps_worker_multi.sh 5
📊 Monitoring & Performance
Check Success: Look for ✅ Job Complete in your terminal.

Dashboard: Your RESULT count on points.republicai.io should update every 30-60 mins.

VRAM Management: If your RTX 3060 crashes, reduce your slots from 5 to 3.

📂 Repository Files
local_worker.py: Synced Python script that watches for local jobs.

vps_worker_multi.sh: Multi-slot manager for job submission.

Created by [YourUsername] | Supported by the Republic Community


---

### **Part 3: The Actual Files for your Repo**

Since you asked for the files, here is the code for the two most important ones. You can copy these into Notepad on Windows, save them with the correct names, and then upload them to GitHub.

#### **File 1: `local_worker.py`**
```python
import os
import time
import subprocess
import glob

# CONFIGURATION
VALIDATOR = "raivaloper1yhcjx7cgs2387quars36njfs4y2vsqtqlfnc6l"
# Note: Update this URL every time you restart!
PUBLIC_URL = "[https://your-current-url.trycloudflare.com](https://your-current-url.trycloudflare.com)"
BASE_DIR = os.getcwd()

def run_mining():
    print(f"Starting SYNCED Republic GPU Worker...")
    print(f"Watching for jobs in: {BASE_DIR}")

    while True:
        job_files = glob.glob(os.path.join(BASE_DIR, "job_*.txt"))
        
        if not job_files:
            time.sleep(2)
            continue

        for job_file in job_files:
            try:
                with open(job_file, 'r') as f:
                    job_id = f.read().strip()
                
                if not job_id: continue
                
                print(f"\nFound Real Job: {job_id}")
                result_file = os.path.join(BASE_DIR, f"result_{job_id}.json")
                
                cmd = ["docker", "run", "--rm", "--gpus", "all", "republic-llm-inference:latest"]
                result = subprocess.run(cmd, capture_output=True, text=True)
                
                with open(result_file, "w") as rf:
                    rf.write(result.stdout)

                print(f"✅ Job {job_id} complete. Result saved!")
                
                if os.path.exists(job_file):
                    os.remove(job_file)

            except Exception as e:
                print(f"Error: {e}")
        time.sleep(1)

if __name__ == "__main__":
    run_mining()
