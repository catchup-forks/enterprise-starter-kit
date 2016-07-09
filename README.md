## Installing

### Acquire a copy
There are multiple ways to acquire a copy of L51ESK. You can download a ZIP archive, clone the project and finally fork 
your own repository then clone it.

#### Option 1: Download
To download a ZIP archive, simply got to the main repository page at of 
[L51ESK](https://github.com/sroutier/laravel-5.1-enterprise-starter-kit) and click on the "Download ZIP" button. 

#### Option 2: Clone
Simply clone this repository on your machine using the command:

```
git clone https://github.com/sroutier/laravel-5.1-enterprise-starter-kit.git l51esk
```

#### Option 3: Fork & Clone
Finally the recommended method would be to first fork this repository then clone your own repository. Follow this guide
to learn how to [fork an existing repository](https://help.github.com/articles/fork-a-repo/). Then use a command 
similar to this to clone your own repository:

```
git clone https://github.com/YOUR-NAME/laravel-5.1-enterprise-starter-kit.git l51esk
```

### Homestead (optional)
For development nothing is better than [Homestead](http://laravel.com/docs/5.1/homestead), I would highly recommend 
using it. 

If you choose to use Homestead, you will have to edit the *Homestead.yaml* config file, edit set the folder mappings and
site mappings. Additionally I like to set the *APP_ENV* variable to *development*. 

You can edit the config file with
```
homestead edit
```

Here is an example of what a *Homestead.yaml* might look like for this project:
```
---
ip: "192.168.10.10"
memory: 2048
cpus: 1
provider: virtualbox

authorize: ~/.ssh/id_rsa.pub

keys:
    - ~/.ssh/id_rsa

folders:
    - map: ~/projects
      to: /home/vagrant/projects

sites:
    - map: l51esk.dev
      to: /home/vagrant/projects/shared/l51esk/public

databases:
    - homestead

variables:
    - key: APP_ENV
      value: development
```

Next you will have to add an entry in your host file.
```
echo "192.168.10.10   l51esk-demo.dev" | sudo tee -a /etc/hosts
```

Finally the last step is to provision the Homestead VM.
```
homestead provision
```

Once provisioned, you can ssh into the running homestead VM, and change directory to the root of the project.
```
homestead ssh
cd projects/shared/l51esk
```


### Fetch dependencies

#### Composer
This project includes the file 'composer.lock'. The purpose of the 'composer.lock' file is to lock the version of the various
packages needed by a project as this one. By using the included lock file you are sure to use the same version of those 
packages as were used during the design. 

Fetch all dependencies using *composer* by issuing one of the following commands depending on which environment you are 
trying to configure:

For a development environment use:
```
composer install
```

For a production environment use:
```
composer install --no-dev
```


**_NOTE:_** To bypass the lock on package versions run the command *composer update*.

**_NOTE:_** On a production server, prior to running the *composer install* command, you will want to deploy a copy of your file 
*composer.lock* from your development server, to guarantee that the exact version of the various 
packages that you have developed on and tested gets installed. Never run the *composer update* 
command on a production server.


#### Node.js
Fetch all dependencies for Node.js using *npm* by using the following command:

```
npm install
```

**_NOTE:_** If the *npm install* command fails or hangs, check the tip on *Node.js* in the [Troubleshooting](#troubleshooting) section.
### Basic configuration

#### Create your *.env* file
Create a *.env* file from the *.env.example* supplied.

For a Development environment use:
```
cp .env.example-dev .env
```

For a other environments, such as QA and Production use:
```
cp .env.example-qa .env
```

**_NOTE:_** Do not use the environment file for the development environment in any other environment as this will cause a lot of failures in the bootstrap aand kernel part of the application due to some dependencies being dynamically injected for the development environment only.

### Basic configuration

#### Generate application key
Generate the unique application key:
````
./artisan key:generate
````

#### Grant permissions to some directories. 
Either grant permission to all users, or just to the Web Server user by making it a member of the group that owns these folders.
````
chmod -R ugo+rw storage/
chmod -R ugo+rw bootstrap/cache/
````

#### Review default settings
Review and edit the *.env* file and all the files under */config*. Paying particular attention to */config/app.php* and 
*/config/database.php*. More details can be found in the [Configuring](#Configuring) and 
[Documentation](#Documentation) section below.

**_Note:_** By default all settings are set for a development environment, You will want to review and tweak before deploying to
Prod. 


### Migration
After having configured your database settings, you will want to build the database.
 
If you kept the default database settings your will first have to initialize the SQLite file
```
touch storage/database.sqlite
```

To run the migration scripts run this command
 ```
 ./artisan migrate
 ```
 
 To seed the database run the command below, note that in the development environment a few extra user and permissions
 are created.
 ```
 ./artisan db:seed
 ```

### First login and test
You should now be able to launch a Web browser and see your new Web application. To log in using the *root* account
the default password is *Password1*. Please change it ASAP.


## Configuration

### Authentication & Authorization
During the installation the database seeder scripts created a few things to help get started:

* The super user *root*.
* The roles *admins* and *users*.
* The permissions *basic-authenticated*, *guest-only* & *open-to-all*.

Additionally, on a development environment a few more test users and permissions would have been created.

The relationship between users, roles & permissions is relatively simple: users are assigned roles (many-to-many)
and roles are assigned permissions (many-to-many). This enables a simple role based permission assignment system.
For more information please refer to the documentation of the [zizaco/entrust](https://github.com/zizaco/entrust) 
package. Also while not recommended by autorization best practices, user based permission assignment is 
supported. Permissions can be directly assigned to individual users if needed.

Where things get a little more interesting is the addition of the *Route* model. Not to be confused with the 
[Route](http://laravel.com/api/5.1/Illuminate/Routing/Route.html) from Laravel, the *Route* model is still 
closely related to Laravel routes. In fact all *Routes* can be automatically built by inspecting the Web 
site routing table. Initially if you navigate to the *Admin > Security > Routes* page, you will be 
greeted with an empty table. To automatically load all routes defined within the Web site simply 
click on the *load* button. After a short delay, the page will reload and you will be able to 
assign any of the defined permission to each route.
 
Once *Routes* are assigned a single *Permission* each and permissions are assigned to *Roles* and finally *Users* are 
granted *Roles*, then the matching *AuthorizeRoute* middleware can authorize or block access to all routes for both 
guest and authenticated users. This feature will probably not be used by any site user or even administrators,
but by the site developer(s). In fact one of the first things that I would recommend is to restrict all routes
to the *Route* management pages to a permission given to developers only. What this feature does is make 
the authorization process very flexible, powerful and easy to change on the fly.
  
Some important hard-set rules to note are:

* Except when specifically stated otherwise below, routes, permissions, roles and users can be disabled.
* Routes
    * If a route is either not defined or not assigned any permission, it will not be accessible, except to the root 
    user or any user granted the admins role.
    * Routes to the controllers *AuthController* and *PasswordController* are not restricted by the *AuthorizeRoute* 
    middleware. Otherwise users could not log in or reset their passwords.
    * A route assigned the permission *open-to-all* will be authorize for all users, authenticated or not.
    * A route assigned the permission *guest-only* will only be authorized for guest users, not for authenticated ones.
    * A route assigned the permission *basic-authenticated* will be authorized for any user that has logged into the
    system. No other permission will be required. But the same route will be denied for guest users.
    * Failure to be authorized to access a route will redirect to the error page 403 (access denied).
    * When loading *Routes* from the Web site routing table, all routes to the *AuthController* and *PasswordController*
    will be skipped over. Also any route to the *DebugBar* will be skipped. If required they can be added by creating 
    a route manually.
    * Disabling a route prevents the route from being accessible or authorized.

* Permissions
    * The permissions *guest-only* and *basic-authenticated* cannot be edited or deleted.
    * A permission that is assigned to a *Route* or a *Role* cannot be deleted.
    * The permission *guest-only* cannot be assigned to any role. It is reserved for guest or un-authenticated users.
    * The permission *basic-authenticated* is forced onto every role.
    * The permission assignment for the role *admins* cannot be changed.
    * Disabling a permission prevents it from granting access to any route assigned to that permissions.

* Roles
    * The roles *admins* and *users* cannot be edited or deleted
    * The role *users* is force onto every user.
    * Disabling a role prevent prevents the users assigned this role from getting the abilities of that role.

* Users
    * The user *root* and any user with the *admin* role are not restricted by the *AuthorizeRoute* middleware. They 
    can go anywhere they want, even to routes that are either disabled or not defined.
    * If a user is disabled while he is logged into and using the Web site, he will get automatically logged out the 
    next time he tries to access a route protected by the *AuthorizeRoute* middleware.
    * The user *root* cannot be edited or deleted.
    * A user cannot disable or delete his own currently logged in user.

### LDAP/AD authentication.
To enable the optional LDAP/AD authentication module, set the *LDAP_ENABLED* variable to *true* in the *.env* file as shown 
below:
````
LDAP_ENABLED=true
````
By default the LDAP/AD authentication module is set to off or false, as it requires some extra configuration on your part.
For more information on how to configure the module, refer to documentation of the underlying package at 
[sroutier/eloquent-ldap](https://github.com/sroutier/eloquent-ldap). Additionally, every option is explained in the config file 
*/config/eloquent-ldap.php*.

### Menu system
The menus system can be configured with the menu editor included in the administration section under the security group.
To edit an existing menu item simply click on it in the menu tree on the left, and edit the entry details, on the right, 
once loaded. Edit the entry then click on the *Save* button.
To delete an existing menu item simply select it on the menu tree, then once loaded click on the *Delete* button.
To create a new menu item, first click on the *Clear* button, if you previously had an existing menu entry selected, 
then fill in the form details following these guidelines:

* *Name*: The internal and unique name used as a reference. Required field.
* *Label*: The label shown to the users.
* *Position*: A number used to sort the menu item amongst it siblings. In the event that two items have the same 
position value the items are sorted alphabetically. The position numbers do not have to be sequential, gaps in the 
numbering can created to allow for future insertion, or to ensure that an item or branch is placed last, as is the 
case of the ``Admin`` branch.
* *Icon*: The class of a Font-Awesome icon, with the optional colour qualifier. For example ``fa fa-bolt`` or 
``fa fa-warning fa-colour-orange``
* *Separator*: Checkbox that indicates if this item is to be a separator. When enabled, the label is not shown so a 
good label to use is a few dashes (ie: "----"). Also the item when rendered will not have a URL.
* *URL*: The URL field is best used to create an menu entry pointing to an external resource, otherwise when rendered
the URL should be generated from the associated *Route*.
* *Enabled*: Allows to enable or disable any entry. Any disabled entry will not be rendered. Useful to temporally 
disable an entire branch.
* *Parent*: List all other menu entries, except separators, to select, as the name indicates the parent of the current 
entry. This allows setting the hierarchy.
* *Route*: Associate the current menu entry with a Application Route. This will also give the menu entry both a URL 
and a Permission.
* *Permission*: If the item is not associated with a *Route*, a specific permission can be selected. Best used to secure
access to external resources when specified with the *URL* field.

### Walled garden.
To enable the optional walled garden mode simply set the *WALLED_GARDEN* variable to *true* in the *.env* file as shown 
below:
````
WALLED_GARDEN_ENABLED=true
````
By default the walled garden mode is set to off or false. When enabled all guest or un-authenticated users will be 
redirected to the login page.

### Themes
To change the default theme, set the *DEFAULT_THEME* variable in the *.env* file:
````
DEFAULT_THEME=red
````
L51ESK comes with 3 themes: default, green and red.
Both the red and green themes inherit much of there look from the default theme which is mostly blue and based on the 
look of the [almasaeed2010/AdminLTE](https://github.com/almasaeed2010/AdminLTE) Web template.
For more details on how to configure and develop your own themes refer to the documentation of the 
[yaapis/Theme](https://github.com/yaapis/Theme) package.

### Audit log
To enable the optional audit log simply set the *AUDIT_ENABLED* variable to *true* in the *.env* file as shown 
below:
````
AUDIT_ENABLED=true
````
By default the audit log is disabled. When enabled some user actions can log an entry in the application audit log by calling the static function *Audit::log()* as demonstrated below:
````
// Create simple audit log entry
Audit::log(Auth::user()->id, 'User', 'Access list of users.');
````
The full parameter list for the *log()* function is:
````
log($user_id, $category, $message, Array $attributes = null, $data_parser = null, $replay_route = null)
````
Parameter info:
* *$user_id:* The id of the user that triggered the action, can be null when a user is not authenticated.
* *$category:* Free text to group log entries, later we should be able to filter and search against the category.
* *$message:* Free text containing the main message.
* *$attributes:* An array containing any additional data that should either be displayed on the *show* page or to be used by the replay action.
* *$data_parser:* The full name, including namespace and class name of the function to call to parse the data array store in *$attribute*.
* *$replay_route:* The name of the Laravel route that will be called to replay this action.

#### Replay action
The replay action feature, as the name suggest, allows to replay or repeat an action that was previously logged from the audit log. For the replay action to be available both the *attributes* and the *replay_route* parameters must be specified.
Perhaps the best and easiest way to understand how it function is to follow a concrete example. Below I will describe how the replay action is used in the case of a user edit.

##### Creating a replay-able audit log entry
1. The operator (human) click on the link to edit the entry of a user, say ID #3, the URL would look something like this *http://l51esk/admin/users/3/edit*.
2. The controller *UsersController* and it's function *edit* are invoked. The *edit* function prepares the data that will be displayed and edited then pass it all the the view *admin.users.edit*. Note that in the *edit* function an audit log entry is created stating that the operator initiated the edition of the user. This is just a simple audit log entry that does not save any *attributes* or sets the *replay_route*, it is there simply for audit purposes.
3. The view is built and returned to the operator to see in his browser.
4. The operator makes the changes that are required and submits the form.
5. The controller *UsersController* and it's function *update* are invoked. As expected *update* function will capture the data and update the user's record in the database, but it will also create an audit log entry that can be replayed. Here are the steps required in details:
  1. Capture the posted attributes from the request: 
  
     ```$attributes = $request->all();```
  2. Add any data that would be needed by the replay function, here we want to make sure to add the id of the user since it was passed as a separate parameter to the *update* function:
  
     ````$replayAtt["id"] = $id;````
  3. Finally create the audit log entry: 
  
     ````
     Audit::log( Auth::user()->id, 'User', "Edit user: $user->username",
            $replayAtt, "App\Http\Controllers\UsersController::ParseUpdateAuditLog", "admin.users.replay-edit" );
     ````

That is it, you are done, a replay-able audit log entry has been created. Note the 4th and 6th parameters. 
The 4th parameter (*replayAtt*) is the array of attributes that will be filtered to remove the session token and passwords, then converted into JSON and stored in the *data* column.
The 6th parameter (*admin.users.replay-edit*) is the name of the Laravel route that will be invoked when a replay action is requested. 
The 5th parameter is the fully qualified function name that will be used to parse the *data* field. That will be described in a section below on displaying entry details and custom rendering partials.

##### Triggering a replay-able entry
Following the example above, here is a description of how triggering a replay-able action functions:

1. The operator (human) access the audit log page at: *http://l51esk/admin/audit*
2. Locates the replay-able entry by it's spinning recycling icon in the action column, and click on it.
3. The *replay* function of the *AuditsController* controller is invoked. The *replay* function locates the audit log entry from the id passed in, and redirect to the Laravel route that is store in the *replay_route*. In this case it is *admin.users.replay-edit*.
4. The *admin.users.replay-edit* route triggers the function *replayEdit* in the *UsersController* controller, as defined in the *app\Http\routes.php* file.
5. The *replayEdit* function perform the following:
  1. Locates the audit log entry from the ID passed in from *AuditsController::replay()*.
  2. Grabs the data field and decodes it from json the associative array.
  3. Locates the user object in question from the ID field of the data array.
  4. Creates a new audit log entry for good measure.
  5. Update or overwrite the value of the user object with the value from the data array.
  6. Save the user object.
  7. Redirect to the user's edit page for the operator to confirm or edit some more.

##### Displaying entry details and custom rendering partials
An audit log entry that is replay-able will most likely have some data attributes that make it unique and has to be processed 
in order to be displayed properly. In the call to the *Audit::log()* function above that creates the replay-able entry, the 
5th parameter, the *data_parser*, is the fully qualified name of a function that will get called to prepare the data before 
returning the view displaying the details to the browser. 
Additionally the *data_parser* function can add to the *data* array an entry with a key of *show_partial* that points to 
a partial blade file that will be responsible for rendering the parsed data of the audit log entry. If no *show_partial* is 
specified the default behaviour is to use *var_dump()* to simply dump the value on the page.

### jqGrid datatables & reports.
Advanced datatables and reports can be easily created using [jqGrid](http://www.trirand.com/blog/) and 
[mgallegos/laravel-jqgrid](https://github.com/mgallegos/laravel-jqgrid). 

In a development environment, 3 reports
are available as sample: 

* A simple user list.
* A list of routes with their associated permission, using a join.
* A list of permissions and roles for each user, grouped by user and roles. This is implemented by first creating a view
 in the database as it would have been too complicated for Laravel Query Builder to handle.
 
**_NOTE:_** The view is created in a migration, but will only be created (and dropped) in the development environment.

**_NOTE:_** All jqGRids reports are using the base jQuery UI, except the routes report that uses the trontastic theme, 
to show how easy it is to change jQuery UI theme.

### Rapyd demo
To enable the demo mini sub-site that comes with [rapyd-laravel](https://github.com/zofe/rapyd-laravel) uncomment the 
following line at the end of the file *app/Http/rapyd.php*:

```
// Uncomment to enable the demo route.
Route::controller('rapyd-demo', '\Zofe\Rapyd\Demo\DemoController');
```

For more information on how to use the [Rapyd CRUD and Grid](https://github.com/zofe/rapyd-laravel) feature please refer 
to original package notes.

### Modules
Small features can be grouped into modules and managed, enabled and upgraded, without impacting the entire site.
Refer to the documentation on the [l51esk-modules](https://github.com/sroutier/l51esk-modules) and look at a 
concrete example [Module Demo](https://github.com/sroutier/l51esk-modules-demo-1) for more details.  

### LERN
To enable LERN (Laravel Exception Recorder and Notifier) set the configuration option as show below:
```
LERN_ENABLED=false
```
Once enabled, LERN can record exceptions in the table ```lern_exceptions``` as well as email them if your mail system is properlly
configured. A few more options are available to tweak the behaviour of LERN, refer to the ```lern.php``` configuration page and the [project home page](https://github.com/tylercd100/lern).
All recorded exceptions can be seen in the ``` Error ``` page from the ```Admin``` menu. Individual exception also can be reviewed and should the need arise, old entries can be deleted with the purge button. The setting ```ERRORS_PURGE_RETENTION``` let you configure how 
far back, in days, exceptions are kept when purging. By default any exception older than ```30``` days is deleted.

### Context sensitive help

Context sensitive help can be enabled by setting the configuration option ```APP_CONTEXT_HELP_AREA``` to true. When context sensitive help is enabled, a question mark (?) appears in the top-right area of the Web application. The question mark is either dimmed and disabled, or lit and enabled dependint on whether context sensitive help is available or not. For example on the home page the question mark will be dimmed and disabled but on the User edit page it will be lit and enabled. When clicked on, a small box will appear with the content of the context help inside. The help box can be dismissed by clicking anywhere outside the box.

To create a new context sensitive help box, simply create a blade file under the ```resources/themes/default/views/context_help/``` folder followed by a struture representing the name of your route. For example the user edit page is accessed by the ```admin.users.edit``` route so create a blade page named ```edit.blade.php``` under ```resources/themes/default/views/context_help/admin/users/``` and it will automatically be loaded and shown when a user click on the question mark (?) icon.

Module can also create context sensitive help by following the same principle but they must also set the __context__ parameter when they call the parent __constructor__, here is how the (Active Directory Inspector)[https://github.com/sroutier/L51ESK-Module_ActiveDirectoryInspector] module sets it, notice the 3rd parameter in the ```parent::__construct()``` call:
```
...
    /**
     * Custom constructor to get a handle on the Application instance.
     * @param Application $app
     */
    public function __construct(Application $app, Audit $audit)
    {
        parent::__construct($app, $audit, "activedirectoryinspector");
        $this->app = $app;
        $this->ldapConfig = $this->app['config']['activedirectoryinspector'];
    }
...
```
Once the context set, the context sensitive help for the __home__ page is automatically loaded from ```app/Modules/ActiveDirectoryInspector/Resources/views/context_help/activedirectoryinspector/home.blade.php```This path is generated using this formula: ```app/Modules/<module namespace>/Resources/views/context_help/<route name>/<last part of route name>.blade.php```


## Deploying to production
Before deploying to a production or live server, you should take care of a few tasks.

### Combine and minimize
Although not required, it may be helpful to combine and minimize both the CSS and JS files. Add any CSS and JS file
that you may have added to the *styles* and *scripts* command (respectively) in the *gulpfile.js* and run the 
*gulp* command with the *--production* command line parameter to trigger the minimization process, as shown 
below:

```
gulp --production
```

## Troubleshooting
Below are some troubleshooting tips that we have encoutered and resolved:

### Node.js

#### Old version
As pointed out by [thassan](https://github.com/thassan) in [Issue 6](https://github.com/sroutier/laravel-5.1-enterprise-starter-kit/issues/6), 
if you distribution or OS ships with an older version of Node.js the name of the executable may be 'nodejs'. In recent versions the name has 
been changed to 'node'. This will cause some Node.js packages to fail during the installation as they expect to find the 'node' executable. 
To resolve this issue you can either create a symbolic link from the 'nodejs' executable to 'node', or you may want to consider installing 
a more recent version of Node.js.
To create a symbolic link issue the command:
```
sudo ln -s /usr/bin/nodejs /usr/bin/node
```

#### ENOENT error
If the installation of Node.js packages fails with a 'ENOENT' error, you may need to create a empty file at the root of the project as 
explained on [Stack Overflow](http://stackoverflow.com/questions/17990647/npm-install-errors-with-error-enoent-chmod). 
To create the empty file run:
```
touch .npmignore
```

#### Hangs
If the installation of Node.js packages appears to hang, it may be due to a race condition that does not manifest itself when invoked
with the ``-ddd`` after having deleted the ``node_modules`` directory at the root of the project:
```
rm -rf node_modules
npm install -ddd
```

## Change log
Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Security
If you discover any security related issues, please email sroutier@gmail.com instead of using the issue tracker.

## Issues
For the list of all current and former/closed issues see the [github issue list](https://github.com/sroutier/laravel-5.1-enterprise-starter-kit/issues).
If you find a problem, please follow the same link and create an new issue, I will look at it and get back to you ASAP.

## Contributing
I would be glad to accept your contributions if you want to participate and share. Just follow GitHub's guide on how 
to [fork a repository](https://help.github.com/articles/fork-a-repo/). Clone your repository to your machine, make 
your change then create a pull request after submitting your change to your repository.

## Credits & inspirations
It goes without saying that none of this could have been done without the great [Laravel](http://laravel.com/) 
framework, a big thank you goes out to [Taylor Otwell](http://taylorotwell.com/) and the hundreds of volunteers 
of the Laravel & Open Source community.

I would like to thank [Jeffrey Way](https://twitter.com/jeffrey_way) for the excellent [Laracast](https://laracasts.com/)
 a never ending source of knowledge.

Additionally credit goes out to the authors of the various components and modules, noted in the sections above, used 
as part of this project. 
 
Finally I would like to point to a number of projects that served as inspiration and great source of learning material.
These projects are similar to the L51ESK, but did not fully cover the requirements that I had. You may want to
 have a look at them, here is the list:
 
* [yajra/laravel-admin-template](https://github.com/yajra/laravel-admin-template) Laravel 4.2 Bootstrap Admin Starter Template, with Oracle DB Support.
* [start-laravel/sb-admin-laravel-5](https://github.com/start-laravel/sb-admin-laravel-5) Starter template / theme for Laravel 5.
* [Zemke/starter-laravel-angular](https://github.com/Zemke/starter-laravel-angular) Laravel and AngularJS Starter Application Boilerplate featuring Laravel 5 and AngularJS 1.3.13.
* [mrakodol/Laravel-5-Bootstrap-3-Starter-Site](https://github.com/mrakodol/Laravel-5-Bootstrap-3-Starter-Site) Laravel Framework 5 Bootstrap 3 Starter Site is a basic application with news, photo and video galleries.                                                                                                                
* [todstoychev/Laravel5Starter](https://github.com/todstoychev/Laravel5Starter) A Laravel 5 starter project. It contains user management with roles and basic admin panel with application settings.

### License
The L51ESK is open-sourced software licensed under the GNU General Public License Version 3 (GPLv3). 
Please see [License File](LICENSE.md) for more information.


[ico-version]: https://img.shields.io/badge/packagist-v0.1.0-orange.svg
[ico-license]: https://img.shields.io/badge/licence-GPLv3-brightgreen.svg

[link-packagist]: https://packagist.org/packages/sroutier/laravel-5.1-enterprise-starter-kit

