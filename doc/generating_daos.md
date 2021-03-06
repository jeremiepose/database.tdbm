Generating DAOs
===============

Although TDBM can be used without generating DAOs, the easiest way to use it is to generate DAOs using Mouf.
This feature will allow you to cleanly separate the database code (that you will put in DAOs) from the rest of your code.

Getting started
---------------

###The tdbmService instance

During installation, TDBM will generate a "tdbmService" instance.

This instance is already configured, but sould you need to modify the configuration, you need to know that
a `TDBMService` must be connected to a database connection, and to a caching service.

<img src="images/tdbm_service_instance.png" alt="" />

###Setting up the cache service

By default the "NoCache" service is used (this is a dummy cache that does not cache anything). When going
in production mode, you can use a better caching method (like MemcacheCache or APCCache).
TDBM requires a cache in order to store database related data. Indeed TDBM stores in cache the structure of the database,
and relies heavily on declared foreign keys to perform the "smartest" queries. Instead of querying the database for those
foreign keys, it will query them once, and put them in cache. This means that if you modify the database model, you will
need to purge the TDBM cache.

TDBM does not provide itself the caching mechanism. You are free to choose the best one amongst the Mouf components 
implementing the CacheInterface interface. Mouf provides a number of cache implementations:

- `SessionCache`: use the user's cache as a session.  Very easy to set-up, but limited, since the cache is 
  not shared between users.
- `FileCache`: stores data in a file on the server's file system. Pretty efficient.
- `APCCache`: uses the APC caching system. The most efficient way to perform caching, but you will need
  to have the APC extension installed on your system to use it.
- `NoCache`: this is a caching system that... does not cache anything! You should never use it in 
  a production environment, but it can be pretty useful in a development environment when you are frequently
  changing your database model and are not willing to purge the cache each time you make a change.


Generating the DAOs
-------------------

In this chapter, we will see how to generate DAOs and beans, using the `TDBM_Service` instance.
This process is automatically done when you install the package, but should you modify the data model, you will need to 
regenerate the DAOs.
First, go to the "`tdbmService`" instance page (select it from the "View declared instances", or use the full-text search feature).

On the right part of the screen, select the "Generate DAOs" link.

<img src="images/generate_daos.png" alt="" />

On this screen, you can choose the directory that will contain the DAO classes, and the directory that will contain the Beans. Also,
a DAOFactory object (that allows easy access to each DAO) will be generated. Let's just keep the default settings and click the
"Generate DAOs" button.

That's it, we generated all the DAOs for our database. Let's have a closer look at what was generated.

Note: if you are using Eclipse, we strongly recommend you to refresh your project, to load the new classes.

The DAOs structure
------------------

For each table in your database, TDBM will generate a DAO and a bean. The DAO is the object you will use to
query the database. Each row of the database will be mapped to a bean object.

Both DAOs and beans are divided in 2 parts. Let's assume you have a "users" table. TDBM will generate those classes for you:


- `UserDaoBase`: the base class that contains methods to access the "users" table. It is generated by TDBM and you should
  never modify this class.
- `UserDao`: this class extends UserDaoBase. If you have some custom requests, you should perform them in this class. You can
  edit it as TDBM will never overwrite it.
- `UserBaseBean`: the bean mapping the columns of the "users" table. This class contains getters and setters for each and every
  column of the "users" table. It is generated by TDBM and you should
  never modify this class.
- `UserBean`: this class extends UserBaseBean. If you have some custom getters and setters, you should implement them in this class. You can
  edit it as TDBM will never overwrite it.

Let's now have a closer look at the methods that are available in the "UserDao" class:

- `public function create()` : returns a new UserBean object ready to be added in the database.
- `public function save(UserBean $obj)` : saves a UserBean object in database (TDBM can also decide to save the object by
  itself so most of the time, you don't need to call this function explicitly)
- `public function getList()` : returns all users records as an array of "UserBean" objects.
- `public function getById($id, $lazyLoading = false)` : Get a UserBean specified by its ID (its primary key)
- `public function delete($obj, $cascade=false)` : Deletes the UserBean passed in parameter. If $cascade is set to true, it will delete all objects linked to $obj.


The last 2 functions are _protected_. It means they are designed to be used in the UserDao class.


- `protected function getListByFilter($filterBag=null, $orderbyBag=null, $from=null, $limit=null)` : returns a list of
  users based on a filter bag (see the `TDBM_Service` documentation to learn more about filter bags). You can also
  provide an order, and an offset / limit range.
- `protected function getByFilter($filterBag=null)` : this has exactly the same purpose as getListByFilter except
  it returns only 1 bean object instead of a list of bean objects.


