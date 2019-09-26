---
title: "MongoDB Install"
---
MongoDB is an open-source, document-oriented, NoSQL database management system.
Data is stored in a format very similar to JSON. It has many great features like
database distribution, MapReduce, and user-defined JavaScript functions.
<!--sep-->

## Install
Once again, I'll be using the instructions provided by [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-18-04)
to install MongoDB on my Ubuntu 18.04 workstation.

I should note, the version I'm installing is not maintained by MongoDB, but should
work fine for my purposes. Get more information about installing maintained versions
on the [MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/)
install page.

For me, installation is done in 2 commands:
```bash
sudo apt update
sudo apt install -y mongodb
```

After installation, any time you want to access MongoDB, you'll need to ensure
the MongoDB service is running. Use these commands to start, stop, and check status:
```bash
sudo systemctl start mongodb
sudo systemctl stop mongodb
sudo systemctl status mongodb
```

Then, to access MongoDB from the terminal, just type ```mongo```. Additionally,
you can access MongoDB with several languages including Python, JavaScript,
Java, C++, C#, and more.

## Sample Data
To get up and running, I will be loading the
[Historical Sales and Active Inventory](https://www.kaggle.com/flenderson/sales-analysis/home)
data from [Kaggle Datasets](https://www.kaggle.com/datasets) into MongoDB.

After downloading and extracting the csv file, I was able to create a database,
create a collection, and import the data in the terminal, all with just one command.

```bash
mongoimport --db inventory --collection sales --type csv --headerline --file ./SalesKaggle3.csv
```

For more information about this command, see the mongoimport section in the
[MongoDB manual](https://docs.mongodb.com/manual/reference/program/mongoimport/).

Next, I opened up MongoDB in the terminal to make sure the data was imported.

Here is my terminal input/output showing the newly created database, collection,
and a sample of the data:

![Image of Output]({{ site.baseurl }}/assets/images/posts/mongodb-install/data-sample.png)

## Conclusion
So now I have MongoDB installed with some sample inventory sales data.

Next, I'll be looking over the [MongoDB tutorials](https://docs.mongodb.com/manual/tutorial/),
as I am a little rusty writing these types of queries.
