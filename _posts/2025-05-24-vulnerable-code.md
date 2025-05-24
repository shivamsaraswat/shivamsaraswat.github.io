---
layout: post
title: 'How to Write Vulnerable Code: A Master Class in What NOT to Do'
tags: [Cyber Security, SSDLC, Security Engineering]
color: rgb(4, 108, 219)
permalink: vulnerable-code/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

Welcome, future security incident creators! Today we'll learn the fine art of writing code so vulnerable that hackers will send you thank-you cards. Because who needs job security when you can have *security vulnerabilities*?

<!--more-->

<hr>

<h1 id="contents-">Contents <a name="top"></a></h1>
* TOC
{:toc}

<hr>

## Rule #1: Never, Ever Validate User Input

Why waste precious time checking what users send to your application? Trust is the foundation of all relationships, right?

```python
# The PERFECT way to handle user input
def get_user_data():
    user_input = request.get('data')
    # Just use it directly! What could go wrong?
    execute_query(f"SELECT * FROM users WHERE name = '{user_input}'")
```

Congratulations! You've just created an SQL injection vulnerability. Hackers everywhere are applauding your generosity.

## Rule #2: Hardcode All Your Secrets

Environment variables are for cowards. Real developers put their secrets right in the code where everyone can see them.

```python
# Maximum transparency achieved!
DATABASE_PASSWORD = "admin123"
API_KEY = "sk-1234567890abcdef"
AWS_SECRET_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

Don't forget to commit these to your public GitHub repository. Sharing is caring!

## Rule #3: Use HTTP for Everything

HTTPS is just HTTP with extra steps. Who needs encryption when you have optimism?

```
      User's Password
           |
           v
    [Plain HTTP Request]
           |
           v
      [Your Server]
           |
           v
       [Database]
    
"Look ma, no encryption!"
```

## Rule #4: Store Passwords in Plain Text

Hashing passwords is so complicated. Just store them as-is in your database. It's much easier to debug when you can read them!

```python
def create_user(username, password):
    # Why make it complicated?
    db.execute("INSERT INTO users VALUES (?, ?)", username, password)
```

When hackers steal your database, they'll appreciate not having to crack any hashes. So thoughtful of you!

## Rule #5: Never Update Dependencies

That package from 2015 still works, doesn't it? Why fix what ain't broken?

```dockerfile
# Dockerfile of champions
FROM python:2.7  # Python 2 forever!
RUN pip install flask==0.10.1  # Vintage!
RUN pip install requests==1.0.0  # Classic!
```

Those 847 known vulnerabilities in your dependencies? They add character to your application.

## Rule #6: Give Everyone Admin Privileges

Authorization is discrimination. True equality means everyone gets admin access.

```python
def check_permissions(user):
    # Everyone is special!
    return True
```

## Rule #7: Log Everything (Including Sensitive Data)

Debugging is important, so log EVERYTHING. Credit card numbers, passwords, social security numbers - if it exists, log it!

```python
logger.info(f"User {username} logged in with password {password}")
logger.info(f"Processing payment for card {credit_card_number}")
```

Store these logs in a publicly accessible S3 bucket for maximum convenience.

## Rule #8: Disable All Security Headers

Security headers are like vegetables - everyone says they're good for you, but who actually wants them?

```python
# Clean and simple!
@app.after_request
def remove_headers(response):
    # Headers are overrated
    return response
```

No Content-Security-Policy, no X-Frame-Options, no nothing. Pure, vulnerable freedom!

## Rule #9: Trust All File Uploads

Users only upload nice things like cat pictures, right? No need to check file types or scan for malware.

```python
def upload_file():
    file = request.files['file']
    # Just save it! YOLO!
    file.save(f"/var/www/html/{file.filename}")
```

Bonus points if you let them upload PHP files to your web root.

## Rule #10: Never Handle Errors Properly

When something goes wrong, share all the details with the user. Stack traces are basically free documentation!

```python
try:
    do_something_risky()
except Exception as e:
    # Share the love!
    return f"Error: {e}\nStack trace: {traceback.format_exc()}"
```

## The Vulnerability Stack Diagram

```
        [Your "Secure" Application]
                    |
    +---------------+---------------+
    |               |               |
[SQL Injection] [XSS Paradise] [Auth Bypass]
    |               |               |
[Plain Text PWDs] [No HTTPS] [File Upload RCE]
    |               |               |
    +-------+-------+-------+-------+
                    |
              [Data Breach]
                    |
              [You're Fired]
```

## Conclusion

There you have it! Ten foolproof ways to ensure your code is as vulnerable as a newborn kitten in a thunderstorm. By following these guidelines, you'll create applications that security researchers will study for years to come (as examples of what not to do😉).

Remember: Writing secure code is hard. Writing vulnerable code? That's easy! Just ignore every security best practice, and you'll be creating breaches in no time. 🎯

---

**Disclaimer⚠️**: *This blog post is entirely sarcastic and educational. Please do the EXACT OPPOSITE of everything suggested here. Write secure code, validate inputs, use HTTPS, hash passwords, update dependencies, implement proper authorization, protect sensitive data in logs, use security headers, validate file uploads, and handle errors securely. Your future self (and your users) will thank you.*🙏

---


**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

<a href="#top" style="float: right"><strong>Back to Top⮭</strong> </a>
