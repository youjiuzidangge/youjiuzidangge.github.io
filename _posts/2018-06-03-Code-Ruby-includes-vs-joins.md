---
layout: post
title:  Preload, Eagerload, Includes and Joins
#时间配置
date:   2018-06-03 05:30:00 +0800
#大类配置
categories: Ruby
#小类配置
tag: Rails
---

* content
{:toc}

找到两篇关于 includes 和 joins 不错的文章，现转载如下：

[Preload, Eagerload, Includes and Joins](https://blog.bigbinary.com/2013/07/01/preload-vs-eager-load-vs-joins-vs-includes.html)

[Article
Includes vs Joins in Rails: When and where?](http://tomdallimore.com/blog/includes-vs-joins-in-rails-when-and-where/)

个人总结
-----------------------------------------------
1. includes 和 joins 的一个重要区别在于，includes 是热加载， 而 joins 是懒加载，即 includes 会把包含与被包含的表都加载到内存，使用级联查询的时候，直接在内存中查询即可，不用再次连接数据库查询。但是 joins 不会主动把两张表都加载到数据库，使用级联查询的时候会再次查询数据库。因此级联调用的时候，使用 joins 是很费性能的。

2. 当有条件语句的时候，includes 会自动转换为 left outer join，而 joins 默认内连接。


原文如下：

第一篇
-----------------------------------------------

Rails provides four different ways to load association data. In this blog we are going to look at each of those.

### Preload

Preload loads the association data in a separate query.
~~~ ruby
User.preload(:posts).to_a

# =>
SELECT "users".* FROM "users"
SELECT "posts".* FROM "posts"  WHERE "posts"."user_id" IN (1)
~~~

This is how includes loads data in the default case.

Since preload always generates two sql we can’t use posts table in where condition. Following query will result in an error.

~~~ ruby
User.preload(:posts).where("posts.desc='ruby is awesome'")

# =>
SQLite3::SQLException: no such column: posts.desc:
SELECT "users".* FROM "users"  WHERE (posts.desc='ruby is awesome')
~~~

With preload where clauses can be applied.

~~~ ruby
User.preload(:posts).where("users.name='Neeraj'")

# =>
SELECT "users".* FROM "users"  WHERE (users.name='Neeraj')
SELECT "posts".* FROM "posts"  WHERE "posts"."user_id" IN (3)
~~~

### Includes

Includes loads the association data in a separate query just like preload.

However it is smarter than preload. Above we saw that preload failed for query User.preload(:posts).where("posts.desc='ruby is awesome'"). Let’s try same with includes.

~~~ ruby
User.includes(:posts).where('posts.desc = "ruby is awesome"').to_a

# =>
SELECT "users"."id" AS t0_r0, "users"."name" AS t0_r1, "posts"."id" AS t1_r0,
       "posts"."title" AS t1_r1,
       "posts"."user_id" AS t1_r2, "posts"."desc" AS t1_r3
FROM "users" LEFT OUTER JOIN "posts" ON "posts"."user_id" = "users"."id"
WHERE (posts.desc = "ruby is awesome")
~~~

As you can see includes switches from using two separate queries to creating a single LEFT OUTER JOIN to get the data. And it also applied the supplied condition.

So includes changes from two queries to a single query in some cases. By default for a simple case it will use two queries. Let’s say that for some reason you want to force a simple includes case to use a single query instead of two. Use references to achieve that.

~~~ ruby
User.includes(:posts).references(:posts).to_a

# =>
SELECT "users"."id" AS t0_r0, "users"."name" AS t0_r1, "posts"."id" AS t1_r0,
       "posts"."title" AS t1_r1,
       "posts"."user_id" AS t1_r2, "posts"."desc" AS t1_r3
FROM "users" LEFT OUTER JOIN "posts" ON "posts"."user_id" = "users"."id"
~~~

In the above case a single query was done.

### Eager load

eager loading loads all association in a single query using LEFT OUTER JOIN.

~~~ ruby
User.eager_load(:posts).to_a

# =>
SELECT "users"."id" AS t0_r0, "users"."name" AS t0_r1, "posts"."id" AS t1_r0,
       "posts"."title" AS t1_r1, "posts"."user_id" AS t1_r2, "posts"."desc" AS t1_r3
FROM "users" LEFT OUTER JOIN "posts" ON "posts"."user_id" = "users"."id"
~~~

This is exactly what includes does when it is forced to make a single query when where or order clause is using an attribute from posts table.

### Joins

Joins brings association data using inner join.

~~~ ruby
User.joins(:posts)

# =>
SELECT "users".* FROM "users" INNER JOIN "posts" ON "posts"."user_id" = "users"."id"
~~~

In the above case no posts data is selected. Above query can also produce duplicate result. To see it let’s create some sample data.

~~~ ruby
def self.setup
  User.delete_all
  Post.delete_all

  u = User.create name: 'Neeraj'
  u.posts.create! title: 'ruby', desc: 'ruby is awesome'
  u.posts.create! title: 'rails', desc: 'rails is awesome'
  u.posts.create! title: 'JavaScript', desc: 'JavaScript is awesome'

  u = User.create name: 'Neil'
  u.posts.create! title: 'JavaScript', desc: 'Javascript is awesome'

  u = User.create name: 'Trisha'
end
~~~

With the above sample data if we execute User.joins(:posts) then this is the result we get

~~~ ruby
#<User id: 9, name: "Neeraj">
#<User id: 9, name: "Neeraj">
#<User id: 9, name: "Neeraj">
#<User id: 10, name: "Neil">
~~~

We can avoid the duplication by using distinct .

~~~ ruby
User.joins(:posts).select('distinct users.*').to_a
~~~

Also if we want to make use of attributes from posts table then we need to select them.

~~~ ruby
records = User.joins(:posts).select('distinct users.*, posts.title as posts_title').to_a
records.each do |user|
  puts user.name
  puts user.posts_title
end
~~~

Note that using joins means if you use user.posts then another query will be performed.

第二篇
-------------------------------------------------------------------

For the past few months I’ve been hiding away in a cave and working intensely on a not-so-secret project, Trado. So I thought I’d reach out once more to my fellow interwebbers, and share some knowledge I’ve learned on my journey so far.

Now with any e-commerce platform there is bound to be a lot of database relations and models relying on each other for, you guessed it: data, and with many relations come many performance aches and pains. So like any good developer you turn to the Ruby on Rails documentation and stumble upon two very similar yet very promising Active Record methods: includes and joins.

***What is the difference between includes and joins?***

The most important concept to understand when using includes and joins is they both have their optimal use cases. Includes uses eager loading whereas joins uses lazy loading, both of which are powerful but can easily be abused to reduce or overkill performance.

If we first take a look at the Ruby on Rails documentation, the most important point made in the description of the includes method is:

With includes, Active Record ensures that all of the specified associations are loaded using the minimum possible number of queries.

In other words, when querying a table for data with an associated table, both tables are loaded into memory which in turn reduce the amount of database queries required to retrieve any associated data. In the example below we are retrieving all companies which have an associated active Person record:

~~~ ruby
@companies = Company.includes(:persons).where(:persons => { active: true } ).all

@companies.each do |company|
  company.person.name
end
 
@companies = Company.includes(:persons).where(:persons => { active: true } ).all
 
@companies.each do |company|
  company.person.name
end
~~~
 
When iterating through each of the companies and displaying the persons name, we would normally have to retrieve the persons name with a separate database query each time. However, when using the includes method, it has already eagerly loaded the associated person table, so this block only required a single query. Awesome, right?!

So what happens if I want to retrieve all companies with an active associated Person record, but I don’t want to display any data from the Person table? It’s starting to seem a tad overkill loading the associated table…well that’s where the joins method starts to shine!

If we use the above example again, we can start to see how easily people can become confused between the includes and joins method, when very little has changed:

~~~ ruby
@companies = Company.joins(:persons).where(:persons => { active: true } ).all

@companies.each do |company| 
     company.name 
end
 
@companies = Company.joins(:persons).where(:persons => { active: true } ).all
 
@companies.each do |company| 
  company.name 
end
~~~
 
Visually the only difference is replacing the includes method call with joins, however under the hood there is a lot more going on. The joins method lazy loads the database query by utilising the associated table, but only loading the Company table into memory as the associated Person table is not required.Therefore we are not loading redundant data into memory needlessly; although if we wanted to use the Person table data later on from the same array variable, it would require further database queries.

***I’m not convinced, I need some stats…stat!***

Recently I fell victim to not using the awesome power behind the includes method in my Trado codebase. I noticed a severe performance leak when monitoring the database queries in my local instance server logs, which is a habit I would advise starting.

The following code in my index method was producing an abnormal amount of database queries, as seen below:

~~~ ruby
def index
   @shippings = Shipping.active.all
   respond_to do |format|
      format.html # index.html.erb
      format.json { render json: @shippings }
   end
 end

def index
  @shippings = Shipping.active.all
  respond_to do |format|
    format.html # index.html.erb
    format.json { render json: @shippings }
  end
end
~~~ 

<p><img style="width:100%" src="{{ '/styles/images/2018/06/includes_vs_joins_1.png' | prepend: site.baseurl }}" /></p> 

<p><img src="{{ '/styles/images/2018/06/includes_vs_joins_2.png' | prepend: site.baseurl }}" /></p> 

(The database query list was so big I couldn’t fit it in the screenshot!)

As you can see, for every row in the table, it was making two database queries to grab data for the associated zones and tiers tables. When scaled up this starts to become a heavy load on resource with an Active Record loading time of 265.2ms. So in light of preserving scalability and performance in my application, I modified the index method to take of advantage of the includes method for the zones and tiers table associations:

~~~ ruby
def index
  @shippings = Shipping.active.includes(:zones, :tiers).all

  respond_to do |format|
    format.html # index.html.erb
    format.json { render json: @shippings }
  end
 end
 
def index
  @shippings = Shipping.active.includes(:zones, :tiers).all

  respond_to do |format|
    format.html # index.html.erb
    format.json { render json: @shippings }
  end
end
~~~

<p><img style="width:100%" src="{{ '/styles/images/2018/06/includes_vs_joins_3.png' | prepend: site.baseurl }}" /></p> 

<p><img src="{{ '/styles/images/2018/06/includes_vs_joins_4.png' | prepend: site.baseurl }}" /></p> 

You can quickly see here that the number of database queries has been reduced to an optimised number of just 5, which in turn drastically reduces the Active Record loading time to just 2.8ms – that’s a 99% reduction!
