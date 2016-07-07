Simple Keyring In Perl (Skip)
=============================

1. Introduction

    Skip is a lightweight and portable tool for managing secrets such as
    login passwords and easily automating the programs that use them
    without leaving the secrets unencrypted on disk.  Skip has been
    tested successfully with several common programs including
    fetchmail, getmail, msmtp, scp, ssh, and ssh-add without the need
    for any modification on Linux, OSX, and Windows under Cygwin.

    The security profile of Skip is similar to that of other agent-based
    authentication stores such as ssh-agent and gpg-agent.  Namely, when
    an agent is running, any user who can access the agent's socket
    (e.g. root or an attacker who gains access to the user's account)
    can utilize that agent for authentication purposes.  While the
    secrets in a Skip agent are easier to extract directly (since its
    purpose is to provide unencrypted secrets to programs), the
    effective security is still similar since even stores such as
    ssh-agent that do not provide direct extraction of private keys
    are subject to extraction via memory analysis:

        http://c0decstuff.blogspot.com.au/2011/01/in-memory-extraction-of-ssl-private.html

    Hence, like all such programs, the main benefit of Skip is to
    protect secrets at rest (i.e. when not actively on the system using
    a Skip agent) and to prevent unencrypted secrets from being backed
    up to untrusted locations (e.g. the cloud) even when the agent is
    running.  Note that it is possible for the agent's memory to be
    written out to disk if the agent process is swapped out of main
    memory or if the system goes into hibernation (e.g. when running on
    a laptop) but these are not generally-accessible files.


2. Installation

    Skip requires Perl >= 5.6 and the non-standard modules:
    
        o Crypt::CBC
        o Crypt::Rijndael
        o Expect
        o IO::Multiplex
        o Net::Server
        o Term::ReadKey

    To install, simply place the "skip" executable somewhere accessible
    in your $PATH (e.g. ~/bin for single users or /usr/local/bin for
    multiple users):

        cp skip /usr/local/bin
    
    Skip stores encrypted secrets in the file ~/.skip.db, which can be
    changed with the -d option, and the Skip agent uses a unix socket
    ~/.skip.sock, which can be changed with -s.


3. Usage

    Skip supports the following options (defaults in brackets):

        Options (defaults in brackets):
            -a       start agent
            -d FILE  use FILE for storing data [~/.skip.db]
            -e EXPR  run COMMAND and reply to EXPR with secret of -q name
            -h       help
            -i NAME  insert secret (prompted) for NAME
            -q NAME  query stored secret for NAME
            -r NAME  remove secret for NAME
            -s FILE  use FILE for socket [~/.skip.sock]
            -t SECS  abort COMMAND/exit agent after TIME seconds [15/0]
            -x       direct agent to exit

    3.1. Insert/remove secret

        Each secret is referenced by a chosen name and inserted using -i:

            skip -i secret_name

        For example, to store the password to your email account under
        the name "email":

            skip -i email
            Enter db password:
            Enter email secret:
            Secret email stored

        Secrets can be removed using -r:

            skip -r secret_name

        For example, to remove the previously stored email secret:

            skip -r email
            Enter db password:
            Secret email removed

    3.2. Start/stop agent

        An agent process keeps decrypted secrets in memory so it can
        provide them to programs that need them.  The agent is started
        using -a:

            skip -a
            Enter db password:
            Agent started

        Skip will detect if an agent is already running so this
        command can be safely used in login scripts and shell rc files:

            skip -a
            Agent already running

        To stop the running agent, use -x:

            skip -x
            Agent killed

        The -t option can be used to start an agent that will
        automatically exit after a given number of seconds:

            skip -a -t seconds
            
        For example, to start an agent that will exit after 8 hours:
        
            skip -a -t 28800
            Enter db password:
            Agent started

    3.3. Query secret

        Stored secrets can be queried from a running agent using -q:

            skip -q secret_name

        For example, to query the previously stored email secret:

            skip -q email
            yourpassword

        Note that no password is needed to query secrets.

    3.4. Run command and provide secret for prompt

        Skip can automatically provide stored secrets to commands that
        require them when given an expression to match via -e, the
        secret to provide via -q, and the command to run:

            skip -e match_expression -q secret_name command_to_run

        Note that Skip will only match a single prompt until the program
        runs to completion.

        For example, to use the previously stored email secret with
        fetchmail:

            skip -e 'Enter password' -q email fetchmail
            Enter password for you@example.com@mail.example.com:
            100 messages (100 seen) for you@example.com at mail.example.com.


4. Example Use Cases

    All the following examples assume the relevant secret for each
    command is stored under the name "X".

    4.1. msmtp

        Use the "passwordeval" setting in ~/.msmtp:

            account X
                passwordeval skip -q X
                ...

    4.2. fetchmail/getmail

        skip -e 'Enter password' -q X getmail
        skip -e 'Enter password' -q X fetchmail

    4.3. ssh/scp

        skip -e 'password:' -q X ssh host
        skip -e 'password:' -q X scp file host:

    4.4. ssh-add

        skip -e 'Enter passphrase' -q X ssh-add


Questions, comments, fixes, and/or enhancements welcome.

--Paul Kolano <pkolano@gmail.com>

