# Prompt Injection

The Hackergram application contains two endpoints that allow attackers to explore vulnerabilities specific to the integration of Large Language Models (LLMs) in web applications: `/generate_post` and `/leaderboard`.

## Improper Output Handling

The `/leaderboard` endpoint of Hackergram demonstrates improper input handling vulnerabilities in practice. In this attack, you will explore the lack of output sanitization from the LLM model. The objective is to attempt to change another user's password using the LLM integrated into the leaderboard endpoint. To perform the attack, follow these steps:

1. Experiment with the functionality by submitting a normal prompt, such as “Give me the count of leaderboard members.” Observe how the prompt generates a query to the application’s database.
2. Next, try to find a way to change another user's password through the endpoint.
3. If you succeed, you should see the SQL query that was executed.
4. To confirm the success of the attack, log out of your current account and log in to the affected account.

!!! note "Additional exercise"
    Drop a table from the Hackergram database.