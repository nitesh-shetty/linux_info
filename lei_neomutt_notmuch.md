This document describes how to set up lei/public-inbox to set up along with
notmuch mail filtering and neomutt as mail reader.

1. lei:
lei helps with fetching mail from lore.kernel.org without subscribing to any
mailing list. Lei also has query features which helps in filtering the mails.
For example to fetch mails from linux-block,linux-fsdevel mailing
list from last one day, below command should be useful.

$ lei q -I https://lore.kernel.org/all -o path_to_store_the_mails --threads \
    --dedupe=mid \
    '(tc:linux-block OR tc:linux-fsdevel) AND rt:1.days.ago..'

Here t and c indicate to and cc mail recipient respectively.
Above query also edited later using lei edit-search.

Once the query is finalized, next time to fetch the mails, simply use
$ lei up path_to_store_the_mails

2. notmuch:
Above lei setup usually fetches together and 
notmuch helps in filtering the mails
Typically a notmuch has a config file in ~/.notmuch-config
This file contails the path to fetch and store the filtered mails

With notmuch we can filter mails and attach tags to mails.
And this tag will be used by neomutt to create different virtual mailboxes.

To filter mails to/cc to linux-block and attach tag linux-block, we use
$ notmuch tag +linux-block -- to:linux-block@vger.kernel.org

For more tagging batch queries, an input file can be passed.
Lets say notmuch-tag-rules,
$ cat notmuch-tag-rules
+linux-block -- to:linux-block@vger.kernel.org
+linux-fsdevel -- to:linux-fsdevel@vger.kernel.org

This file looks similar to command line query, without 'notmuch tag'
To run this batch tagging,
$ nomuch tag --batch --input=notmuch-tag-rules

3. neomutt:
neomuttrc is config file, which can be passed to neomutt as argument.
3.a virtual mailbox:
virtual mailboxes help to arrange the mails.
lets say we want to put mails from linux-block under linux-block mailing list.
    virtual-mailboxes "linux-block" "notmuch://?query=tag:linux-block"
    virtual-mailboxes "linux-fsdevel" "notmuch://?query=tag:linux-fsdevel"

3.b sync/fetch mails
we make use of bash script to chek synb the mails.
This script is combination fetching mails from lei and filtering mail
using notmuch.
let's say we name the script fetch.sh
$cat fetch.sh
lei up path_to_store_the_mails
notmuch new
notmuch tag --batch --input=notmuch-tag-rules
notmuch tag --remove-all +deleted tag:deleted

we assign F to fetch mail using,
macro index F "<shell-escape>path_fetchmail_script 2>&1<enter>" "sync email and notmuch"
