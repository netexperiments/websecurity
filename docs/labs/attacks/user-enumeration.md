#User enumeration

This exercise resorts to kerbrute and a list of popular usernames, to obtain the usernames configured at the DC (valid usernames). Use the [users.txt](../intra-domain-attacks/users.txt) file stored in the course materials folder as the list of popular usernames. Run the following command while performing a Wireshark capture at the attacker’s interface:
```
./kerbrute userenum -d polaris.local users.txt --dc 192.168.122.10
```
You may use a kerberos filter to analyze the Wireshark capture.

!!! question
    Explain the user enumeration process, using a Wireshark capture.

??? success "Answer"
    The attacker sent nine AS-REQ messages over UDP, since there are nine usernames in the users.txt file. Received five PRINCIPAL UNKNOWN error messages, three PREAUTH REQUIRED, and one RESPONSE_TOO_BIG. The latter is due to user angela.moss with does not require pre-authentication. In this case, a TCP connection is established with the AD, the AS-REQ is sent again, and an AS-REP is received. The AS-REP message carries the username of the requesting user. The correlation between requests and responses in the UDP messages is done through the port number. 

!!! question
    How does kerbrute correlate the requests with the responses?

!!! question
    What is the behavior in case of users not requiring pre-authentication?

