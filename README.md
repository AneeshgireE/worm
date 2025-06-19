# worm
dont make it exe file



import os
import shutil
import sys
import logging
import hashlib
import random
import time
import json
from datetime import datetime

# === SAFETY PROTOCOLS ===
TEST_DIR = "/tmp/security_lab"
SAFE_SANDBOX = os.path.join(TEST_DIR, "exfil_simulation")
ALLOWED_FILE_TYPES = ['.txt', '.csv', '.fake']  # Only these file types are touched

if not os.path.exists(SAFE_SANDBOX):
    os.makedirs(SAFE_SANDBOX, mode=0o700)
    # Create dummy files for the simulation
    with open(os.path.join(SAFE_SANDBOX, "sample.fake"), "w") as f:
        f.write("SIMULATED_SENSITIVE_DATA: credit_card=4111111111111111")

if not os.path.abspath(__file__).startswith(TEST_DIR):
    print("[!] SAFETY LOCK: Only runs in designated test directory")
    sys.exit(1)

# === LOGGING ===
LOG_FILE = os.path.join(TEST_DIR, "exfil_monitor.log")
logging.basicConfig(filename=LOG_FILE, level=logging.INFO)
logging.info(f"\n=== DATA EXFIL SIMULATION STARTED @ {datetime.now()} ===\n")

def simulate_data_discovery():
    """Locates 'sensitive' files (only .fake in our sandbox)"""
    found_files = []
    
    for root, _, files in os.walk(SAFE_SANDBOX):
        for file in files:
            if any(file.endswith(ext) for ext in ALLOWED_FILE_TYPES):
                file_path = os.path.join(root, file)
                found_files.append(file_path)
                logging.info(f"Discovered: {file_path}")
    
    return found_files

def simulate_data_theft(files):
    """'Steals' data by copying to simulated attacker server"""
    exfil_dir = os.path.join(SAFE_SANDBOX, "exfiltrated_data")
    os.makedirs(exfil_dir, exist_ok=True)
    
    stolen_data = []
    
    for file_path in files:
        try:
            # Create a 'stolen' copy
            new_name = f"stolen_{os.path.basename(file_path)}"
            dest_path = os.path.join(exfil_dir, new_name)
            shutil.copy(file_path, dest_path)
            
            # Log what would be stolen (without real data)
            with open(file_path, "r") as f:
                content = f.read()
                simulated_data = content[:20] + "...[truncated]"
            
            stolen_data.append({
                "original_path": file_path,
                "exfil_path": dest_path,
                "sample_data": simulated_data,
                "timestamp": str(datetime.now())
            })
            
            logging.info(f"Copied: {file_path} -> {dest_path}")
        except Exception as e:
            logging.error(f"Failed to copy {file_path}: {str(e)}")
    
    return stolen_data

def simulate_exfiltration(data):
    """Pretends to send data to attacker server (just logs)"""
    logging.info("\n=== SIMULATED EXFILTRATION ATTEMPT ===")
    
    # Generate fake C2 server
    fake_c2 = f"https://attacker.{random.randint(1,999)}.com/exfil"
    
    for entry in data:
        entry["c2_server"] = fake_c2
        logging.info(json.dumps(entry, indent=2))
    
    print(f"[*] Simulated exfiltration to {fake_c2}")
    return fake_c2

def main():
    print("\n==== ETHICAL DATA EXFILTRATION SIMULATION ====")
    print(f"Sandbox: {SAFE_SANDBOX}\n")
    
    # Simulation workflow
    found_files = simulate_data_discovery()
    if not found_files:
        print("[-] No test files found (check SAFE_SANDBOX)")
        return
    
    stolen_data = simulate_data_theft(found_files)
    c2_server = simulate_exfiltration(stolen_data)
    
    print("\n[!] SIMULATION COMPLETE")
    print(f"Logs: {LOG_FILE}")
    print(f"Sample 'stolen' files in: {os.path.join(SAFE_SANDBOX, 'exfiltrated_data')}")
    print(f"Simulated C2: {c2_server}")

if __name__ == "__main__":
    main()
