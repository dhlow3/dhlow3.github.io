---
title: "MySQL Install"
---
MySQL is an extremely popular, open-source database management system. It is
the go-to for many developers wanting to store information using relational
databases. Data is queried using SQL, and it is an excellent candidate for
read/write operations in OLTP. <!--sep-->

## Install
For my home workstation, I use Ubuntu (currently 18.04). Installing MySQL on
Ubuntu is incredibly easy. I just followed the instructions put together by the
folks over at [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-18-04).

This may have been the easiest thing I have ever installed. After 3 commands in
the terminal and a couple of prompts, the installation was complete in about 2
minutes.

```bash
sudo apt update
sudo apt install mysql-server
sudo mysql_secure_installation
```

For a more thorough understanding of the installation process, I highly recommend
going through the [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-18-04)
guide, or finding another guide for your specific OS.

## Sample Data
Next, I want to load some sample data to play around with. I chose to use
the employees database from the
[datacharmer / test_db](https://github.com/datacharmer/test_db) repository.
After cloning the repository, navigate to the test_db folder from the terminal.
Then either run `mysql < employees.sql` or `mysql < employees_partitioned.sql`
depending on which tables you want.

Afterwords, we have a database in the following structure:
<figure>
  <img src="{{ site.baseurl }}/assets/images/posts/mysql-install/employees.png" alt="Image of Employees DB"/>
  <figcaption class="caption">Source - https://github.com/datacharmer/test_db; License - https://creativecommons.org/licenses/by-sa/3.0/; No alterations </figcaption>
</figure>

## Conclusion
So now I have MySQL installed with some sample employee data. Honestly, that was
a lot easier than I expected. In comparison, have you ever tried installing
Microsoft SQL Server along with the Northwind or AdventureWorks database?

Next, I'll be looking through the [MySQL Reference Manual](https://dev.mysql.com/doc/refman/8.0/en)
so that I can start understanding some of the other differences between MySQL
and SQL Server.
