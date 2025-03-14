# The Hackergram Web application

Hackergram is an intentionally vulnerable Web application that was designed with the purpose of providing students and security enthusiasts with an environment where they can familiarize themselves with and gain practical experience in identifying, exploiting, and mitigating various types of Web vulnerabilities, mainly CSRF, XSS and SQLi. It consists of a simple social networking website. In Hackergram, users can create an account by registering their username, password, and name. Once registered, users can log into their account and start using its various features:

* Posts are the main feature of the website. Users can write posts that will then be shown to other users on the homepage.
* Posts can also be edited or deleted.
* Users can connect with each other by sending friendship requests, which can be accepted or declined.
* A user can also remove his/her friendship with another user at any time.
* Users can navigate to any user’s profile to see their information (i.e., name, username, picture, and bio), as well as their posts and friends.
* A user is allowed to change his/her own settings (i.e., name, password, picture, and bio).
* There is a search functionality that allows users to search for posts with a specific word or content and search for other users by username.

The initial state of Hackergram already includes several users, posts, friendships, and friendship requests. Moreover, the application includes a special endpoint called /reset that allows to reset the application to its initial state at any moment. This is particularly useful since some attacks completely break the application and this offers the possibility of going back to a clean state whenever necessary.

The following table lists all existing users in the Web application and their respective passwords. The user ```mr_robot``` is the one with the most user data on the application.

| User	    | Password          |
|-----------|-------------------|
| admin     | 1_4m_Th3_4dm1n    |
|mr_robot   | elliot123         |
|dpr        | silk-road         |
|satoshi    | bitcoin2009       |
|heisenberg | walter1958        |
|rick       | RickC-137         |
|stark      | winterfell        |
|anon1      | 1                 |
|anon2      | 2                 |
|anon3      | 3                 |