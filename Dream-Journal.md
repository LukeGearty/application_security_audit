# Application Overview

A web-based dream logging platform built using Flask. It allows users to create accounts, authenticate, and submit free-form text entries describing their dreams. The application uses a form-based authentication workflow and processes user input via HTTP POST requests.

[Link to Repository](https://github.com/LukeGearty/dream_journal)

# Identified Security Issues

## Input Handling and Error Management
The application was vulnerable to a BadRequestKeyError during the login process due to unsafe access of form parameters. The login handler directly references form fields using dictionary-style access without validation or error handling.

```python
if request.method == "POST":
        username = request.form["username"]
        password = request.form["password"]
```

If an expected form field is missing from the request, the application raises an unhandled exception, resulting in a server-side error. This behavior enables a trivial denial-of-service condition and demonstrates insufficient defensive input validation.

## Weak Authentication Controls

The application enforces no complexity or length requirements for usernames or passwords. Additionally, no rate limiting or account lockout mechanisms are implemented, allowing attackers to perform efficient brute-force attacks against the login endpoint.

During testing, automated credential-guessing attacks were successfully conducted using common wordlists, demonstrating that user account compromise is feasible with minimal effort.

```bash
hydra -l adam -P /usr/share/wordlists/rockyou.txt 192.168.104.22 -s 5000 \
http-post-form "/login:username=^USER^&password=^PASS^:Invalid username or password"
```