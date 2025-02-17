Getting Started
===============

Introduction
------------

This chapter provides a brief introduction to creating EPICS IOC
applications. It contains:

-  Instructions for creating, building, and running an example IOC
   application.

-  Instructions for creating, building, and executing example Channel
   Access clients.

-  Briefly describes iocsh, which is a base supplied command shell.

-  Describes rules for building IOC components.

-  Describes makeBaseApp.pl, which is a perl script that generates files
   for building applications.

-  Briefly discusses vxWorks boot parameters

This chapter will be hard to understand unless you have some familarity
with IOC concepts such as record types, device and driver support and
have had some experience with creating ioc databases. Once you have this
experience, this chapter provides most of the information needed to
build applications. The example that follows assumes that EPICS base has
already been built.

.. _Example IOC Application:

Example IOC Application
-----------------------

This section explains how to create an example IOC application in a
directory ``<top>``, naming the application ``myexampleApp`` and the ioc
directory ``iocmyexample``.

Check that ``EPICS_HOST_ARCH`` is defined
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Execute the command:

.. code:: sh

   echo $EPICS_HOST_ARCH

or

.. code:: sh

   set EPICS_HOST_ARCH

This should display your workstation architecture, for example
``linux-x86`` or ``win32-x86``. If you get an “Undefined variable"
error, you should set ``EPICS_HOST_ARCH`` to your host operating system
followed by a dash and then your host architecture, e.g.
``solaris-sparc``. The perl script ``EpicsHostArch.pl`` in the
base/startup directory has been provided to help set
``EPICS_HOST_ARCH``.

Create the example application
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following commands create an example application.

.. code:: sh

   mkdir <top>
   cd <top>
   <base>/bin/<arch>/makeBaseApp.pl -t example myexample
   <base>/bin/<arch>/makeBaseApp.pl -i -t example myexample

Here, ``<arch>`` indicates the operating system architecture of your
computer. For example, ``solaris-sparc``. The last command will ask you
to enter an architecture for the IOC. It provides a list of
architectures for which base has been built.

The full path name to ``<base>`` (an already built copy of EPICS base)
must be given. Check with your EPICS system administrator to see what
the path to your ``<base>`` is. For example:

.. code:: sh

   /home/phoebus/MRK/epics/base/bin/linux-x86/makeBaseApp.pl ...

Windows Users Note: Perl scripts must be invoked with the command
``perl <scriptname>`` on Windows. Perl script names are case sensitive.
For example to create an application on Windows:

.. code:: sh

   perl C:\epics\base\bin\win32-x86\makeBaseApp.pl -t example myexample

Inspect files
~~~~~~~~~~~~~

Spend some time looking at the files that appear under ``<top>``. Do
this *before* building. This allows you to see typical files which are
needed to build an application without seeing the files generated by
make.

Sequencer Example
~~~~~~~~~~~~~~~~~

The sequencer is now supported as an unbundled product. The example
includes an example state notation program, ``sncExample.stt``. As
created by makeBaseApp the example is not built or executed.

Before ``sncExample.stt`` can be compiled, the sequencer module must
have been built using the same version of base that the example uses.

To build sncExample edit the following files:

-  ``configure/RELEASE`` – Set SNCSEQ to the location of the sequencer.

-  ``iocBoot/iocmyexample/st.cmd`` – Remove the comment character # from
   this line:

   ``#seq sncExample, "user=<user>"``

The Makefile contains commands for building the sncExample code both as
a component of the example IOC application and as a standalone program
called ``sncProgram``, an executable that connects through Channel
Access to a separate IOC database.

Build
~~~~~

In directory ``<top>`` execute the command

.. code:: sh

   make

NOTE: On systems where GNU make is not the default another command is
required, e.g. ``gnumake``, ``gmake``, etc. See you EPICS system
administrator.

.. _inspect-files-1:

Inspect files
~~~~~~~~~~~~~

This time you will see the files generated by make as well as the
original files.

Run the ioc example
~~~~~~~~~~~~~~~~~~~

The example can be run on vxWorks, RTEMS, or on a supported host.

-  On a host, e.g. Linux or Solaris

   .. code:: sh

      cd <top>/iocBoot/iocmyexample
      ../../bin/linux-x86/myexample st.cmd

-  vxWorks/RTERMS – Set your boot parameters as described at the end of
   this chapter and then boot the ioc.

After the ioc is started try some of the shell commands (e.g. ``dbl`` or
``dbpr <recordname>``) described in the chapter “IOC Test Facilities".
In particular run ``dbl`` to get a list of the records.

The iocsh command interpreter used on non-vxWorks IOCs provides a help
facility. Just type:

::

   help

or

::

   help <cmd>

where ``<cmd>`` is one of the commands displayed by help. The help
command accepts wildcards, so

::

   help db*

will provide information on all commands beginning with the characters
db. On vxWorks the help facility is available by first typing:

::

   iocsh

Channel Access Host Example
---------------------------

An example host example can be generated by:

.. code:: sh

   cd <mytop>
   <base>/bin/<arch>/makeBaseApp.pl -t caClient caClient
   make

(or gnumake, as required by your operating system)

Two channel access examples are provided:

| ``caExample``
| This example program expects a pvname argument, connects and reads the
  current value for the pv, displays the result and terminates. To run
  this example just type.

``<mytop>/bin/<hostarch>/caExample <pvname>`` where

-  ``<mytop>`` is the full path name to your application top directory.

-  ``<hostarch>`` is your host architecture.

-  ``<pvname>`` is one of the record names displayed by the ``dbl`` ioc
   shell command.

| ``caMonitor``
| This example program expects a filename argument which contains a list
  of pvnames, each appearing on a separate line. It connects to each pv
  and issues monitor requests. It displays messages for all channel
  access events, connection events, etc.

iocsh
-----

Because the vxWorks shell is only available on vxWorks, EPICS base
provides iocsh. In the main program it can be invoked as follows:

::

   iocsh("filename")

or

::

   iocsh(0)

If the argument is a filename, the commands in the file are executed and
iocsh returns. If the argument is 0 then iocsh goes into interactive
mode, i.e. it prompts for and executes commands until an exit command is
issued.

This shell is described in more detail in Chapter
`[chap:IOC Shell] <#chap:IOC Shell>`__, “IOC Shell".

On vxWorks iocsh is not automatically started. It can be started by just
giving the following command to the vxWorks shell.

::

   iocsh

To get back to the vxWorks shell just say

::

   exit

Building IOC components
-----------------------

Detailed build rules are given in chapter :doc:`EPICS Build Facility<EPICSBuildFacility>`.
This section describes methods for building most components needed for IOC
applications. It uses excerpts from the ``myexampleApp/src/Makefile``
that is generated by makeBaseApp.

The following two types of applications can be built:

Support applications

These are applications meant for use by ioc applications. The rules
described here install things into one of the following directories that
are created just below ``<top>``:

| ``include``
| C include files are installed here. Either header files supplied by
  the application or header files generated from ``xxxRecord.dbd`` or
  ``xxxMenu.dbd`` files.

| ``dbd``
| Each file contains some combination of ``include``, ``recordtype``,
  ``device``, ``driver``, and ``registrar`` database definition
  commands. The following are installed:

-  ``xxxRecord.dbd`` and ``xxxMenu.dbd`` files

-  An arbitrary ``xxx.dbd`` file

-  ioc applications install a file ``yyy.dbd`` generated from file
   ``yyyInclude.dbd``.

| ``db``
| Files containing record instance definitions.

| ``lib/<arch>``
| All source modules are compiled and placed in shared or static library
  (win32 dll)

IOC applications

These are applications loaded into actual IOCs.

Binding to IOC components
~~~~~~~~~~~~~~~~~~~~~~~~~

Because many IOC components are bound only during ioc initialization,
some method of linking to the appropriate shared and/or static libraries
must be provided. The method used for IOCs is to generate, from an
``xxxInclude.dbd`` file, a C++ program that contains references to the
appropriate library modules. The following database definitions keywords
are used for this purpose:

::

   recordtype
   device
   driver
   function
   variable
   registrar

The method also requires that IOC components contain an appropriate
epicsExport statement. All components must contain the statement:

.. code:: c

   #include <epicsExport.h>

Any component that defines any exported functions must also contain:

.. code:: c

   #include <registryFunction.h>

Each record support module must contain a statement like:

::

   epicsExportAddress(rset,xxxRSET);

Each device support module must contain a statement like:

.. code:: c

   epicsExportAddress(dset,devXxxSoft);

Each driver support module must contain a statement like:

.. code:: c

   epicsExportAddress(drvet,drvXxx);

Functions are registered using an ``epicsRegisterFunction`` macro in the
C source file containing the function, along with a ``function``
statement in the application database description file. The makeBaseApp
example thus contains the following statements to register a pair of
functions for use with a subroutine record:

.. code:: c

   epicsRegisterFunction(mySubInit);
   epicsRegisterFunction(mySubProcess);

The database definition keyword ``variable`` forces a reference to an
integer or double variable, e.g. debugging variables. The
``xxxInclude.dbd`` file can contain definitions like:

::

   variable(asCaDebug,int)
   variable(myDefaultTimeout,double)

The code that defines the variables must include code like:

.. code:: c

   int asCaDebug = 0;
   epicsExportAddress(int,asCaDebug);

The keyword ``registrar`` signifies that the epics component supplies a
named registrar function that has the prototype:

.. code:: c

   typedef void (*REGISTRAR)(void);

This function normally registers things, as described in Chapter
`[Registry] <#Registry>`__, “Registry" on page . The makeBaseApp example
provides a sample iocsh command which is registered with the following
registrar function:

.. code:: c

   static void helloRegister(void) {
       iocshRegister(&helloFuncDef, helloCallFunc);
   }
   epicsExportRegistrar(helloRegister);

Makefile rules
~~~~~~~~~~~~~~

Building a support application.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: makefile

   # xxxRecord.h will be created from xxxRecord.dbd
   DBDINC += xxxRecord
   DBD += myexampleSupport.dbd

   LIBRARY_IOC += myexampleSupport

   myexampleSupport_SRCS += xxxRecord.c
   myexampleSupport_SRCS += devXxxSoft.c
   myexampleSupport_SRCS += dbSubExample.c

   myexampleSupport_LIBS += $(EPICS_BASE_IOC_LIBS)

The ``DBDINC`` rule looks for a file ``xxxRecord.dbd``. From this file a
file ``xxxRecord.h`` is created and installed into ``<top>/include``

The ``DBD`` rule finds ``myexampleSupport.dbd`` in the source directory
and installs it into ``<top>/dbd``

The ``LIBRARY_IOC`` variable requests that a library be created and
installed into ``<top>/lib/<arch>``

The ``myexampleSupport_SRCS`` statements name all the source files that
are compiled and put into the library.

The above statements are all that is needed for building many support
applications.

Building the IOC application
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following statements build the IOC application:

.. code:: makefile

   PROD_IOC = myexample

   DBD += myexample.dbd

   # myexample.dbd will be made up from these files:
   myexample_DBD += base.dbd
   myexample_DBD += xxxSupport.dbd
   myexample_DBD += dbSubExample.dbd

   # <name>_registerRecordDeviceDriver.cpp will be created from <name>.dbd
   myexample_SRCS += myexample_registerRecordDeviceDriver.cpp
   myexample_SRCS_DEFAULT += myexampleMain.cpp
   myexample_SRCS_vxWorks += -nil-

   # Add locally compiled object code
   myexample_SRCS += dbSubExample.c

   # Add support from base/src/vxWorks if needed
   myexample_OBJS_vxWorks += $(EPICS_BASE_BIN)/vxComLibrary

   myexample_LIBS += myexampleSupport
   myexample_LIBS += $(EPICS_BASE_IOC_LIBS)

``PROD_IOC`` sets the name of the ioc application, here called
``myexample``.

| The DBD definition ``myexample.dbd`` will cause build rules to create
  the database definition include file
| ``myexampleInclude.dbd`` from files in the ``myexample_DBD``
  definition. For each filename in that definition, the created
  ``myexampleInclude.dbd`` will contain an include statement for that
  filename. In this case the created ``myexampleInclude.dbd`` file will
  contain the following lines.

::

   include "base.dbd"
   include "xxxSupport.dbd"
   include "dbSubExample.dbd"

When the DBD build rules find the created file ``myexampleInclude.dbd``,
the rules then call dbExpand which reads ``myexampleInclude.dbd`` to
generate file ``myexample.dbd``, and install it into ``<top>/dbd``.

| An arbitrary number of ``myexample_SRCS`` statements can be given.
  Names of the form
| ``<name>_registerRecordDeviceDriver.cpp,`` are special; when they are
  seen the perl script
| ``registerRecordDeviceDriver.pl`` is executed and given ``<name>.dbd``
  as input. This script generates the
  ``<name>_registerRecordDeviceDriver.cpp`` file automatically.

makeBaseApp.pl
--------------

``makeBaseApp.pl`` is a perl script that creates application areas. It
can create the following:

-  ``<top>/Makefile``

-  ``<top>/configure`` – This directory contains the files needed by the
   EPICS build system.

-  ``<top>/xxxApp`` – A set of directories and associated files for a
   major sub-module.

-  ``<top>/iocBoot`` – A subdirectory and associated files.

-  ``<top>/iocBoot/iocxxx`` – A subdirectory and files for a single ioc.

``makeBaseApp.pl`` creates directories and then copies template files
into the newly created directories while expanding macros in the
template files. EPICS base provides two sets of template files: simple
and example. These are meant for simple applications. Each site,
however, can create its own set of template files which may provide
additional functionality. This section describes the functionality of
makeBaseApp itself, the next section provides details about the simple
and example templates.

Usage
~~~~~

makeBaseApp has four possible forms of command line:

.. code:: sh

   <base>/bin/<arch>/makeBaseApp.pl -h

Provides help.

.. code:: sh

   <base>/bin/<arch>/makeBaseApp.pl -l [options]

List the application templates available. This invocation does not alter
the current directory.

.. code:: sh

   <base>/bin/<arch>/makeBaseApp.pl [-t type] [options] app ...

Create application directories.

.. code:: sh

   <base>/bin/<arch>/makeBaseApp.pl -i -t type [options] ioc ...

Create ioc boot directories.

Options for all command forms:

| ``-b base``
| Provides the full path to EPICS base. If not specified, the value is
  taken from the EPICS_BASE entry in config/RELEASE. If the config
  directory does not exist, the path is taken from the command-line that
  was used to invoke makeBaseApp

| ``-T template``
| Set the template top directory (where the application templates are).
  If not specified, the template path is taken from the TEMPLATE_TOP
  entry in config/RELEASE. If the config directory does not exist the
  path is taken from the environment variable EPICS_MBA_TEMPLATE_TOP, or
  if this is not set the templates from EPICS base are used.

| ``-d``
| Verbose output (useful for debugging)

Arguments unique to ``makeBaseApp.pl [-t type] [options] app ...``:

| ``app``
| One or more application names (the created directories will have “App"
  appended to this name)

| ``-t type``
| Set the template type (use the ``-l`` invocation to get a list of
  valid types). If this option is not used, type is taken from the
  environment variable EPICS_MBA_DEF_APP_TYPE, or if that is not set the
  values “default" and then “example" are tried.

Arguments unique to ``makeBaseApp.pl -i [options] ioc ...``:

| ``ioc``
| One or more IOC names (the created directories will have “ioc”
  prepended to this name).

| ``-a arch``
| Set the IOC architecture (e.g. vxWorks-68040). If ``-a arch`` is not
  specified, you will be prompted.

Environment Variables:
~~~~~~~~~~~~~~~~~~~~~~

| ``EPICS_MBA_DEF_APP_TYPE``
| Application type you want to use as default

| ``EPICS_MBA_TEMPLATE_TOP``
| Template top directory

Description
~~~~~~~~~~~

To create a new ``<top>`` issue the commands:

.. code:: sh

   mkdir <top>
   cd <top>
   <base>/bin/<arch>/makeBaseApp.pl -t <type> <app> ...
   <base>/bin/<arch>/makeBaseApp.pl -i -t <type> <ioc> ...

makeBaseApp does the following:

-  ``EPICS_BASE`` is located by checking the following in order:

   -  If the ``-b`` option is specified its value is used.

   -  If a ``<top>/configure/RELEASE`` file exists and defines a value
      for ``EPICS_BASE`` it is used.

   -  It is obtained from the invocation of the makeBaseApp program. For
      this to work, the full path name to the ``makeBaseApp.pl`` script
      in the EPICS base release you are using must be given.

-  ``TEMPLATE_TOP`` is located in a similar fashion:

   -  If the ``-T`` option is specified its value is used.

   -  If a ``<top>/configure/RELEASE`` file exists and defines a value
      for ``TEMPLATE_TOP`` it is used.

   -  If ``EPICS_MBA_TEMPLATE_TOP`` is defined its value is used.

   -  It is set equal to ``<epics_base>/templates/makeBaseApp/top``

-  If ``-l`` is specified the list of application types is listed and
   makeBaseApp terminates.

-  If ``-i`` is specified and ``-a`` is not then the user is prompted
   for the IOC architecture.

-  The application type is determined by checking the following in
   order:

   -  If ``-t`` is specified it is used.

   -  If ``EPICS_MBA_DEF_APP_TYPE`` is defined its value is used.

   -  If a template ``defaultApp`` exists, the application type is set
      equal to default.

   -  If a template ``exampleApp`` exists, the application type is set
      equal to example.

-  If the application type is not found in ``TEMPLATE_TOP``, makeBaseApp
   issues an error and terminates.

-  If ``Makefile`` does not exist, it is created.

-  If directory ``configure`` does not exist, it is created and
   populated with all the ``configure`` files.

-  If ``-i`` is specified:

   -  If directory ``iocBoot`` does not exist, it is created and the
      files from the template boot directory are copied into it.

   -  For each ``<ioc>`` specified on the command line a directory
      ``iocBoot/ioc<ioc>`` is created and populated with the files from
      the template (with ``ReplaceLine()`` tag replacement, see below).

-  If ``-i`` is NOT specified:

   -  For each ``<app>`` specified on the command line a directory
      ``<app>App`` is created and populated with the directory tree from
      the template (with ``ReplaceLine()`` tag replacement, see below).

Tag Replacement within a Template
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When copying certain files from the template to the new application
structure, makeBaseApp replaces some predefined tags in the name or text
of the files concerned with values that are known at the time. An
application template can extend this functionality as follows:

Two perl subroutines are defined within makeBaseApp:

| ``ReplaceFilename``
| This substitutes for the following in names of any file taken from the
  templates.

::

       _APPNAME_
       _APPTYPE_

| ``ReplaceLine``
| This substitutes for the following in each line of each file taken
  from the templates:

::

       _USER_
       _EPICS_BASE_
       _ARCH_
       _APPNAME_
       _APPTYPE_
       _TEMPLATE_TOP_
       _IOC_

If the application type directory has a file named ``Replace.pl``, this
file may:

-  Replace one or both of the above subroutines with its own versions.

-  Provide a subroutine ``ReplaceFilenameHook($file)`` which will be
   called at the end of the subroutine ``ReplaceFilename`` described
   above.

-  Provide a subroutine ``ReplaceLineHook($line)`` which is called at
   the end of ``ReplaceLine``.

-  Include other code which is run after the command line options have
   been interpreted.

makeBaseApp templetes provided with base
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

support
^^^^^^^

This creates files appropriate for building a support application.

ioc
^^^

Without the ``-i`` option, this creates files appropriate for building
an ioc application. With the ``-i`` option it creates an ioc boot
directory.

example
^^^^^^^

Without the ``-i`` option it creates files for running an example. Both
a support and an ioc application are built. With the ``-i`` option it
creates an ioc boot directory that can be used to run the example.

caClient
^^^^^^^^

This builds two Channel Access clients.

caServer
^^^^^^^^

This builds an example Portable Access Server.

vxWorks boot parameters
-----------------------

The vxWorks boot parameters are set via the console serial port on your
IOC. Life is much easier if you can connect the console to a terminal
window on your workstation. On Linux the ‘screen’ program lets you
communicate through a local serial port; run ``screen /dev/ttyS0`` if
the IOC is connected to ``ttyS0``.

The vxWorks boot parameters look something like the following:

::

   boot device            : xxx
   processor number       : 0
   host name              : xxx
   file name              : <full path to board support>/vxWorks
   inet on ethernet (e)   : xxx.xxx.xxx.xxx:<netmask>
   host inet (h)          : xxx.xxx.xxx.xxx
   user (u)               : xxx
   ftp password (pw)      : xxx
   flags (f)              : 0x0
   target name (tn)       : <hostname for this inet address>
   startup script (s)     : <top>/iocBoot/iocmyexample/st.cmd

The actual values for each field are site and IOC dependent. Two fields
that you can change at will are the vxWorks boot image and the location
of the startup script.

Note that the full path name for the correct board support boot image
must be specified. If bootp is used the same information will need to be
placed in the bootp host’s configuration database instead.

When your boot parameters are set properly, just press the reset button
on your IOC, or use the ``@`` command to commence booting. You will find
it VERY convenient to have the console port of the IOC attached to a
scrolling window on your workstation.

RTEMS boot procedure
--------------------

RTEMS uses the vendor-supplied bootstrap mechanism so the method for
booting an IOC depends upon the hardware in use.

Booting from a BOOTP/DHCP/TFTP server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Many boards can use BOOTP/DHCP to read their network configuration and
then use TFTP to read the applicaion program. RTEMS can then use TFTP or
NFS to read startup scripts and configuration files. If you are using
TFTP to read the startup scripts and configuration files you must
install the EPICS application files on your TFTP server as follows:

-  Copy all ``db/xxx`` files to
   ``<tftpbase>/epics/<target_hostname\>/db/xxx``.

-  Copy all ``dbd/xxx`` files to
   ``<tftpbase>/epics/<target_hostname>/dbd/xxx``.

-  Copy the ``st.cmd`` script to
   ``<tftpbase>/epics/<target_hostname>/st.cmd``.

Use DHCP site-specific option 129 to specify the path to the IOC startup
script.

Motorola PPCBUG boot parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Motorola single-board computers which employ PPCBUG should have their
‘NIOT’ parameters set up like:

| ``Controller LUN =00``
| ``Device LUN     =00``
| ``Node Control Memory Address =FFE10000``
| ``Client IP Address      =``\ ‘Dotted-decimal’ IP address of IOC
| ``Server IP Address      =``\ ‘Dotted-decimal’ IP address of TFTP/NFS
  server
| ``Subnet IP Address Mask =``\ ‘Dotted-decimal’ IP address of subnet
  mask (255.255.255.0 for class C subnet)
| ``Broadcast IP Address   =``\ ‘Dotted-decimal’ IP address of subnet
  broadcast address
| ``Gateway IP Address     =``\ ‘Dotted-decimal’ IP address of network
  gateway (0.0.0.0 if none)
| ``Boot File Name         =``\ Path to application bootable image
  (..../bin/RTEMS-mvme2100/test.boot)
| ``Argument File Name     =``\ Path to application startup script
  (..../iocBoot/ioctest/st.cmd)
| ``Boot File Load Address         =001F0000`` (actual value depends on
  BSP)
| ``Boot File Execution Address    =001F0000`` (actual value depends on
  BSP)
| ``Boot File Execution Delay      =00000000``
| ``Boot File Length               =00000000``
| ``Boot File Byte Offset          =00000000``
| ``BOOTP/RARP Request Retry       =00``
| ``TFTP/ARP Request Retry         =00``
| ``Trace Character Buffer Address =00000000``

Motorola MOTLOAD boot parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Motrola single-board computers which employ MOTLOAD should have their
network ‘Global Environment Variable’ parameters set up like:

| ``mot-/dev/enet0-cipa=``\ ‘Dotted-decimal’ IP address of IOC
| ``mot-/dev/enet0-sipa=``\ ‘Dotted-decimal’ IP address of TFTP/NFS
  server
| ``mot-/dev/enet0-snma=``\ ‘Dotted-decimal’ IP address of subnet mask
  (255.255.255.0 for class C subnet)
| ``mot-/dev/enet0-gipa=``\ ‘Dotted-decimal’ IP address of network
  gateway (omit if none)
| ``mot-/dev/enet0-file=``\ Path to application bootable image
  (..../bin/RTEMS-mvme5500/test.boot)
| ``rtems-client-name=``\ IOC name (mot-/dev/enet0-cipa will be used if
  this parameter is missing)
| ``rtems-dns-server=``\ ’Dotted-decimal’ IP address of domain name
  server (omit if none)
| ``rtems-dns-domainname=``\ Domain name (if this parameter is omitted
  the compiled-in value will be used)
| ``epics-script=``\ Path to application startup script
  (..../iocBoot/ioctest/st.cmd)

The ``mot-script-boot`` parameter should be set up like:

::

   tftpGet -a4000000 -cxxx -sxxx -mxxx -gxxx -d/dev/enet0
           -f..../bin/RTEMS-mvme5500/test.boot
   netShut
   go -a4000000

where the ``-c``, ``-s``, ``-m`` and ``-g`` values should match the
cipa, sipa, snma and gipa values, respectively and the ``-f`` value
should match the file value.

RTEMS NFS access
~~~~~~~~~~~~~~~~

For IOCs which use NFS for remote file access the EPICS initialization
code uses the startup script pathname to determine the parameters for
the initial NFS mount. If the startup script pathname begins with a
‘``/``’ the first component of the pathname is used as both the server
path and the local mount point. If the startup script pathname does not
begin with a ‘``/``’ the first component of the pathname is used as the
local mount point and the server path is “``/tftpboot/``” followed by
the first component of the pathname. This allows the NFS client used for
EPICS file access and the TFTP client used for bootstrapping the
application to have a similar view of the remote filesystem.

RTEMS ‘Cexp’
~~~~~~~~~~~~

The RTEMS ‘Cexp’ add-on package provides the ability to load object
modules at application run-time. If your RTEMS build includes this
package you can load RTEMS IOC applications in the same fashion as
vxWorks IOC applications.
