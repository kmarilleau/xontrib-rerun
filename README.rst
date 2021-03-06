xontrib-rerun
===============================

version number: 0.1.01
author: Kevin Marilleau

Overview
--------

Rerun previous commands

Installation
------------

To install use pip:

    $ xpip install xontrib-xontrib-rerun


Or clone the repo:

    | $ git clone https://github.com/kmarilleau/xontrib-rerun.git
    | $ xpip install ./xontrib-rerun

Usage
-----

    | $ rerun [--edit <editor>] [--startswith <sw>] [--contain <ct>]...
    | $       [--replace <old=new>]... [--confirm]
    | $ rerun LEN [OFFSET] [options] [-C <ct>]... [-R <old=new>]... [-c | --confirm-each]
    | $ rerun (-h | --help)
    | $ rerun rerun [<index>] [--confirm | --confirm-each]
    | $ rerun rerun list
    | $ rerun (-h | --help)

Commands
--------
    | $ list                     View current history.
    | $ ls                       Alias for ``list``.
    | $ rerun                    Re-execute commands that have already been executed.

Arguments
---------
    | $ LEN                      Number of commands to re-execute.
    | $ OFFSET                   Re-execute commands before n last commands.
    | $ INDEX                    Index of commands in ``rerun rerun list``.

Options
-------
    | $ -h, --help               Do we really need to explain?
    | $ -c, --confirm            View commands and confirm their execution.
    | $ --confirm-each           Confirm the execution of each command.
    | $ -e, --edit <editor>      Edit commands.
    | $ -C, --contain <ct>       Rerun from last command containing <ct>.
    | $ -S, --startswith <sw>    Rerun from last command starting with <sw>.
    | $ -R, --replace <old=new>  Replace all substring in selected commands.
    | $ --eq                     Rerun only if len of selected commands == LEN.


Description
-----------

    Let's assume that our history looks like this:
        >>> git init myproject && cd myproject
        >>> touch Dockerfile \\
        ...   && echo 'FROM nginx' >> Dockerfile \\
        ...   && echo 'WORKDIR /usr/share/nginx/html/' >> Dockerfile \\
        ...   && echo 'RUN echo "<h1>Foo</h1>" > index.html' >> Dockerfile
        >>> docker build . -t last_build
        >>> docker run --rm -it -p 8080:80 last_build
    We created a Dockerfile, we built a docker image with it. Then we runned a 
    container that uses this image.
    
    |  Note:
    |      All these commands work, you can launch them on your shell and then see 
    |      the result from your browser at http://localhost:8080.
    
    If we decide to stop the container with Ctrl+C and recreate the same container, 
    many shells offer to find and re-execute the last command by press 'Up Arrow' 
    then 'Enter'. Otherwise, you can do the same thing by simply typing:
        >>> rerun
    We stop our container with Ctrl+C, we run the commands below to check that 
    our container is no longer running:
        >>> docker ps
    Let's add the Dockerfile to our project and then make our first commit:
        >>> git add Dockerfile
        >>> git commit -am "create Dockerfile"
    Now let's suppose that we need to modify the Dockerfile:
        >>> echo 'RUN echo "<h1>Bar</h1>" >> index.html' >> Dockerfile
    
    If we run ``rerun list`` or ``rerun ls``, We can see our history look like this:
        10: rerun ls
        09: git init myproject && cd myproject
        08: touch Dockerfile \\
              && echo 'FROM nginx' >> Dockerfile \\
              && echo 'WORKDIR /usr/share/nginx/html/' >> Dockerfile \\
              && echo 'RUN echo "<h1>Foo</h1>" > index.html' >> Dockerfile
        07: docker build . -t last_build
        06: docker run --rm -it -p 8080:80 last_build
        05: rerun
        04: docker ps
        03: git add Dockerfile
        02: git commit -am "create Dockerfile"
        01: echo 'RUN echo "<h1>Bar</h1>" >> index.html' >> Dockerfile
    
    To see the result in our browser, we need to re-execute commands below:
        >>> docker build . -t last_build
        >>> docker run --rm -it -p 8080:80 last_build 
    
    To do this, we just have to run:
        >>> rerun 2 5
    It means:
        "Restarts the two commands before the last five commands that have been 
         executed."
    If we want to see selected commands before and confirm their execution, we only 
    need to add '--confirm' option:
        >>> rerun 2 5 --confirm
    If we want to see and confirm execution of each command, we can do this:
        >>> rerun 2 5 --confirm-each
    Let's update our Dockerfile in our git branch, make another commit, then view 
    and confirm results commands:
        >>> rerun 2 -S 'git add' --replace create=update -c
    It means:
        "Restart the two commands from the first latest command starting with 
        'git add', replace all 'create' substrings by 'update' then show me the 
        result and let me confirm execution"
    The commands to be executed should now look like this:
        >>> git add Dockerfile
        >>> git commit -am "update Dockerfile"
    
    If instead we wanted to change the name of the Dockerfile by Mydockerfile, make 
    a commit, then view and confirm commands, do:
        >>> rerun 2 -c -S 'git add' \\
        ...            -R create=rename \\
        ...            -R 'add Dockerfile'='mv Dockerfile Mydockerfile'
    
    |  Note:
    |      You can use single/double quotes and even none with ``--replace`` 
    |      arguments. Just remember to escape quotes from your sub-chains.
    |      See examples below:
    |     
    |      >>> rerun --replace old="new"
    |      >>> rerun --replace "For what it's worth"='Buffalo Springfield'
    |      >>> rerun --replace TVSeries='That\\'s 70 Show'
    
    As you can see, you can replace as many substrings as you want. 
    But, you can quickly find it tiring to make all these changes with the command 
    line. You can use an editor with --edit instead of --replace:
    
    To use it, simply add the option --edit (or -e):
        >>> rerun 2 -c -S 'git add' --edit
    Modify your text and quit the editor. 
    
    |  Warning:
    |      All comments are deleted when you exit the text editor.
    
    If you have not specified one, rerun use $RERUN_EDITOR as default editor. And 
    $RERUN_EDITOR points directly to $EDITOR if you have not assigned a value to it.
    For example, if you want to use nano as default editor for rerun, add 
    ``$RERUN_EDITOR = nano`` to your .xonshrc.
    Now let's imagine that we need to re-execute commands that have already been 
    re-executed. Let's start by listing the commands:
        >>> rerun rerun list
        4: docker run --rm -it -p 8080:80 last_build
        3: docker build . -t last_build
           docker run --rm -it -p 8080:80 last_build
        2: git add Dockerfile
           git commit -am "update Dockerfile"
        1: git mv Dockerfile Mydockerfile
           git commit -am "rename Dockerfile"
    
    If you need to re-execute the last re-executed commands, just run:
        >>> rerun rerun
    Else, just specified index of commands:
        >>> rerun rerun 3
    Like the rerun command, you can use modifiers and confirm arguments like 
    ``--replace``, ``--edit``, ``--confirm`` or ``--confirm-each``:
        >>> rerun rerun 3 -e nano -c

Credits
---------

This package was created with cookiecutter_ and the xontrib_ template.

.. _cookiecutter: https://github.com/audreyr/cookiecutter
.. _xontrib: https://github.com/laerus/cookiecutter-xontrib
