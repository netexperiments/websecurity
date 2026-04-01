# NoSQL Injection
This attack consists of exploiting NoSQL databases as an attack vector. NoSQL databases are nonrelational databases that store data in formats like documents, key-value pairs, graphs, or wide columns and are created to deal with unstructured or semi-structured data.
In contrast to traditional SQL databases, which manage structured data, NoSQL databases are often preferred for their scalability and flexibility. However, they can be more susceptible to certain types of attacks. These vulnerabilities may result in unauthorized access, data modification, denial of service, increased privileges, and data exposure by exploiting misconfigurations or weaknesses in query design and access controls.

Hackergram application is vulnerable to two types of NoSQL injection. The following exercises will address these vulnerabilities and the corresponding countermeasures, using different endpoints in each.

## Syntax Injection

The Hackergram application is vulnerable to syntax-based NoSQL injection via the /chatlog endpoint. This endpoint allows users to view their LLM chat history. However, it fails to properly sanitize the input provided in the search field, making it possible for an attacker to inject code that alters the backend query logic. To verify the vulnerability, follow the steps below:

1.	Using the webterm, navigate to the chatbot messages page on Hackergram while logged in.

2.	In the search field, enter the following payload:
``` ') || true || (' ``` 

3.	Observe that the attack is successful, since instead of filtering by user-specific queries, the application displays all chat messages stored in the system.

## Operator Injection

In addition to syntax manipulation, the Hackergram application is also vulnerable to NoSQL operator injection via the /messages endpoint. To verify the vulnerability, follow these steps:

1.	In the webterm, locate the search messages page of Hackergram.
2.	In the search field, submit the following payload: 
{"$gt": ""}
3.	Observe that the server answers with an error, meaning that the backend query is indeed influenced by the injected operators.
Additional exercise: Based on the error and information, try to extract all messages from the database.


## Countermeasure
To mitigate this type of attack, the two vulnerable endpoints must be refactored to eliminate insecure
query construction practices. In particular, the use of dangerous operators, such as the $where operator,
must be avoided. Arbitrary JavaScript Object Notation (JSON) queries constructed directly from user
input should be prohibited, and input used in regular expressions must be properly sanitized. The following presents the corrected, secure implementation of both endpoints.

??? note "Secure endpoint implementation"

    ```python
    @app.route("/chatlog")
    def chatlog():
        if 'username' in session:
            username = session['username']
            user = models.get_user_settings(username)

            search = request.args.get('search', '').strip()
            if search:
                # Use safe MongoDB operators instead of $where
                query = {
                    "user": username,
                    "prompt": {
                        "$regex": re.escape(search),
                        "$options": "i"  # case insensitive
                    }
                }
            else:
                query = {"user": username}

            try:
                logs = models.get_chat_history(query)
            except Exception as e:
                return error(e)

            return render_template("chatlog.html", current_user=user, chatlogs=logs)
        else:
            return redirect(url_for('login'))

    @app.route('/messages', methods=['GET'])
    def messages():
        if 'username' not in session:
            return redirect(url_for('login'))

        username = session['username']
        user = models.get_user_settings(username)
        search = request.args.get('search', '').strip()

        query = {
            "$or": [
                {"sender": username},
                {"recipient": username}
            ]
        }
        if search:
            query["message"] = {
                "$regex": re.escape(search),
                "$options": "i"
            }

        messages = list(models.mongo.db.direct_messages.find(query).sort("timestamp", -1))
        for msg in messages:
            msg["_id"] = str(msg["_id"])

        return render_template("messages.html", current_user=user, messages=messages, search=search)
    ```
