# Incrypted-chat-app
client.py
import socket
from cryptography.fernet import Fernet

# Step 1: Get encryption key from server
key = input("Enter the encryption key from server: ").encode()
cipher = Fernet(key)

# Step 2: Connect to server
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('localhost', 9999))
print("\n✅ Connected to server.\n")

while True:
    # Get user message
    msg = input("You: ")

    # Encrypt message
    encrypted_msg = cipher.encrypt(msg.encode())
    print(f"[Encrypted Message Sent]: {encrypted_msg.decode(errors='ignore')}")
    print(f"[Plain Message]: {msg}\n")

    # Send encrypted message
    client.send(encrypted_msg)

    # Receive reply
    encrypted_reply = client.recv(1024)
    print(f"[Received Encrypted Reply]: {encrypted_reply.decode(errors='ignore')}")

    # Decrypt reply
    reply = cipher.decrypt(encrypted_reply).decode()
    print(f"[Decrypted Reply from Server]: {reply}\n")
server.py
import socket
from cryptography.fernet import Fernet

# Step 1: Generate key (only once)
key = Fernet.generate_key()
cipher = Fernet(key)
print("\n🗝️ Encryption Key (share this with client):", key.decode())
print("🔒 Server started...\n")

# Step 2: Create socket
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('localhost', 9999))
server.listen(1)
print("📡 Waiting for client connection...")

conn, addr = server.accept()
print("✅ Connected with:", addr, "\n")

while True:
    encrypted_msg = conn.recv(1024)
    if not encrypted_msg:
        break

    # Show encrypted data from client
    print(f"\n[Received Encrypted Data]: {encrypted_msg.decode(errors='ignore')}")
    
    # Decrypt message
    decrypted_msg = cipher.decrypt(encrypted_msg).decode()
    print(f"[Decrypted Message from Client]: {decrypted_msg}")

    # Server reply input
    reply = input("\nYour reply: ")

    # Encrypt reply
    encrypted_reply = cipher.encrypt(reply.encode())

    # Show encryption process
    print(f"[Encrypted Reply Sent]: {encrypted_reply.decode(errors='ignore')}")
    print(f"[Plain Reply]: {reply}\n")

    # Send encrypted reply
    conn.send(encrypted_reply)

conn.close()
