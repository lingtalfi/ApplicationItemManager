ApplicationItemManager
========================
2017-03-30 -> 2017-03-21




A manager for the modules of your application.



This is part of the [universe framework](https://github.com/karayabin/universe-snapshot).





What is it?
================

This planet contains a set of tools helping you implementing a management system for your modules.


A picture being worth a thousand words, let's start with a picture:

[![ApplicationItemManager-overview.jpg](https://s19.postimg.org/kn2gln6g3/Application_Item_Manager-overview.jpg)](https://postimg.org/image/5ecj7vcrj/)


As you can probably see, there are different objects:

- Repository: this object represents a web repository (like a github repo for instance). A repository contains items of a certain type.
            The type can be what you want: a module, a plugin, a theme, a widget, you name it.
            
            
- Application: this is your application. There is an important directory called **import directory**, which is where
                the items will be imported.
                
- Importer: knows the technique of how to copy an item from the web repository to the **import directory** of your app.
                
- Installer: some items (like modules for instance) might require to be installed after they are imported.
                The Installer is the object for installing/uninstalling items in your application.
                What installing means depends on your item type, it might involve actions like copying files in your
                application, creating tables in your database, etc...
                
- ApplicationItemManager: this is the manager of all those objects: it coordinates the actions between the different
                            actors and provide a simple api for the developer, with methods like import, install, search, and so on.
                            
- Program: the program wraps the ApplicationItemManager into a console program, allowing you to control the ApplicationItemManager from the command line.                            



Note: the Program is actually a planet itself: it's the [Program]([Program](https://github.com/lingtalfi/program)) planet.



Install
==========
Download the repository directly, or you can use the [uni importer](https://github.com/lingtalfi/universe-naive-importer):


```bash
uni import ApplicationItemManager
```



How?
==========

First, decide what's an item: is it a module?, a plugin?, a theme?
 
Then, decide where you want to import them, this directory is called the import directory.
The import directory is where your items will be downloaded.


Then, you can use the ApplicationItemManager.

An ApplicationItemManager is an object that provides useful commands for importing/installing/listing/searching items.

To use the ApplicationItemManager as a standalone tool, you can use the following example as a starting point:




```php
<?php

use ApplicationItemManager\Importer\GithubImporter;
use ApplicationItemManager\Installer\KamilleModuleInstaller;
use ApplicationItemManager\Installer\KamilleWidgetInstaller;
use ApplicationItemManager\LingApplicationItemManager;
use ApplicationItemManager\Repository\KamilleModulesRepository;
use ApplicationItemManager\Repository\KamilleWidgetsRepository;
use ApplicationItemManager\Repository\LingUniverseRepository;
use Kamille\Architecture\ApplicationParameters\ApplicationParameters;
use Output\WebProgramOutput;

require_once __DIR__ . "/../boot.php";
require_once __DIR__ . "/../init.php";


$test = "modules";


if ("universe" === $test) {

    //--------------------------------------------
    // UNIVERSE APP MANAGER
    //--------------------------------------------
    $output = WebProgramOutput::create(); // testing from a browser, change this to ProgramOutput to test from cli
    $importDir = "/myphp/kaminos/app/planets";
    $manager = LingApplicationItemManager::create()// the LingApplicationItemManager is just Output friendly
    ->setOutput($output)
        ->addRepository(LingUniverseRepository::create())
        ->bindImporter('ling', GithubImporter::create()->setGithubRepoName("lingtalfi"))
        ->setFavoriteRepositoryId('ling')
        ->setImportDirectory($importDir);


    // below are the most useful commands
    a($manager->search("ba")); // search the term "ba" in the available items
    $manager->listAvailable(); // list the available items
    $manager->install("Bat"); // install the Bat item (will import it if necessary)
    $manager->import("AdminTable"); // import the AdminTable item
    $manager->listImported(); // list the imported items
    $manager->listInstalled(); // list the installed items

} elseif ("widgets" === $test) {
    //--------------------------------------------
    // KAMILLE WIDGETS APP MANAGER
    //--------------------------------------------
    $output = WebProgramOutput::create();
    $appDir = ApplicationParameters::get("app_dir"); // this is specific to the kaminos app I'm testing, don't worry
    LingApplicationItemManager::create()
        ->setOutput($output)
        ->setInstaller(KamilleWidgetInstaller::create()->setOutput($output)->setApplicationDirectory($appDir))
        ->bindImporter('KamilleWidgets', GithubImporter::create()->setGithubRepoName("KamilleWidgets"))
        ->setFavoriteRepositoryId('KamilleWidgets')
        ->setImportDirectory("/myphp/kaminos/app/class-widgets")
        ->addRepository(KamilleWidgetsRepository::create(), ['km'])// notice the km alias, because KamilleWidgets is a long prefix to type
        ->install("BookedMeteo");


} elseif ("modules" === $test) {
    //--------------------------------------------
    // KAMILLE MODULES APP MANAGER
    //--------------------------------------------
    $output = WebProgramOutput::create();
    $appDir = ApplicationParameters::get("app_dir");
    LingApplicationItemManager::create()
        ->setOutput($output)
        ->setInstaller(KamilleModuleInstaller::create()->setOutput($output)->setApplicationDirectory($appDir))
        ->bindImporter('KamilleModules', GithubImporter::create()->setGithubRepoName("KamilleModules"))
        ->setFavoriteRepositoryId('KamilleModules')
        ->setImportDirectory("/myphp/kaminos/app/class-modules")
        ->addRepository(KamilleModulesRepository::create())
        ->install("Connexion");
//        ->uninstall("Connexion");
}






```


The ApplicationItemManagerInterface exposes the following methods:

- import($item, $force = false)
- install($item, $force = false)
- uninstall($item)
- listAvailable($repoId = null, array $keys = null)
- listImported()
- listInstalled()
- search($text, array $keys = null, $repoId = null)



The most important are perhaps the import and install/uninstall methods.

The algorithm for those methods can be found in this repository: I made two pdf documents
describing those algorithms, in the "design" directory of this repository.

- https://github.com/lingtalfi/ApplicationItemManager/blob/master/doc/design/ApplicationItemManager-import-install-item-algo.pdf
- https://github.com/lingtalfi/ApplicationItemManager/blob/master/doc/design/ApplicationItemManager-uninstall-item-algo.pdf





Creating Console Programs
============================


Once you've configured an ApplicationItemManager instance to your likings,
you can create a console program out of it.

The ApplicationItemManagerProgram object helps you a long way with that, encapsulating 
your ApplicationItemManager instance and providing program commands for free (using the
[Program](https://github.com/lingtalfi/program) planet under the hood):





```txt
Usage
-------

The word item is defined like this:
- item: itemId | itemName
- itemId: repositoryId.itemName | repositoryAlias.itemName



myprog import {item}                       # import an item and its dependencies, skip already existing item(s)/dependencies
myprog import -f {item}                    # import an item and its dependencies, replace already existing item(s)/dependencies
myprog install {item}                      # install an item and its dependencies, will import them if necessary, skip already existing item(s)/dependencies
myprog install -f {item}                   # install an item and its dependencies, will import them if necessary, replace already existing item(s)/dependencies
myprog uninstall {item}                    # call the uninstall method on the given item and dependencies
myprog list {repoAlias}?                   # list available items
myprog listd {repoAlias}?                  # list available items with their description if any
myprog listimported                        # list imported items
myprog listinstalled                       # list installed items
myprog search {term} {repoAlias}?          # search through available items names
myprog searchd {term} {repoAlias}?         # search through available items names and/or description
myprog clean                               # removes the .git, .gitignore, .idea and .DS_Store files in your items directories, recursively


For instance:
    myprog import Connexion
    myprog import km.Connexion
    myprog import -f Connexion
    myprog import -f km.Connexion
    myprog install Connexion
    myprog install km.Connexion
    myprog install -f Connexion
    myprog install -f km.Connexion
    myprog uninstall Connexion
    myprog uninstall km.Connexion
    myprog list
    myprog list km
    myprog listd
    myprog listd km
    myprog listimported
    myprog listinstalled
    myprog search ling
    myprog search ling km
    myprog searchd kaminos
    myprog searchd kaminos km
    myprog clean
```






Here is the code required to create such a console program.
The example below use the ApplicationItemManager for the universe.



```php
#!/usr/bin/env php
<?php


use ApplicationItemManager\Importer\GithubImporter;
use ApplicationItemManager\LingApplicationItemManager;
use ApplicationItemManager\Program\ApplicationItemManagerProgram;
use ApplicationItemManager\Repository\LingUniverseRepository;
use CommandLineInput\ProgramOutputAwareCommandLineInput;
use Kamille\Architecture\ApplicationParameters\ApplicationParameters;
use Output\ProgramOutput;

//--------------------------------------------
// UNIVERSE PROGRAM
//--------------------------------------------
$_SERVER['APPLICATION_ENVIRONMENT'] = "dev"; // hack environment here depending on your prefs
require_once __DIR__ . "/../boot.php"; // start your autoloaders...

$output = ProgramOutput::create();
$appDir = ApplicationParameters::get("app_dir");
$importDir = "/myphp/kaminos/app/planets";
$manager = LingApplicationItemManager::create()
    ->setOutput($output)
    ->addRepository(LingUniverseRepository::create(), ['km'])
    ->setFavoriteRepositoryId('ling')
    ->bindImporter('ling', GithubImporter::create()->setGithubRepoName("lingtalfi"))
    ->setImportDirectory($importDir);


$input = ProgramOutputAwareCommandLineInput::create($argv)
    ->setProgramOutput($output)
    ->addFlag("f")
    ->addFlag("v");

ApplicationItemManagerProgram::create()
    ->setManager($manager)
    ->setInput($input)
    ->setOutput($output)
    ->setImportDirectory($importDir)
    ->start();

```




Some programs which use this system are:

- [uni](https://github.com/lingtalfi/universe-naive-importer) (the universe naive importer)
- [kamille installer tool](https://github.com/lingtalfi/kamille-installer-tool) (which can install modules and widgets)











More about repositories
=====================


Testing repositories
----------------------

```php


//--------------------------------------------
// TESTING REPOSITORIES
//--------------------------------------------
a(KamilleModulesRepository::create()->getDependencies("KamilleModules.Connexion"));
a(KamilleModulesRepository::create()->getHardDependencies("KamilleModules.Connexion"));
a(LingUniverseRepository::create()->getDependencies("ling.ArrayStore"));
```


Dependency and hard dependency
------------------------------------

Do you know the difference between a dependency and a hard dependency?

The dependency is a link between two items A and B, which is honored upon installation,
such as when you install item A, item B is also installed (assuming B depends on A).


A hard dependency is also a link between two items A and B, which is honored upon uninstallation,
such as when you uninstall item A, item B is also uninstalled (assuming B depends on A).










History Log
------------------
    
- 1.1.0 -- 2017-03-30

    - renamed ItemList to Repository
    
- 1.0.0 -- 2017-03-30

    - initial commit
    




