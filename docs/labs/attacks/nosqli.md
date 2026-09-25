# NoSQL Injection

## Objective

This experiment demonstrates that MongoDB-backed endpoints are just as injectable as SQL ones when user
input is dropped into a query document unsanitized: a plain string can smuggle in a MongoDB operator
(`$where`, `$gt`, `$regex`, …) and change what the query actually matches.

## Affected Hackergram functionality

- `/chatlog`: LLM chat-history search (`search` query parameter)
- `/messages`: direct-message search (`search` query parameter)

## Prerequisites

Both endpoints read from MongoDB, which is included in both the simple and the full deployment. The
`/messages` exercise works with either one. The `/chatlog` exercise needs the full deployment: that endpoint
shows the LLM chat history, which only exists when the LLM features are available, even though this
particular attack is classical rather than LLM-mediated.

## Initial state

Log in as any user on the webterm; `/reset` is not strictly required for the syntax-injection sub-attack, but
run it first if you want a clean chat/message history to compare against.

## Vulnerable implementation

`views.py`: `chatlog()` and `messages()`. The vulnerable versions pass the raw `search` string into the
MongoDB query without escaping or operator filtering, either via `$where`-style string evaluation or by
interpolating the search value directly into a `$regex` filter without `re.escape()`, and without rejecting
non-string JSON operators submitted as the parameter value. Contrast with the fixed versions in "Inspect and
modify" below, which apply `re.escape(search)` and construct the query document explicitly server-side.

## Experiment

### Syntax injection (`/chatlog`)

The Hackergram application is vulnerable to syntax-based NoSQL injection via the `/chatlog` endpoint. This
endpoint allows users to view their LLM chat history, but fails to properly sanitize the input provided in
the search field.

1. Using the webterm, navigate to the chatbot messages page on Hackergram while logged in.
2. In the search field, enter the payload:
   ```
   ') || true || ('
   ```
3. Observe that the attack is successful: instead of filtering by user-specific queries, the application
   displays all chat messages stored in the system.

### Operator injection (`/messages`)

The Hackergram application is also vulnerable to NoSQL operator injection via the `/messages` endpoint.

1. In the webterm, locate the search-messages page of Hackergram.
2. In the search field, submit the payload:
   ```
   {"$gt": ""}
   ```
3. Observe that the server answers with an error, confirming that the backend query is influenced by the
   injected operator.

## Expected result

- Syntax injection: the chat log view shows every user's chat messages, not just the logged-in user's.
- Operator injection: the server returns a MongoDB error that reveals the query was built directly from the
  raw `search` value (rather than treating it as an opaque string).

## Why it works

Both endpoints hand the client-supplied `search` value to MongoDB close to as-is. MongoDB's query language is
JSON, so a string containing an operator key (`$gt`, `$where`, …) is indistinguishable from a legitimate
filter fragment once it reaches the query builder, because there's no separate "data" channel the way parameterized
SQL provides.

## Reset / cleanup

Run `/reset` to restore the original chat/message history.

## Inspect and modify

To mitigate this type of attack, the two vulnerable endpoints must be refactored to eliminate insecure query
construction practices. In particular, dangerous operators such as `$where` must be avoided, arbitrary JSON
queries built directly from user input must be prohibited, and any input used in a regular expression must be
sanitized with `re.escape()`.

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

Apply this fix, repeat both attacks, and confirm neither the `') || true || ('` nor the `{"$gt": ""}` payload
still changes what the query matches.

## Exercise

Based on the error message from the operator-injection sub-attack, try to extract all messages from the
database.

## Hint

<details>
<summary>💡 Hint</summary>

If the raw `search` value is ever accepted as a non-string JSON operator object (rather than coerced to a
string first), a value like `{"$ne": null}` matches every document, bypassing the intended per-user filter
entirely.

</details>
