---
layout: post
title: "First look at MongoDB"
date: 2015-06-19 23:06:28 -0700
comments: true
categories: [web, framework, meteorjs, ruby on rails, javascript, mongodb, nosql, introduction, tutorial, embedded references]
---

## The scenario

My (augmented todo-list) website has Categories, and **each Category has a set
of Tasks** (aka *"one-to-many"*).

> Using a relational database, one does not simply store references to all the
> Tasks directly in each Category. With Mongo, one may do exactly that.

After looking at various **"embedded vs references"**
[documents](http://docs.mongodb.org/manual/tutorial/model-referenced-one-to-many-relationships-between-documents/), I have decided to go with the hybrid
model (the second of the three on that linked page), and **embed references to
Tasks in an array in each Category**.

### The console shall reveal the way to accomplish this

My lack of sustained googling didn't turn up any online resources about how to
do it, so here you go.

#### Install MongoDB

After installing MongoDB with Homebrew, it became clear that the configuration
was still incomplete (perhaps because brew lacks root access to the root
directory). StackOverflow told me how to finish. I show you.

```bash
brew install mongodb
sudo mkdir -p /data/db
sudo chmod 755 /data/db
sudo chown -R `id -u` /data/db

# now run
mongod # in one window (server daemon)
mongo  # in another window (interactive console)
```

#### Write the code

Now let's do an example of having a one-to-many relationship between Categories
and Tasks, where in particular, we are storing an array of Task id's in each
Category. In the end we will write a query to get all Tasks belonging to a
Category.

```javascript

/* create the collections */
> db.createCollection("categories")
> db.categories.find().count()
> c = db.categories
> db.createCollection("tasks")
> t = db.tasks

/* insert some elements into them */
> c.insert({name: "first category", tasks: []})
> t.insert({name: "first task"})
> t.insert({name: "second task"})

/* add the references from the coll to the tasks */
> c.update({name: "first category"}, {$addToSet:{"tasks":t.findOne({name: "first task"})._id}})
> c.update({name: "first category"}, {$addToSet:{"tasks":t.findOne({name: "second task"})._id}})
> c.find().pretty()
{
    "_id" : ObjectId("5584fe86e2c6cdd3813233ed"),
    "name" : "first category",
    "tasks" : [
        ObjectId("5584fec9e2c6cdd3813233ee"),
        ObjectId("5584fed0e2c6cdd3813233ef")
    ]
}

/* get all task documents referenced by the "first category"
> t.find({_id: {$in: c.findOne().tasks}})
{ "_id" : ObjectId("5584fec9e2c6cdd3813233ee"), "name" : "first task" }
{ "_id" : ObjectId("5584fed0e2c6cdd3813233ef"), "name" : "second task" }
```
