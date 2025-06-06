This document describes the detailed usage of `hkml`.  This doesn't cover all
details of `hkml` but only major features.  This document may not complete and
up to date sometimes.  Please don't hesitate at asking questions or help for
this document via GitHub issues.

Demo
====

![interactive list](images/hkml_interactive_list_demo.gif)
[![asciicast](https://asciinema.org/a/632442.svg)](https://asciinema.org/a/632442)

Note that the above demos are recorded on an old version of `hkml`.  Latest
version may have different features/interface.

Pre-requisites
==============

To use full features of hackermail, your system should be able to use below
commands.

- `git`: Required for reading mails from the mailing list.
- `git send-email`: Required for sending mails.

Commands
========

Hackermail can be executed using `hkml` command.  Using sub-command of it,
users can manipulate mails.  Users can show the list of sub-commands via the
`--help` option of `hkml`.  Similarly, each subcommand support `--help` option.

Initialization
==============

To properly work, hackermail requires 1) working directory, and 2) mailing
lists manifest.

Working Directory
-----------------

Working directory is a directory to save the fetched mails and hackermail's
metadata.  You may think this as something similar to `.git` directory of git.

You can explicitly set the path to the directory using `HKML_DIR` environment
variable, or `--hkml_dir` option of each sub-command.  If the path is not
specified, hackermail assumes the directory is named as `.hkm` and placed under
current directory, the `hkml` executable file placed directory, or your home
directory and try to find it.  If it cannot find a proper working directory,
most of `hkml` sub-command will fail.

Manifest File
-------------

Manifest file is a file for information on mailing lists that you want to
communicate with using `hkml`.  It describes from where in the internet the
mails you want to read can be fetched, name of the mailing lists archived in
the site, and the site-relative path to the git repositories for each mailing
list in json format.  It's very similar to that of lore[1] except the fact that
hackermail manifest is containing the site information.  A sample manifest file
for the linux kernel mailing lists can be generated as a file using
`hkml manifest fetch_lore --fetch_lore_output <file path>`.

You can explicitly set the path to the manifest file using `--manifest` option
of each sub-command.  If it is not specified, hackermail assumes it is placed
under the working directory in name of `manifest` and try to use it.

[1] https://www.kernel.org/lore.html

`init` sub-command
------------------

`init` sub-command of `hkml` does setting the working directory and manifest
file.  For example, if you call below command on this directory, it will create
the working directory as `.hkm` directory under the repo, and Linux kernel
mailing list as the manifest.  Hence, you will be able to use `hkml` for Linux
kernel mails from anywhere by using `hkml` file in this directory.

```
$ hkml manifest fetch_lore --fetch_lore_output ./lore.json
$ hkml init --manifest ./lore.json
```

If `--manifest` is not given, `hkml` will ask if the user wants to use `hkml`
for Linux kernel development, and set the manifest for lore.kernel.org if so.

Fetching Mails
==============

Users can download mails from specific mailing list that described on the
manifest file onto the local storage using `fetch` sub-command.  It receives
the names of the mailing lists to fetch.  By default, it receives latest
epoch's mails of given mailing lists.  For example, below command downloads
recent mails from linux-mm mailing list.

```
$ hkml fetch linux-mm
```

Note that fetching can be done with `list` sub-command, which will be described
below.  In some use cases, `fetch` sub-command may not frequently used.

Listing Mails
=============

Users can list mails of specific mailing list on the manifest file via `list`
sub-command.  By default, it lists the downloaded mails of the mailing list
that sent within last three days.  Users can let the command to do downloading
of the mails together via `--fetch` option.  The sent time range of the mails
to list can be adjusted using `--since` and `--until` options.

In addition to the sent dates range, users can filter mails on the list by
author of the mail, keywords on subject or body, whether those are newly
started threads, via `--author`, `--subject_contains`, `--contains`, and
`--new` options, respectively.

The list shows all mails grouped by threads and sort the threads by sent date
of the latest mail of each thread.  Mails in each thread are sorted in the
reply order.  Users can show only first mail of each thread using `--collapse`
option.  It could be useful for mailing lists that people send huge amount of
mails every day.  Threads sorting key can also be customized using
`--sort_threads_by` option.  `--descend` option makes the sorting be done in
descendent order.  `--hot` option is a short cut for sorting threads by number
of comments in descendent way.

Below example lists mails that sent to [DAMON](https://damonitor.github.io/)
mailing list from 2024-02-15 to 2024-02-17.

```
$ hkml list damon --fetch --since 2024-02-15 --until 2024-02-17 --min_nr_mails 0
# 9 mails, 2 threads, 2 new threads
# 9 patches, 2 series
# oldest: 2024-02-16 11:40:23-08:00
# newest: 2024-02-16 16:58:42-08:00
[0] [PATCH 0/5] Docs/mm/damon: misc readability improvements (SeongJae Park, 24/02/16 16:58)
[1]   [PATCH 1/5] Docs/mm/damon/maintainer-profile: fix reference links for mm-[un]stable tree
      (SeongJae Park, 24/02/16 16:58)
[2]   [PATCH 2/5] Docs/mm/damon: move the list of DAMOS actions to design doc (SeongJae Park,
      24/02/16 16:58)
[3]   [PATCH 3/5] Docs/mm/damon: move DAMON operation sets list from the usage to the design
      document (SeongJae Park, 24/02/16 16:58)
[4]   [PATCH 4/5] Docs/mm/damon: move monitoring target regions setup detail from the usage to
      the design document (SeongJae Park, 24/02/16 16:58)
[5]   [PATCH 5/5] Docs/admin-guide/mm/damon/usage: fix wrong quotas diabling condition
      (SeongJae Park, 24/02/16 16:58)
[6] [PATCH 0/2] mm/damon: fix quota status loss due to online tunings (SeongJae Park, 24/02/16
    11:40)
[7]   [PATCH 1/2] mm/damon/reclaim: fix quota stauts loss due to online tunings (SeongJae
      Park, 24/02/16 11:40)
[8]   [PATCH 2/2] mm/damon/lru_sort: fix quota status loss due to online tunings (SeongJae
      Park, 24/02/16 11:40)
```

The output maybe intuitive to understand.  The first column of the each mail is
called index or identifier of the mail, and be used by other sub-commands that
will be described below.

The output is shown by `hkml`'s interactive viewer by default.  Refer to the
[section](#interactive-viewer) for details.

Supporting Mail Source Types
----------------------------

The sub-command support not only mailing lists on the manifest file, but more
sources.  All types of the supported sources of mails are as below.

- mailing lists.  Name of the mailing lists on the manifest can be passed.
- Message-Id of a mail of a thread that the user wants to list.
- mbox files.  Paths to the mbox files of the mails can be passed.
- Special keyword, 'clipboard'.  If this keyword is passed as source of mails,
  the command assumes users have copied mbox-format string of the mails to list
  on the clipboard, read the clipboard, and show the mails.
- [`hkml tag`](#tagging)-added tags.  Users can add arbitrary tags to specific
  mails using `hkml tag` command, which explained below.  The tags can be used
  here.
- Nothing.  If no source of mails is given, `hkml list` shows the
  last-generated list output again.

`hkml list` tries to infer the type of given source with its best effort.  If
it cannot find the type of there are multiple possible types, the command will
fail with an error message.  To avoid such failures, users can provide the type
of the given source via `--source_type` option.

Patches Filtering
-----------------

Note: this feature is experimental.

Users can filter mails to list only patch mails of specific condition, using
`--patches_for` option of `hkml list`.

`--patches_for review` shows patch mails that not yet received any
`Reviewed-by:` tag reply.

`--patches_for pick` shows patch mails that received one or more `Reviewed-by:`
tag replies.

`--patches_for reviewer <reviewer>` shows patch mails that touching
files that `<reviewer>` is maintaining or revieweing, following `MAINTAINERS`
file of Linux kernel.  In this case, `hkml list` should be executed from a
directory having `MAINTAINERS` file.   `<reviewer>` should be the maintainer or
reviewer identity that written in the `MAINTAINERS` file.  For example, 

    $ hkml list damon --patches_for reviewer "SeongJae Park <sj@kernel.org>" \
                      --stdout --fetch
    # stat for total mails
    # 66 mails, 17 threads, 17 new threads
    # 27 patches, 14 series
    # oldest: 2025-04-08 00:55:53-07:00
    # newest: 2025-05-24 09:29:10-07:00
    #
    # stat for filtered mails
    # 2 mails, 2 threads, 2 new threads
    # 2 patches, 2 series
    # oldest: 2025-04-08 00:55:53-07:00
    # newest: 2025-05-21 00:07:47-07:00
    [03] [PATCH] mm/damon: make region calculations more precise (Enze Li, 2025/05/21
         00:07)
    [50] [RFC PATCH] mm/damon: add full LPAE support for memory monitoring above 4GB (Ze
         Zuo, 2025/04/08 00:55)

Public-inbox Search
-------------------

Note: this feature is experimental.

`hkml list` provides only simple filtering options.  In case of public-inbox
based mails fetching setup, users can use the sophisticated search query of
public-inbox using `--pisearch` option.  For example:

```
$ hkml list damon --pisearch "foo AND range damos filter"
# 2 mails, 2 threads, 0 new threads
# 2 patches, 0 series
# oldest: 2023-07-28 13:34:35-07:00
# newest: 2023-08-02 14:43:03-07:00
[0] [PATCH 04/13] selftests/damon/sysfs: test address range damos filter (SeongJae Park,
    2023/08/02 14:43)
[1] [RFC PATCH 04/13] selftests/damon/sysfs: test address range damos filter (SeongJae Park,
    2023/07/28 13:34)
```

Interactive Viewer
==================

![interactive list](images/hkml_interactive_list_demo.gif)

(Note that the above gif is recorded on an old version of `hkml`.  Latest
version may have different features/interface.)

Outputs of `list`, `thread`, and `open` are shown by `hkml`'s interactive
viewer by default.  The viewer maintains focus on a line of the content.  Users
can move the focus and make the focused-line's context-based actions using
short cut keys.  The actions include moving focus, open/forwarding a mail,
replying to a mail, managing tags of a mail, opening selection-based menu, etc.

Default Key Bindings
---------------------

Pressing '?' key shows available key bindings of current screen.  The available
key bindings and context-based menu items depend on the content of the screen
and focused line.

Regardless of the type, below keys are supported.

`j` and `k` key move focus down and up one row.  `J` and `K` keys move focus up
and down 1/2 screen, respectively.

Pressing `m` key opens context-based menu that shows items that can be
selected, respectively.

`:` key opens input window to enter the line number to move focus to.  `start`
and `end` keyword could also be used, to specify the start and end line of the
content.

`/` key starts keyword searching.  `n` and `N` keys move the focus to
next/previous line containing the keyword.

`h` and `l` keys scroll the screen left and right one column, respectively.

`q` key quits current screen.  `Q` key quits `hkml` at once.

Mails List Screen
-----------------

If the content of the screen is mails list (`hkml list` or `hkml thread`
output), actions for mail of the focused line becomes available.

Pressing `o` or `<Enter>` key opens the focused mail's content.  The content is
shown with a text screen viewer [type](#text-screen).

Pressing `t` lists the complete thread of the focused mail.  The list is shown
with mails list screen [type](#mails-list-screen).

Pressing `c` collapses the thread starting from the focused mail.

Pressing `e` expands the collapsed thread again.

Pressing `r` starts writing a reply to the focused mail.

Pressing `f` starts forwarding the focused mail.

Pressing `m` or opens a selection-based menu.  From the menu, users can do
abovely explained actions.  In addition to those, the menu provides below
options.

- Writing a mail using the focused mail as its draft (same to `hkml write
  --draft <focused mail>`.  Refer to 'Writing New Mails'
  [section](#writing-new-mails) for details.).
- Listing/adding/removing tags (Refer to 'Tagging' [section](#tagging) for
  details) of the mail.
- Checking/Applying patches (same to `hkml patch check <focused mail>`.  Refer
  to 'Patches Management' [section](#patches-management) for details.)
- Refreshing the mails list.
- Exporting focused mail, specific range of mails of the list, or entire mails
  of the list to an mbox file (same to `hkml export`.  Refer to 'Exporting
  Mails' [section](#exporting-mails) for details.).
- Save the content of the list as a text to a file, or the clipboard.

Text Screen
-----------

If the screen is showing a general text (`hkml open` output or mail content
opened from `hkml list` output), the screen is called text screen type.

From this type, the `r` and `f` key bindings for replying and forwarding mail
are supported, if the screen is displaying a mail.  Also, the menu (`m` key) on
this screen shows below options.

- Saving the text to a file, or the clipboard.
- If the focused line contains git commit ids, the menu shows items for showing
  outputs from `git show` or `git log` of the commit ids.  The outputs are
  shown as general text.
- If the focused line contains public-inbox mail links (e.g.,
  https://lore.kernel.org/example-messgae-id@org), the menu shows items for
  opening the mail or listing the thread of the mail.  The text screen and
  mails list screen are used for opening the mail and listing the thread
  options, respectively.
- If the focused line contains paths to files, the menu shows items for opening
  the files by `hkml` or `vim`.
- If currently showing content is a mail, the menu shows below additional items
  that similar to those of mails list screen's menu.
  - replying
  - forwarding
  - continue draft writing
  - managing tags
  - handle as patches

Note that `hkml open` [allows](#reading-more) opening normal text files, git
commits, and given command's outputs.  Hence the commit ids, public-inbox
links, and file paths based features can help browsing of commit history,
related discussions, and source files.

Writing New Mails
=================

Users can write new mail in a way similar to `reply` sub-command, using `write`
sub-command.  Users can specify subject, recipients, Cc list, etc via command
line.  Then `write` sub-command formats the basic mail, and let the user
additionally make more edits interactively and finally send it, in a way pretty
similar to that of `reply`.  Again, `git send-email` setup is required.

Users can write a mail using another mail as the original draft, using
`--draft` option.  The option receives a mail identifier from the previously
generated `list` or `thread` output.  Then, it format the content of the mail
as same to the specified mail, and allow user continue writing and sending it.

For example,

```
$ hkml reply 3
[...] # hkml reply open VIM.  Below is an example output after closing VIM.

Will send above mail.  Okay? [y/N] n
Tag as drafts? [Y/n] y
$ hkml tag list
[...]
drafts: 5 mails
$ hkml list drafts
[...]
$ hkml write --draft 0
[...] # hkml write open VIM.  Below is an example output after closing VIM.
Will send above mail.  Okay? [y/N] n
Tag as drafts? [Y/n] y
```

Synchronizing
=============

Users may use `hkml` on multiple devices or need to change their devices.  In
such cases, users can backup and restore some `hkml` settings and outputs via
remote storage using `sync` command.  Currently supported data includes

- manifest (created via `hkml init`),
- monitoring request (created via `hkml monitor`), and
- mails tagging information (created via `hkml tag`).

The command backs up and synchronizes the data using remote `git` repository.
Users therefore need to first create a `git` repository that accepts pushing
commits from the `hkml`-running machine.  The backup repo could be on the local
storage, a private server, or public git hosting services like `Gitlab` or
`GitHub`.  Then, the user could do the synchronization by running `hkml sync`.
For the first time of synchronization, user should provide the valid `git` url
for the backup repo to `hkml sync` via `--remote` option.

Below example shows how the mails tagging information can be synchronized
between two different machines using a GitHub repo.

```
$ # from the first machine
$ ssh machine_1
machine_1:~$ hkml tag list
to_read_later: 3 mails
to_review_later: 3 mails
to_test_later: 4 mails
machine_1:~$ hkml sync --remote git@github.com:$USER/hkml_backup_repo.git
$
$ # move to the second machine
$ ssh machine_2
machine_2:~$ hkml tag list
machine_2:~$
machine_2:~$ hkml sync --remote git@github.com:$USER/hkml_backup_repo.git
[...]
machine_2:~$ hkml tag list
to_read_later: 3 mails
to_review_later: 3 mails
to_test_later: 4 mails
```

Signatures
==========

Users can set and list their mail signatures using `hkml signature` command.
The command supports `list`, `add`, `edit`, and `remove` actions.  Please use
`-h` options for more details.

The signature is set as `Sent using hkml (https://github.com/sjp38/hackermail)`
by default.  Users can remove, edit, add their signatures using `hkml
signature` command.  If there are one or more signatures, those are
automatically added to mail drafts.  To avoid adding the default signature on
the mail drafts, users should explicitly remove it via `hkml signature remove`.

Monitoring Mails
================

Users can monitor new mails to specific mailing lists using `hkml monitor`
command.  Using this, users can get periodic notification of summary or updates
on specific mail threads that the user is not already in the loop, without
subscribing to the mailing list.  It works with sub-sub commands, `add`,
`status`, `remove`, and `start`.

`hkml monitor add`
------------------

`add` command adds monitoring request.  Users can specify
- the mailing lists to monitor,
- How often the monitoring should be done,
- what mails should be filtered in/out from the new mails,
- how the monitored mails should be displayed, and
- how the monitoring result should be delivered to users via command line
  options.

The command line options for specifying what mails to filtered and how those
should be displayed are very similar to that of `list` command.

It provides the monitoring results via the terminal with some execution logs by
default, but users can asks it to send the new findings via sending emails to
specific addresses, or writing to specific files.

For example, users ask `hkml` to monitor updates to the `DAMON Beer/Coffee/Tea
chat series`
[thread](https://lore.kernel.org/r/20220810225102.124459-1-sj@kernel.org) for
every hour, format notification text with lore links for found new mails, and
send the notification text to their personal email address, like below:

```
$ hkml monitor add damon \
    --subject_keywords "DAMON Beer/Coffee/Tea chat series" \
    --monitor_interval 3600 --lore --noti_mails $YOUR_EMAIL_ADDRESS \
    --name "DAMON chat series monitoring"
```

`hkml monitor remove`
---------------------

Remove added monitoring requests.  User can specify the request to remove using
the index of the request on the requests list that can be shown via `hkml
monitor status`, or the name of the request which the user can set with `hkml
monitor add` command.

`hkml monitor status`
---------------------

Show status of the monitoring, including list of currently added monitoring
requests.

`hkml monitor start`
--------------------

Start the requested monitoring.  Monitoring requests that added after
monitoring is started is not automatically added to the running instance.  You
should start a new instance for new requests.  To stop running instance, you
can simply Ctrl-C.

Listing Entire Thread of a Given Mail
=====================================

On huge mailing list, reading all `hkml list` output takes time.  Also, because
`hkml list` lists mails that sent in user-specified time range (last three days
by default), some old mails of some threads may not listed.  Users can list all
mails of a specific threads containing a specific mail using `thread`
sub-command.  The mail of the thread can be specified by passing the mail
identifier of the mail to the sub-command.  The mail identifier should be that
of previously generated list, or the message-id.

For example, below command shows the thread of the third mail on the above
`hkml list` output.

```
$ hkml thread 3
# 6 mails, 1 threads, 1 new threads
# 6 patches, 1 series
# oldest: 2024-02-16 16:58:37-08:00
# newest: 2024-02-16 16:58:42-08:00
[0] [PATCH 0/5] Docs/mm/damon: misc readability improvements (SeongJae Park, 24/02/16 16:58)
[1]   [PATCH 1/5] Docs/mm/damon/maintainer-profile: fix reference links for mm-[un]stable tree
      (SeongJae Park, 24/02/16 16:58)
[2]   [PATCH 2/5] Docs/mm/damon: move the list of DAMOS actions to design doc (SeongJae Park,
      24/02/16 16:58)
[3]   [PATCH 3/5] Docs/mm/damon: move DAMON operation sets list from the usage to the design
      document (SeongJae Park, 24/02/16 16:58)
[4]   [PATCH 4/5] Docs/mm/damon: move monitoring target regions setup detail from the usage to
      the design document (SeongJae Park, 24/02/16 16:58)
[5]   [PATCH 5/5] Docs/admin-guide/mm/damon/usage: fix wrong quotas diabling condition
      (SeongJae Park, 24/02/16 16:58)
```

The command downloads whole mails of the thread from the internet and shows the
list.  If the system is not able to download the mails from the internet, only
the mails of the thread in the previously generated list is listed.  Note that
the mail indices are newly generated when mails are downloaded from the
internet.

The output is shown by `hkml`'s interactive viewer by default.  Refer to the
[section](#interactive-viewer) for details.

Reading Mails
=============

Users can open the content of the mail using `open` subcommand.  It receives
the identifier of the mail to read.  The identifier should be that of the last
generated `list` or `thread` output.

For example, below command shows the 18-th mail of the above list.

```
$ hkml open 3
Local-Date: 2024-02-16 16:58:40-08:00
Date: Fri, 16 Feb 2024 16:58:40 -0800
Subject: [PATCH 3/5] Docs/mm/damon: move DAMON operation sets list from the usage to the design document
Message-Id: <20240217005842.87348-4-sj@kernel.org>
From: SeongJae Park <sj@kernel.org>
To: Andrew Morton <akpm@linux-foundation.org>
CC: SeongJae Park <sj@kernel.org>, Jonathan Corbet <corbet@lwn.net>, damon@lists.linux.dev, linux-mm@kvack.org, linux-doc@vger.kernel.org, linux-kernel@vger.kernel.org

The list of DAMON operation sets and their explanation, which may better
to be on design document, is written on the usage document.  Move the
detail to design document and make the usage document only reference the
design document.

Signed-off-by: SeongJae Park <sj@kernel.org>
[...]
```

The output is shown by `hkml`'s interactive viewer by default.  Refer to the
[section](#interactive-viewer) for details.

Reading More
------------

`open` subcommand supports not only mails from `list` or `thread` output, but
also normal text files, git commit, and commands.  If the given target is a
command, `open` executes the command and shows the output.  For example, below
commands work.

```
$ hkml open ./foo-bar.patch
$ hkml open HEAD~2
$ hkml open "git log -10"
```

Since the command shows the content with `hkml`'s interactive viewer, which
allows easy browsing of commits and public-inbox mail links, this can be useful
for browsing git history and related discussions for given changes.

Tagging
=======

Some classification of mails, e.g., important, unread, to-read-next, etc, can
be helpful for managing flooding mails.  Users can add, remove, or list tags
for mails using 'hkml tag'.  The command receives a sub-command for the three
different actions.

`hkml list` supports tags as source of the mails to list up.  Users can
therefore use `hkml list` to list mails of a specific tag.  Below example lists
mails of a tag `tag_damon`.

```
$ hkml list tag_damon
[...]
[0] Re: [PATCH 1/2] mm/damon/sysfs: Implement recording feature (cuiyangpei, 24/02/05 19:26)
[1] [PATCH] mm/damon/sysfs: handle 'state' file inputs for every sampling interval if possible
    (SeongJae Park, 24/02/05 18:51)
```

`hkml tag add`
--------------

Add tags to a specific mail.  The mail could be specified with the index of the
mail from the last-generated list or thread output.  Tags can be any arbitrary
string.  Below example adds three tags namely `tag_example`, `damon_patch`, and
`not_yet_merged` to the fifth mail of the last-generated list output.

```
$ hkml tag add 5 tag_example damon_patch not_yet_merged
```

`hkml tag list`
---------------

Users can show generated tags and how many mails of the tag exists, using `hkml
tag list` command like below:

```
$ hkml tag list
tag_damon: 2 mails
tag_example: 1 mails
damon_patch: 1 mails
not_yet_merged: 1 mails
```

Users can show the list of mails for specific tags using `hkml list` by
providing the tag name as the source of mails.

`hkml tag remove`
-----------------

Users can remove tags from a specific mail using `hkml tag remove` command.
The mail to remove the tags from can be specified by the index of the mail from
the last-generated list or thread output.  Below example removes tag
`tag_damon` from the 0-th mail on the list and confirm it has removed using
`hkml tag list`.

```
$ hkml tag remove 0 tag_damon
$ hkml tag list
tag_damon: 1 mails
tag_example: 1 mails
damon_patch: 1 mails
not_yet_merged: 1 mails
```

Replying
========

Users can reply to specific mail on the last generated list, using `reply`
sub-command.  Similar to `thread` and `open`, it receives the identifier of the
mail to reply for, on the last generated `list` or `thread` output.

The command formats reply mail for the given mail and open VIM for the user's
interactive writing of the content.  Once the user finishes writing it and
close the editor, the command will show the content of the written reply mail
and ask it is ok to send the mail.  If the user answers the mail is correct and
ok to send, `reply` sub-command will send it.

For example,

```
$ hkml reply 3
[...] # hkml reply open VIM.  Below is an example output after closing VIM.

Will send above mail.  Okay? [y/N] n
```

Note that `hkml` send mail using `git send-email`.  Hence `git send-email`
should be configured correctly on the user's system.

Drafts
------

Regardless of users answer for the sending mail confirmation question, the
command further asks if the user wants to tag and save the written content as
`drafts`.  Like other drafts management, it is for a case that the user wants
to continue writing the mail later.

Hackermail tags and saves it as a draft by converting the written content to a
normal mail, and add a [tag](#tagging).  The name of the tag becomes `sent` or
`drafts`, depending on if the user approved posting the mail.  Hence users can
manage the drafts as usual tagged mails.  Refer to [Tagging](#tagging) section
for the detail.  The user can continue writing and sending the drafts using
[`write`](#writing-new-mails) command.

Forwarding
==========

Users can forward a specific mail on the last generated list, using 'forward'
sub-command.  Similar to `thread` and `open`, it receives the identifier of the
mail to forward, on the last generated `list` or `thread` output.  Users can
specify subject, recipients, Cc list, etc via command line.  Then `forward`
sub-command formats the basic mail, and let the user additionally make more
edits interactively and finally send it, in a way pretty similar to that of
`reply`.  Again, `git send-email` setup is required.

Exporting Mails
===============

Users can export specific mails on the last `hkml list`-generated list to a
`mbox` file, using `export` sub-command.  It receives the path to the file that
will save the exported mail.  The file name should have `.mbox` suffix.  Users
can specify which mails from the list need to be exported via `--range` option
of the sub-command.

For example, below exports the first to fourth mails from above example mails
list to `foo.mbox` file, and then lists the exported mails in it again.

```
$ hkml export --range 0 4 foo.mbox
$ hkml list foo.mbox --since 2024-02-15 --until 2024-02-17
[...]
[0] [PATCH 0/5] Docs/mm/damon: misc readability improvements (SeongJae Park, 24/02/16 16:58)
[1]   [PATCH 1/5] Docs/mm/damon/maintainer-profile: fix reference links for mm-[un]stable tree
      (SeongJae Park, 24/02/16 16:58)
[2]   [PATCH 2/5] Docs/mm/damon: move the list of DAMOS actions to design doc (SeongJae Park,
      24/02/16 16:58)
[3]   [PATCH 3/5] Docs/mm/damon: move DAMON operation sets list from the usage to the design
      document (SeongJae Park, 24/02/16 16:58)
```

Users could also import the mbox file on their other mbox-supporting mail
client.

Patches Management
==================

For mails of patches, `hkml patch` command is supported.  Using the command,
users can check the patches, apply the patches on top of their local source
tree, and export the patches as normal files.  For the three purpose, it
provides sub-commands, `check`, `apply`, and `export`.

The three subcommands receives the identifier of the patch mail.  If it is
given with an index of a mail on last-generated list or thread, and if the
index is not for the real patch mail but a cover letter mail of the patch
series, the command fetches all patches of the thread and applies the action to
those.

In the cover letter mail provided use case, the command will add the message in
cover letter into the first patch, like Andrew Morton usually
[does](https://git.kernel.org/sj/c/78f2f60377ee4).  Users can turn off the
behavior using `--dont_add_cv` option.

The command automatically adds 'Link:' and the user's 'Signed-off-by:' tags to
the patches.  It also checks if others provided 'Tested-by:', 'Reviewed-by:',
or 'Acked-by:' tags via reply, reports the finding to the user, and add the
tags to the patches before applying the actions if the user asks to do.

Checking Patches
----------------

`hkml patch check` receives the identifier of the mail of the patches and a
patch checker program to use for checking the patches.  The checker program
should be executable and receive the patch file as a positional argument.  If
there is `./scripts/checkpatch.pl`, it is used as the checker program by
default.  Then, this command fetches the patches and run the checker program.

For example, below runs Linux' `checkpatch.pl` for a patch series.

```
$ hkml list damon
[...]
[28] [RFC PATCH v2 0/4] mm/damon: add a DAMOS filter type for page granularity access recheck
     (SeongJae Park, 2024/03/11 13:45)
[29]   [RFC PATCH v2 1/4] mm/damon/paddr: implement damon_folio_young() (SeongJae Park,
       2024/03/11 13:45)
[...]
$
$
$ hkml patch check 28 ../linux/scripts/checkpatch.pl
[RFC PATCH v2 1/4] mm/damon/paddr: implement damon_folio_young()
total: 0 errors, 0 warnings, 65 lines checked

/tmp/hkml_patch_7fkfxxat has no obvious style problems and is ready for submission.
[RFC PATCH v2 2/4] mm/damon/paddr: implement damon_folio_mkold()
total: 0 errors, 0 warnings, 57 lines checked

/tmp/hkml_patch_s9i6r8xh has no obvious style problems and is ready for submission.
[RFC PATCH v2 3/4] mm/damon: add DAMOS filter type YOUNG
total: 0 errors, 0 warnings, 21 lines checked

/tmp/hkml_patch_8223u8v9 has no obvious style problems and is ready for submission.
[RFC PATCH v2 4/4] mm/damon/paddr: support DAMOS filter type YOUNG
total: 0 errors, 0 warnings, 11 lines checked

/tmp/hkml_patch_4anxkvj3 has no obvious style problems and is ready for submission.
```

Applying Patches
----------------

`hkml patch apply` receives the identifier of the patch mail and apply the
patches to local source tree.  The command assumes current working directory is
the local source tree.  If not, the user can set the path to the tree via
`--repo` option.

For example, below applies the patch series that `hkml patch check` example
was used for.

```
$ hkml patch apply 28 --repo ../linux
[...]
Applying: mm/damon/paddr: implement damon_folio_young()
Applying: mm/damon/paddr: implement damon_folio_mkold()
Applying: mm/damon: add DAMOS filter type YOUNG
Applying: mm/damon/paddr: support DAMOS filter type YOUNG
$
$
$ git -C ../linux log -4 --oneline
af5c978e1153 (HEAD) mm/damon/paddr: support DAMOS filter type YOUNG
d8e76d9430ed mm/damon: add DAMOS filter type YOUNG
f76e7c473942 mm/damon/paddr: implement damon_folio_mkold()
29829d099dd9 mm/damon/paddr: implement damon_folio_young()
```

Exporting Patches
-----------------

`hkml patch export` receives the identifier of the patch mail, and export the
patches as files.  The output explains what mail is exported to what file.  For
example:

```
$ hkml list damon
[...]
[11] [PATCH 0/9] Docs/damon: minor fixups and improvements (SeongJae Park, 2024/07/01 12:26)
[12]   [PATCH 1/9] Docs/mm/damon/design: fix two typos (SeongJae Park, 2024/07/01 12:26)
[...]
$ hkml patch export 11
use patch.msgid.link domain for patch origin? [Y/n]
Given mail seems the cover letter of the patchset.
Adding the cover letter on the first patch.
patch file for mail '[PATCH 1/9] Docs/mm/damon/design: fix two typos' is saved at '/tmp/hkml_patch_-patch-1-9--docs-mm-damon-design--fix-two-typosudjxu63b'
patch file for mail '[PATCH 2/9] Docs/mm/damon/design: clarify regions merging operation' is saved at '/tmp/hkml_patch_-patch-2-9--docs-mm-damon-design--clarify-regions-merging-om5v8v5tv'
[...]
```

Cover Letter Purpose Bogus Commit
---------------------------------

Managing multiple or long changes that should be split into multiple patchsets
is not simple.  One way to do that is having a special bogus commit for each
patchset.  Refer to [below section](#cover-letter-management-git-workflow) for
detailed explanation of two such workflows that `hkml` supports.  `hkml patch
commit_cv` helps making such bogus commit.

By default, it creates the bogus commit [as a baseline
commit](#cover-letter-as-a-baseline-commit).  It receives a message describing
the bogus commit,  Then, it makes a file having `hkml_cv_bogus_` prefix under
`hkml_cv_bogus` directory and commit it on the repo.  The received message is
used as the commit message of the commit.

It also supports creating te bogus commit
[as a merge commit](#cover-letter-as-a-merge-commit).  For this use case, users
should pass the baseline commit of the change via `--as_merge` option.

[`hkml patch format`](#formatting-patches) expects bogus commit usage, and
automatically make cover letter with the bogus commit when user asks.

Formatting Patches
------------------

`hkml patch format` receives git commits range, and formats those into patch
files that can be submitted using `git send-email`.  It uses `git format-patch`
internally, and do below additional works.

### Subjects Based Commits Identification

The commits range given to `hkml patch format` should be somewhat normal `git`
commands support.  `hkml` further allows specifying commits on the argument
with commit subjects in `subject(<subject>)` format.  For example, below
command will convert three commits before a commit of subject "hkml_patch:
implement feature X" to patch files.

    hkml patch format "subject(hkml_patch: implement feature X)~4..subject(hkml_patch: implement feature X)~1"

### Cover Letter Edit

If it is for multiple commits, it creates a cover letter.  Assuming a cover
letter [bogus commit](#cover-letter-purpose-bogus-commit) is used, `hkml patch
format` helps filling the cover letter with the fake commit's message if the
user wants.  In the case, the cover letter draft commit's commit message should
starts with the cover letter subject, then one blank line, and body of the
cover letter.  The subject of the commit and last signed-off-by like tags
paragraph are ignored.  For example, it would look like below:

    $ git log cv_commit --pretty=%B
    ==== Marking start of the work (hkml patch format will ignore this) ====

    hkml_patch_format: support new feature X

    long description of the patch series.
    long description of the patch series line 2.

    Signed-off-by: SeongJae Park <sj@kernel.org>
    $

Or, users can provide a file containing the cover letter content via `--cv`
option.  The file should have the cover letter subject at the first line.
Then, a blank line and the body of the message should follow.

### Recipients Fillup

If it is called on linux tree and `get_maintainer.pl` file exists, it runs the
script to find appropriate recipients for each generated patch file, and add
them as cc to the patch file.  For the cover letter, all the recipients for any
of the patch files are added as cc.  The user can set recipients for all
patches via `--to` and `--cc` options, like `git format-patch`.  If a source
file path is passed to the options, `hkml` finds appropriate recipients for the
source file using `get_maintainer.pl` and add them as cc.

If user didn't set 'To:' recipients, `hkml` shows automatically found and/or
manually user-added cc recipients and let the user picks one of them to be set
as 'To:' recipient (maybe the maintainer of the tree that the patch aim to
landed on) for all patches.

### Checker Run

In the linux tree use case, it also runs `checkpatch.pl` to resulting patch
files, and show found problems if the user wants.

### Subjects Review

If the user wants, `hkml` will list subjects of the generated patch files to
review.  Users can avoid remaining works if they find any problem in subjects.

### Recipients Review

After doing the above works, if the user wants, `hkml` lists recipients of the
generated patch files in a format that easy to review for multiple patches.

### Sending Patches

If the user wants, `hkml` can further send the patch files using `git
send-email` command.

### Example Output

Below is an example usage and output of `hkml patch format`.

```
$ hkml patch format "subject(Docs/admin-guide/mm/damon/usage: add intervals_goal directory on the hierarchy)~8..subject(Docs/admin-guide/mm/damon/usage: add intervals_goal directory on the hierarchy)"
made below patch files
./0000-cover-letter.patch
./0001-mm-damon-add-data-structure-for-monitoring-intervals.patch
./0002-mm-damon-core-implement-intervals-auto-tuning.patch
./0003-mm-damon-sysfs-implement-intervals-tuning-goal-direc.patch
./0004-mm-damon-sysfs-commit-intervals-tuning-goal.patch
./0005-mm-damon-sysfs-implement-a-command-to-update-auto-tu.patch
./0006-Docs-mm-damon-design-document-for-intervals-auto-tun.patch
./0007-Docs-ABI-damon-document-intervals-auto-tuning-ABI.patch
./0008-Docs-admin-guide-mm-damon-usage-add-intervals_goal-d.patch

get_maintainer.pl found.  add recipients using it.

You did not set --to, and we will set below as Cc:
0. Andrew Morton <akpm@linux-foundation.org>
1. Jonathan Corbet <corbet@lwn.net>
2. SeongJae Park <sj@kernel.org>
3. damon@lists.linux.dev
4. linux-doc@vger.kernel.org
5. linux-kernel@vger.kernel.org
6. linux-mm@kvack.org
Shall I set one of above as To: for all mails? [N/index] 0

May I add the base commit to the coverletter? [Y/n]
I will do below to the coverletter (./0000-cover-letter.patch)
- replace "*** SUBJECT HERE ***" with

    mm/damon: auto-tune aggregation interval

- replace "*** BLURB HERE ***" with

    DAMON requires time-consuming and repetitive aggregation interval
    tuning.  Introduce a feature for automating it using a feedback loop
    [...]
    will conduct more evaluations with more massive and realistic workloads
    and share the results by the time that we drop the RFC tag.

looks good? [Y/n]

checkpatch.pl found.  shall I run it?
(hint: you can do this manually via 'hkml patch check')
[Y/n] y
Below 0 patches may have problems
Looks good? [Y/n] y

would you review subjects of formatted patches?
[Y/n] y
below are the subjects
[PATCH 0/8] mm/damon: auto-tune aggregation interval
[PATCH 1/8] mm/damon: add data structure for monitoring intervals auto-tuning
[PATCH 2/8] mm/damon/core: implement intervals auto-tuning
[PATCH 3/8] mm/damon/sysfs: implement intervals tuning goal directory
[PATCH 4/8] mm/damon/sysfs: commit intervals tuning goal
[PATCH 5/8] mm/damon/sysfs: implement a command to update auto-tuned monitoring intervals
[PATCH 6/8] Docs/mm/damon/design: document for intervals auto-tuning
[PATCH 7/8] Docs/ABI/damon: document intervals auto-tuning ABI
[PATCH 8/8] Docs/admin-guide/mm/damon/usage: add intervals_goal directory on the hierarchy

Looks good? [Y/n] y

would you review recipients of formatted patches?
(hint: you can do this manually via 'hkml patch recipients')
[Y/n] y
Common recipients:
  To: Andrew Morton <akpm@linux-foundation.org>
  Cc: SeongJae Park <sj@kernel.org>
  Cc: damon@lists.linux.dev
  Cc: linux-kernel@vger.kernel.org
  Cc: linux-mm@kvack.org
Additional recipients for "[PATCH 0/8] mm/damon: auto-tune aggregation interval"
  Cc: Jonathan Corbet <corbet@lwn.net>
  Cc: linux-doc@vger.kernel.org
Additional recipients for "[PATCH 6/8] Docs/mm/damon/design: document for intervals auto-tuning"
  Cc: Jonathan Corbet <corbet@lwn.net>
  Cc: linux-doc@vger.kernel.org
Additional recipients for "[PATCH 8/8] Docs/admin-guide/mm/damon/usage: add intervals_goal directory on the hierarchy"
  Cc: Jonathan Corbet <corbet@lwn.net>
  Cc: linux-doc@vger.kernel.org
Looks good? [Y/n] y

May I send the patches?  If you say yes, I will do below

    git send-email \
            ./0000-cover-letter.patch \
            ./0001-mm-damon-add-data-structure-for-monitoring-intervals.patch \
            ./0002-mm-damon-core-implement-intervals-auto-tuning.patch \
            ./0003-mm-damon-sysfs-implement-intervals-tuning-goal-direc.patch \
            ./0004-mm-damon-sysfs-commit-intervals-tuning-goal.patch \
            ./0005-mm-damon-sysfs-implement-a-command-to-update-auto-tu.patch \
            ./0006-Docs-mm-damon-design-document-for-intervals-auto-tun.patch \
            ./0007-Docs-ABI-damon-document-intervals-auto-tuning-ABI.patch \
            ./0008-Docs-admin-guide-mm-damon-usage-add-intervals_goal-d.patch \

You can manually review and modify the patch files before answering the next question.
Do it? [y/N] n
```

Cover Letter Management Git Workflows
-------------------------------------

There are multiple ways to manage cover letters on git repo as a commit.  hkml
supports some of such workflows including below.

### Cover Letter as a Baseline Commit

In this workflow, developers create an empty or meaningless change commit, on
top of the real baseline of their changes.  Then, they manage the cover letter
content as the commit message of the baseline commit.  The subject of the
commit usually has special marker prefix or suffix to be easy to be found from
commit log.  For example, below commit log history can be imagined.

    $ git log --pretty="%s"
    feature A development change 3
    feature A development change 2
    feature A development change 1
    ==== feature A development ====
    [...]

`apply`, `format`, and `commit_cv` sub-commands of `hkml patch` support this
worklfow.

### Cover Letter as a Merge Commit

In this workflow, developers create a non-fast-forward fake merge commit that
points the last commit of their work and the baseline as its parent.  Then,
they put the cover letter content as the commit message of the merge commit.
For example, below commit log history can be imagined.

    $ git log --pretty="%s" --graph
    *   Merge "feature Y development"
    |\
    | * feature Y developemnt change 2
    | * feature Y development change 1
    |/
    * Merge "feature X development"
    [...]

`apply`, `format`, and `commit_cv` sub-commands of `hkml patch` support this
worklfow.
