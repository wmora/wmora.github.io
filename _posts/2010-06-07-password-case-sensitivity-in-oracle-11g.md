--- 
title: Password Case Sensitivity in Oracle 11g
date: "2010-06-07T22:21:00.000-03:00"
author: William Mora
tags: 
- DBA
- sec_case_sensitive_logon
- Oracle 11g
- password case sensitivity
- Oracle
- SQL
redirect_from: 
- /2010/06/password-case-sensitivity-in-oracle-11g.html
---

Oracle 11g now sets the password case sensitivity setting to `TRUE` by default. If you are running any application that connects to an earlier version of an Oracle database, chances are you are sending the connect string in all caps within the application (ex: `USER/USER@ORCL11G` when it really should be `user/user@orcl11g`). Make sure you are aware of this when developing your app and if the case is that you want to have case sensitive passwords, use double quotes to enclose the password string (ex: `USER/"user"@ORCL11G`). To disable password case sensitivity, just change the value of the system parameter `sec_case_sensitive_logon`:

```bash
alter system set sec_case_sensitive_logon = FALSE;
```

Where `FALSE` disables password case sensitivity and `TRUE` enables it.

Cheers!