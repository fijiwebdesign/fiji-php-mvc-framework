Fiji PHP MVC Framework
======================

A PHP MVC framework that allows the developer to focus soley on the functionality of their application. True rapid application development. 

Storage Agnostic Models
-----------------------

Remove the data storage specifics completely from your application code. 
You only work with Models that are just classes with public properties.
You don't need to define SQL tables or SQL relationships. Just define references in your model. 
SQL Relationships are built for you when you save your models. 

In fact, forget that you're working with SQL because the storage can be changed to an key/value storage such as Redis and your Models behave just the same way. 

You don't need to define a model class in fact. You can create it dynamically. But it is good practice to create models so you have a self documented normalized data structure. 

Example of a simple Model with references:

```
namespace Fiji\App\Model;

use Fiji\Factory; // only hard dependency

/**
 * Simple message model
 */
class Message
{

  /**
   * @var {String} Message title
   */
  public $title;
  
  /**
   * @var {String} Message html body
   */
  public $body;
  
  /**
   * @var {Fiji\App\Model\User} Reference to message sender Fiji\App\Model\User
   */
  protected $Sender;
  
  /**
   * @var {Fiji\App\ModelCollection} References to message recipients Fiji\App\Model\User
   */
  protected $Recipients;
  
  public function __construct()
  {
    // make "Sender" a reference to a model Fiji\App\Model\User
    $this->Sender = Factory::createModel('User');
    // make "Recipients" a reference to a collection Fiji\App\ModelCollection of models Fiji\App\Model\User
    $this->Recipients = Factory::createModelCollection('User');
  }
  
}
```

Now create a Model, add some data and save your Model: 

```
// our message model
$Message = Factory::createModel('Message');

$Message->title = "Hello Team";
$Message->body = "Meeting at Conference room 5 2pm!";
$Message->Sender = Factory::createModel('User')->findBySession(); // logged in User Model

// get all users in the logged in users team as a ModelCollection
$Message->Recipients = Factory::createModelCollection('Team')
  ->findById( $Message->Sender->Team->id )
  
$id = $Message->save(); // 1
```

Now in your storage you have the Message you just created, with the Sender being a reference to the logged in User Model and the Recipients being a reference to the ModelCollection of Users in the Sender's team. 

If you have an SQL storage then these would be a table called Message with a new entry. 
A sender_id referencing the Sender entry in the Users table in a one to many relation. 
A separate table called message_user which maps the Message to the list of Users in a Many to Many relation. 

This persists the state of your $Message object so the next time you retrieve it all the references are lazy loaded when you ask for them. 

```
// our message model
$Message = Factory::createModel('Message')->find('Hello Team');

echo $Message->title; // "Hello Team";
echo $Message->body; // "Meeting at Conference room 5 2pm!";
echo $Message->Sender->name; // My Name
foreach ($Message->Recipients as $User) {
  echo $User->name; 
}
```

Never require() a file
----------------------

Using PHP's autoload, your classes are loaded lazily on their first invoke. You should never need to use require(). 
This removes the need to manage dependencies yourself.

Simple Controllers and Create Read Update Delete (CRUD)
-------------------------------------------------------

```
namespace app/Controller;

class Gallery extends Controller
{

  public function __construct()
  {
    // access control
    Factory::getAccessControl()->allow(array(
      'owner',
      'member',
      'admin'
      ), function() {
        Factory::getApp()->redirect('/', 'You do not have access to view the gallery');
      });
  }
  
  /**
   * Default view. List of Galleries
   */
  public function index()
  {
    // retrieves all the galleries in storage
    $GalleryList = Factory::createModelCollection('gallery');
    $GalleryList->find();

    $this->View->set('GalleryList ', $GalleryList );
    $this->View->display('list');
  }

  /**
   * Create a gallery
   */
  public function create()
  {
    $model = Factory::createModel('gallery');

    $model->title = 'Cool Gallery';
    $model->description = 'Nice Description';

    // no need to define the structure. Storage automatically creates it. 
    $id = $model->save();

    Factory::getApp()->redirect('gallery/view/' . $id, 'Created a Gallery!');

  }

  /**
   * View a specific gallery
   */
  public function view()
  {
    $id = Factory::getRequest()->getInt('id');

    $model = Factory::createModel('gallery');
    $model->findById($id);
    
    $view = Factory::createView('gallery');
    $view->set('model', $model);
    $view->display();

  }

  /**
   * Edit a gallery
   */
  public function edit()
  { 
    $id = Factory::getRequest()->getInt('id');

    $model = Factory::createModel('gallery');
    $model->findById($id);

    $model->title = 'New Title';
    $model->save();

    Factory::getApp()->redirect('gallery/view/' . $id);

  }

}

```


Main aims
---------

* Every piece of the MVC layer dependencies is an interface. ie. can be overridden or replaced with no effect on other parts.
* Developer does not care about storage or caching. ie. use storage agnostic models.
* Developer should not need to define schema/structure of storage/models up front. Models build their own structure in storage on saving. (Don't worry, you can tweak it later)
* Developer should never worry about where classes are (never require() a file)
* UI should not be a concern to developer so they can quickly prototype (eg. Bootstrap)
* Developer should be able to write a simple application in 5 minutes (see example)

###Current achieved goals###
Most of the MVC layer can be replaced or overridden by your application.

All classes are namespaced (PHP 5.3+) and loaded via Factory pattern. Currently this is via static methods but may change for easier testing.

Storage is abstracted away from models allowing models to be free of storage specific code. Any storage is treated as a service. The Services layer (links Models to storage) is interfaced so that you can plug in different DataProvider classes to handle a specific storage service.

Caching is considered a services as well, allowing priority storage and retrieval from cache and fallback to another storage service. Services can be memory, REST, MySQL, memcached or any other.


Todo: 
----

More features soon.
