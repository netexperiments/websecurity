# NoSQL Injection

Hackergram application is vulnerable to two types of NoSQL injection. The following exercises will address these vulnerabilities and the corresponding countermeasures, using different endpoints in each

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
