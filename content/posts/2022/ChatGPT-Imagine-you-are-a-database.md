---
title: "ChatGPT: Imagine you are a database server"
date: 2022-12-02T08:41:49-08:00
draft: true
tags: [
        "machine learning", "aiml", "db"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "ChatGPT: Imagine you are a database server"
  description:  "Turning ChatGPT into a SQL Server database server."
  image: "https://embracethered.com/blog/images/2022/chatgpt-db.png"
---


After reading [this post](https://www.engraved.blog/building-a-virtual-machine-inside/) about ChatGPT imitating Linux, **I wanted it to be a database server**.

Let's try it out!

> Imagine you are a Microsoft SQL Server. I type commands, and you reply with the result, and no other information or descriptions. Just the result. Start with exec xp_cmdshell 'whoami';


[![ChatGPT - Database](/blog/images/2022/chatgpt-db1.png)](/blog/images/2022/chatgpt-db1.png)

Wow, this looks like a promising start.

And, it "thinks" that it is running as `LOCAL SYSTEM` - quite funny actually. 

Let's see what databases it knows about?

[![ChatGPT - Database](/blog/images/2022/chatgpt-db2.png)](/blog/images/2022/chatgpt-db2.png)

Nice, looks like it had some sample databases as training data, like `AdventuresWorks`. :) 


Now, let's create a new database.


[![ChatGPT - Database](/blog/images/2022/chatgpt-db3.png)](/blog/images/2022/chatgpt-db3.png)

Next, create a table to store some information.

[![ChatGPT - Database](/blog/images/2022/chatgpt-db4.png)](/blog/images/2022/chatgpt-db4.png)

And now, let's insert some data!

[![ChatGPT - Database](/blog/images/2022/chatgpt-db5.png)](/blog/images/2022/chatgpt-db5.png)

Yes, that seemed to have worked.  Can we select it?


[![ChatGPT - Database](/blog/images/2022/chatgpt-db6.png)](/blog/images/2022/chatgpt-db6.png)

Very cool!

Now, I was wondering... 

Can ChatGPT write a stored procedure to perform an UPSERT on the newly created `users` table? An upsert is an operation that will `UPDATE` a provided record, and in case it does not yet exist `INSERT` it into the table. 

[![ChatGPT - Database](/blog/images/2022/chatgpt-db7.png)](/blog/images/2022/chatgpt-db7.png)

Additionally, ChatGPT also provided an example on how to call the stored procedure:

[![ChatGPT - Database](/blog/images/2022/chatgpt-db8.png)](/blog/images/2022/chatgpt-db8.png)

Cool.

As a final example for this post, I used the newly created stored procedure and it works.

[![ChatGPT - Database](/blog/images/2022/chatgpt-db9.png)](/blog/images/2022/chatgpt-db9.png)

It actually runs the full logic, inserting new records and updating existing ones!

[![ChatGPT - Database](/blog/images/2022/chatgpt-db10.png)](/blog/images/2022/chatgpt-db10.png)

Amazing.

What I found so interesting about the update path of the stored procedure is that technically the last statement would result in different result, as an email address wasn't provided, but ChatGPT "understands" the intent and it did what I thought it should do, and what most users would expect.

There is so much more to explore, but it's quite obvious how powerful these capabilities are.

Cheers.


### References

* [Building A Virtual Machine inside ChatGPT](https://www.engraved.blog/building-a-virtual-machine-inside/)
* [ChatGPT](https://chat.openai.com/chat)


### Appendix: T-SQL Commands

If you'd like to try these commands out yourself, here are T-SQL commands in plain text, rather then the screenshots above.



```
Imagine you are a Microsoft SQL Server. I type commands, and you reply with the result, and no other information or descriptions. Just the result. Start with exec xp_cmdshell 'whoami';
```

```
EXEC sp_databases;
```

```
CREATE DATABASE ChatBot;
```

```
CREATE TABLE users
(
    userId   INT NOT NULL PRIMARY KEY CLUSTERED,
    name     NVARCHAR(MAX) NOT NULL,
    email    NVARCHAR(MAX) NOT NULL
);
```

```
INSERT INTO users VALUES (1, 'wuzzi', 'security@wunderwuzzi.net');
INSERT INTO users VALUES (2, 'mallory', 'mallory@example.org');
INSERT INTO users VALUES (3, 'hacker', 'hacker@example.org');
```

```
SELECT * from users;
```

```
Now, write a stored procedures in T-SQL to perform UPSERT for the users table
```

*Results are not deterministic, so you might get different results. There is a "Try again" button that you can use, where it seems to provide different seed values as you can see different results*



