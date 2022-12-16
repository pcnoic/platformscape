---
title: A story on indexing and why it is important for modern applications
description: Capitalize on low hanging fruit to improve the speed of modern web applications.
author: Christos Alexiou
date: 2022-02-16
tags:
  - sql
  - databases 
  - webdev
---

### Modern web applications need performance

The majority of modern web applications, if not all of them, are utilising **some kind of data management** and **storing solution**. Most of these applications are relying on modern database systems _such as MySQL or MongoDB_ to name a few.

<img src="https://community-cdn-digitalocean-com.global.ssl.fastly.net/assets/tutorials/images/large/Database-Mostov_v4.1_twitter-_-facebook.png?1550071669" />

As **more and more** users are **moving their workloads online**, either in the form of performing simple word processing and light image editing or in heavily CPU and data retrieval bound applications such as BPM solutions for large corporations, **_"speed and power"_** (sic) are indispensable. 

<img src="https://i.imgur.com/hO9dyio.jpg" width="600px"/>

Modern database solutions have **a vast amount of perks** for even a seasoned web developer to utilize in order to make her **application fly**. 

### Utilizing indexes for faster data retrieval

Indexing stuff **for the sake of finding it faster** is not a new kid in the block. For example, _scholars in the great Library of Alexandria in Egypt_ have been **using a lookup system** for quickly finding papers that were interesting to them as back as 280BC. That system was **heavily relying on indexing** and was designed by Calimachus, a poet, critic and scholar of the time. The name of the system was **"Pinakes"**, translating in English as "Tables". 

<img src="https://s1.dmcdn.net/v/MiLjC1Q797KywoQ5k/x1080" />

#### Is indexing still relevant today ?

Let's start **our short journey** in indexing by explaining _what indexing actually is_ in a context **that may actually be useful for modern day data retrieval solutions developers** and not for ancient Alexandria scholars. 

Typical database systems are storing data on slow disks where I/O bottlenecks are almost innevitable. Even with modern solid state drives the speed of retrieving raw data out of the disk has not given us yet _that "whoosh" moment_. 

<img src="https://image.shutterstock.com/image-vector/cartoon-comic-book-sonic-boom-260nw-412369423.jpg" height="400px" width="600px"/>

In-memory database solutions are becoming a thing today with Redis and other caching mechanisms and message brokers but they are still **a long way for being a fully fledged database system** that we can actually store all of our data - largely due to hardware limitations and cost. 

Even if we utilize such caching solutions like Redis, there is a certainty that on some occassions the application will have to query the database and then, the database retrieve data that they are stored on disk. So, I/O lag **is a thing** and we have to make up for that by **making our database searches more performant**.  

Indexes are an **incredibly powerful feature of relational databases.** In fact, using indexes properly can often improve the performance of your queries **by an order of magnitude.** 

Indexes allow you to “ask questions” **without having to search the entire table on disk.** They’re really useful for answering “needle in a haystack” type of questions, where you’d **otherwise have to look through many rows** to pick out only the one that you actually need.

An index is a data structure that’s stored persistently on disk, and is typically implemented as either:

- "B-Tree" -- fast for all sorts of comparisons (>, <, ==)

<img src="https://cdn-images-1.medium.com/max/1600/1*pE4SEz7CprzFd7Zww-axfQ.jpeg"/>

- "Hash Table" -- fast only for equality comparison (==) 

<img src="https://lh3.googleusercontent.com/proxy/vq_qcab7UQqWetwEtMMNbFFiWZNKKWBg6uC50HFTf8KYbMVfxH4s3_4mFt4somQRL6cK0H-P994uEjoCFlijhrtctvy2Ujeore50BYqRYhKFBqfyjnIwtXi6kHx1y5tHT1jLS6lAD-xx9xNFLa1VQtJ3AfQV9DU0-VU"/>

**Without an index**, performing a SELECT ... FROM table WHERE ...; statement requires **the DBMS to search through every single row** in the table to see if it matches that exact WHERE clause. If you have more than a few hundreds rows in a table - _and most modern web applications have millions if not billions_ - this takes a non-negligible amount of time.

<img src="https://d2r55xnwy6nx47.cloudfront.net/uploads/2018/09/Hay_1300Lede.jpg" height="300px" width="600px"/> 


**But** by **adding an index to a column that you frequently run lookups on**, you can decrease this amount of time required. Instead of having to scan n rows (where n is the total number of rows in the database), the database can return your query by looking at either log(n) rows (if indexes are implemented as a b-tree) or **simply one row (if indexes are implemented as a hash table).** 

Once you’ve set an index on a particular column, **you don’t need to do anything to make subsequent queries use it.** The query planner in the DBMS will use that index if it needs to.

<img src="https://media.makeameme.org/created/yeah-baby-yeah-nse0oq.jpg" height="400px" width="600px" />

#### Is it all that sugary and sweet ?

What was mentioned above are the stellar reasons of using indexes in your relational database. **But**, _as with everything in this world_, there are a few downsides to be aware of. Since the index is stored as a separate data structure, building too many indexes will result in additional space on disk.

One other thing to keep in mind when using indexes is **the overhead of index creation and maintenance.** 

Whenever a new row is added to the table, **the index’s data structure must also be updated** so that the index remains accurate.

Sacrificing a bit of performance on writes to get much better performance on reads seems a good deal for me.

Thanks for your time,
stay safe and **sane**. 
