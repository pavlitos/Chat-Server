2018-08-22/fki Revised for lab 1 version 7
07-feb-2011/FK
06-feb-2009/FK
07-feb-2008/FK

---
2018-08-22: Changes in lab 1 version 7

- Source code and compilation brought up to be compatible with Java 5
  and upwards.

- Use of the rmic tool was removed, including the generation of static
  skeleton and stub files.

- The download files ChatServer-dl.jar and CharClient-dl.jar were removed.

- The separate client and server directories in the codebase server
  directory were removed. All jar files are now in the top directory
  of the codebase server.

- Build and run files for Windows (develop/bat, test/bin/pc) were
  reviewed, debugged, and refactored.

- ChatClient.java and ChatServer.java:

    = Import clauses are no longer wildcarded

    = Code upgraded to use generic collection types and iterators

    = An issue with hanging threads on shutdown was circumvented
      by calling System.exit(0) at the end of main.

    = The deprecated RMISecurityManager was replaced with the more
      modern SecurityManager.

- ChatClient.java:

    = The method connectToChat was rewritten to lock the servers
      Vector for a minimal amount of time and provide a cleaner
      logic. The server name pattern matching was changed from infix
      to prefix, and to be case-sensitive.

    = The method stringJoin was modified to use a StringBuilder
      instead of a String.

    = The command interpreter in method readLoop was simplified in its
      string management. It performs exactly as in previous versions,
      but with less code.

    = The client takes a default name from the user.name
      property. This can still be changed with the .name command.

- ChatServer.java:

    = Synchronization on the clients Vector is applied to protect
      iterators created from it.

    = The serverName string was changed so that it begins with the
      user-supplied name (defaults to user.name).

    = The method getNextMessage was rewritten to be more efficient.

  --

INTRODUCTION TO LAB CHAT

The system presented here consists of a chatserver and a client.
The server and the clients are both Jini applications, so they depend
on a Jini infrastructure being present in order to find each other.

Furthermore, both server and client are deployed in a distributed
fashion. This means that they are not run directly from local files.
Instead, a small and general bootstrap program (JarRunner.jar) is
used, and when executed with the correct parameters it fetches the
real program from a web server and runs it.

Your assignment is to select a development task from a list of
alternatives (given separately), and then to modify the system
according to that task. This text describes the technical setup of the
system and how to work with it.

This distribution has two major parts:

  - The application sources and the files needed to compile and build
    them. These files are found in the   develop   subdirectory.

  - Files that start the Jini service infrastructure and runs the
    chatserver and client. These files reside in the   test   sub-
    directory.


There will be several technical points to go through, but before that
some emphasis on things that can be learned from this excercise:

  - An application deployed on-demand from a webserver is much easier
    to maintain for a large group of users, since upgrades only are
    done on the webserver. No need to run around among users or rely
    on their ability to perform the upgrade themselves.

  - A separation of the development and test environment reflects how
    the program will actually be used. When the compiler and the
    runtime are on two different computer systems it is much
    easier to spot incompatibilities between them.

  - Dynamic discovery of services through the Jini middleware
    illustrates flexibility in a client-server or peer-to-peer
    system. Services and clients can be added, moved, and removed
    dynamically, without having to reconfigure the components.


PREPARATION

Initially there are some choices to be made.

    You must select a development and test platform (Mac, Linux or Windows)
    You must choose a webserver (a default is provided)


Development platform

  The distribution as given was verified (August 2018) on Ubuntu
  version 18, and Windows 7. At this time the Java source level was
  raised from Java 1.4 to be compatible with Java 5, 6, 7, and 8.

  Your computer should have:

  Windows, Mac/OS, or Linux, of a recently modern flavour.

  Java SDK 1.5 or later.

  emacs (or some other source-code editor)

  ant (Another Neat Tool) is available from ant.apache.org and is used
  to compile, build, and install the software. Instructions for ant
  are in the file build.xml.



  As an alternative to ant, the develop/bat directory contains Windows
  BAT-files that helps with compilation, building, and installation.

  It may be possible to use an IDE for development, but this is not
  supported.

  In order to build and install the software by other means than ant
  or Windows BAT-files, the general outline is this (see build.xml for
  details):

  - Compile the sources into class files, using the provided
    libraries. The develop/build directory is the recommended target
    for class files.

  - Put the class files in develop/build into jar-files, using the
    manifest files in develop/mf. Put the jar-files in the
    develop/dist directory.

  - Copy the jar-files in develop/dist to test/cbs

  - (Once) Make sure that the files in develop/lib (jini-core.jar,
    jini-ext.jar, and reggie-dl.jar) are copied to the test/cbs
    directory.


Deployment and codebase webserver

  A critical component of the system is the codebase server. This
  consists of a web-server that can serve class-files to the
  applications as they need them. A small web-server is provided with
  this distribution, and its default codebase (the directory from
  which it serves its files) is in the test/cbs directory. The script
  test/bin/pc/r1_httpd.bat starts the web-server and its code is in
  the lib/tools.jar archive which is part of the Jini distribution.

  However, if you have access to another webserver which you think
  would be better for you, you can use that. Just remember to change
  the launching scripts in test/bin/{pc,unix} to provide the correct
  URL to the applications as they start, and to arrange for an easy
  way to upload new versions of the jar files each time you rebuild
  the system.

  The CHAT system needs these files in the codebase server:

  test/cbs/jini-core.jar   The Jini middleware
  test/cbs/jini-ext.jar    The Jini middleware
  test/cbs/reggie-dl.jar   The Jini middleware

  test/cbs/ChatServer.jar  The chat server
  test/cbs/ChatClient.jar  The chat client


Testing platform

  
  The chat-server and client will not find each other unless a Jini
  lookup service (reggie) is running. The lookup server is actually an
  RMI service, so it needs rmid (the RMI daemon) running on its
  computer.



INSTALLING

  Installation is easy, just unzip the distribution archive into a
  folder of your choice.

  If you pick a non-default directory for installation, edit:

      develop/build.xml              Path to webserver directory

  Verify that you are able to:

    - compile the sources

    - install jar-files on the webserver:

	  jini-core.jar              copied from develop/lib
	  jini-ext.jar               copied from develop/lib
	  reggie-dl.jar              copied from develop/lib

	  ChatClient.jar             compiled from sources
	  ChatServer.jar             compiled from sources

    - Once the webserver is running, verify that files can be fetched
      by opening a web browser and pointing it to the URLs of the
      files.

  It is important that you are able to automate the installation on
  the webserver, because it is otherwise easy to forget that step when
  chasing a bug over frequent recompilations.


RUNNING THE SYSTEM

  * Windows *

  If you are on Windows, open a command window (CMD.EXE) and go to
  test/bin/pc

  Run r1_httpd.bat to launch the HTTP server. It opens in a separate,
  minimized window labelled httpd. To shutdown the HTTP server, close
  that window.

  Run r2_rmid.bat to start the RMI deaemon. It opens in a separate,
  minimized window. To shutdown the RMI daemon (and reggie), give the
  command rmid -stop in any command window with a listening prompt.

  Run r3_reggie.bat to start the Jini lookup server (reggie). Upon
  startup it registers itself with rmid as an activatable service. As
  a result, later output from the lookup server is printed in the rmid
  window.

  Assuming all goes well, the Jini middleware is now running.

  Starting a chat server:

    Open a new command window, e.g. with: START CMD

    Run chatserver.bat

    To see commandline options and exit, run

      chatserver -h

  Starting a chat client:

    Open a new command window, e.g. with: START CMD

    Run chatclient.bat


  * Linux, Mac *

    Mac users may want to make sure that the unix script files have
    proper line termination characters (^M) for their system.

  On Ubuntu, the keyboard command ctrl+alt-t opens a new Gnome
  Terminal application, with the current directory being your home
  directory.

  While IN the Gnome terminal application, the keyboard command
  ctrl+shift+n opens a new terminal with the current directory
  unchanged. This is very helpful below.

  Open a command shell and go to test/bin/unix

  Open a new command shell in the test/bin/unix directory and run

    ./r1_httpd.sh

    This starts the HTTP server. To shut it down, type ctrl+c twice.

  Open a new command shell in the test/bin/unix directory and run

    ./r2_rmid.sh

    This starts the RMI daemon. To shutdown the RMI daemon (and
    reggie), give the command

    rmid -stop

    in some other command shell (terminal).

  Open a new command shell in the test/bin/unix directory and run

    ./r3_reggie.sh

    This starts the Jini lookup server (reggie). Upon startup it
    registers itself with rmid as an activatable service. As a result,
    later output from the lookup server is printed in the rmid window.

  Assuming all goes well, the Jini middleware is now running.

  Starting a chat server:

    Open a new command shell in the test/bin/unix directory and run

      ./chatserver.sh

    To see commandline options and exit, run

      ./chatserver.sh -h

  Starting a chat client:
  
    Open a new command shell in the test/bin/unix directory and run

      ./chatclient.sh


TESTING

  The Jini middleware, i.e. the local webserver, rmid and the lookup
  server (reggie) can be left running while you start and stop the
  chat-server and clients (in their own command windows).

  It is not always necessary to restart rmid and reggie after each
  recompile, but it is of course the only way to be sure of a clean
  restart. The lookup server (reggie) may be holding on to
  registrations for at least 5 minutes after the service has died, so
  there is a risk of old or stale entries conflicting with new code.
