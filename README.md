# Advanced-File-Integrity-Checker-Using-Cryptographic-Hashing
his project ensures file integrity by verifying if files have been tampered with. It uses cryptographic hash functions (e.g., SHA-256) to generate unique checksums for files. The program can:  Create a database of file hashes for a directory. Verify the current file hashes against the saved hashes. Identify tampered, deleted, or newly added files.
import hashlib
import os
import json

def calculate_hash(file_path):
    """Calculate SHA-256 hash of a file."""
    hash_algo = hashlib.sha256()
    try:
        with open(file_path, "rb") as f:
            while chunk := f.read(4096):
                hash_algo.update(chunk)
        return hash_algo.hexdigest()
    except FileNotFoundError:
        return None

def create_hash_database(directory, output_file):
    """Generate hashes for all files in a directory and save them to a JSON file."""
    hash_database = {}
    for root, _, files in os.walk(directory):
        for file in files:
            file_path = os.path.join(root, file)
            hash_database[file_path] = calculate_hash(file_path)
    
    with open(output_file, "w") as f:
        json.dump(hash_database, f, indent=4)
    
    print(f"Hash database saved to {output_file}.")

def verify_hashes(directory, hash_file):
    """Verify files in a directory against a stored hash database."""
    with open(hash_file, "r") as f:
        stored_hashes = json.load(f)
    
    current_hashes = {}
    for root, _, files in os.walk(directory):
        for file in files:
            file_path = os.path.join(root, file)
            current_hashes[file_path] = calculate_hash(file_path)
    
    # Compare hashes
    tampered_files = []
    missing_files = []
    new_files = []
    
    # Check for tampered and missing files
    for file_path, old_hash in stored_hashes.items():
        if file_path not in current_hashes:
            missing_files.append(file_path)
        elif current_hashes[file_path] != old_hash:
            tampered_files.append(file_path)
    
    # Check for new files
    for file_path in current_hashes:
        if file_path not in stored_hashes:
            new_files.append(file_path)
    
    # Report results
    print("\nFile Integrity Report:")
    print(f"Tampered Files: {len(tampered_files)}")
    for file in tampered_files:
        print(f"  - {file}")
    
    print(f"Missing Files: {len(missing_files)}")
    for file in missing_files:
        print(f"  - {file}")
    
    print(f"New Files: {len(new_files)}")
    for file in new_files:
        print(f"  - {file}")
    
    print("\nVerification Complete.")

# Main menu for the tool
def main():
    print("File Integrity Checker")
    print("1. Create Hash Database")
    print("2. Verify Files Against Hash Database")
    choice = input("Enter your choice (1/2): ")
    
    if choice == "1":
        directory = input("Enter the directory to hash: ")
        output_file = input("Enter the output file name (e.g., hashes.json): ")
        create_hash_database(directory, output_file)
    elif choice == "2":
        directory = input("Enter the directory to verify: ")
        hash_file = input("Enter the hash database file (e.g., hashes.json): ")
        verify_hashes(directory, hash_file)
    else:
        print("Invalid choice. Exiting.")

# Run the program
if __name__ == "__main__":
    main()
