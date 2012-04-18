Design principles
=================

Highly loosly coupled system, built on git following the git-ish unix-ish
philosophy of small tools, doing one thing.

Loose coupling allows us to choose the best languages for the job for the
different layers. For instance, IIF import/export is likely best done via 

Data storage in easy to read, easy to manipulate form:

Written directly to bare Git repository, that other tools can manipulate
using git and the filesystem.

Data storage using hashed filesystem directories and JSON data files.

Designed to make common merge conditions easy to integrate.

Scalability
===========

While the intitial intent is for what might be better called micro-business,
where there's an owner and perhaps a few contractors or part time employees,
being able to scale up to small-business, in the 10-100 employee range would
be nice, and mid-size business in the <1000 employee range on the far horizon.

We need to better understand the transaction volume at each of these levels.
We started by discussing splitting date based transaction indexes by month, but
it strikes me that it is worth considering splitting out by month and then
day...  eg accounts/ACCT/YYYY-MM/DD.json

Seeing as even small busineses can see 1000s of transactions a month. 
Mid-size 10s of thousands?

Understanding access patterns will improve understanding of how data should
be layed out, but that said, I'm currently quite fond of the
inverted_example, in which actual data is hashed out and stored separately,
one per transaction, regardless of how many entries the transaction has, and
the accounts are just indexes point at the transaction ids. You can change
the order that transactions are displayed for a particular day by simply
reordering them.

Or should the transactions be stored, one per file, in directories for the
particular day or month, and just duplicated to other accounts?  Duplicating
may not actually be dangerous, as git itself will maintain the same hash,
and thus know their the same file?  Or is cross linking like this
impossible?  Does the hash contain the path as well?

Ah, so even if git could support cross linking like that, we won't do it as
it violates the principle that we should be able to edit accounts with just
git, a text editor and a program to generate the indices.

There would be other indexes for various categories.  Category indexes would
be broken out... like accounts? Or by other time periods?

Will we need to be able to cross indices, eg, pull out by category and
month?  Probably, but will we need to do that any time we won't want to be
doing a full scan in one or the other index anyway?  Doubtful.

What kind of searching is desirable? Necessary?

Todo
====

Gather some basic user stories for different use cases, particularly those that
involve concurrency and thus imply merging. 

But more general stories too, to start formalizing capturing what the basic
needs actually are in a more realistic form then a feature list.

---

Infrastructure
==============

* What can vary between different portions of the same transaction?
  * Date?
  * Memo?
* Make accounts the indexes to hashed transactions? Transaction objects
  would then be variable in size, but complete.
* What is the minimal chart of accounts?
* What are the minimal required reports?
  * Aggregated
  * Browsable
* What does the most basic data entry look like?
* What other metadata should be tied to transaction?
  * Check #s for checks, obviously
  * This gets into the idea of /kinds/ of transactions.  We need the minimal
    list of the kinds of transactions required.  It may be able to make these
    sorts of things data driven.  In so far as that's possible, we'll want
    to do that.
* How will we represent the closing of a month? Keep closed months separately?
* Is a month the smallest unit that can be closed?
* Is closing a year different then closing 12 months individually?

Data integrity tools
====================

* Every storage design will have data integrity concerns, at least, around
  indexes.  From the start there should be tools to verify this and declare
  one or the other to be canonical.

---

User interface requirements
===========================

* What data entry mechnisms are required?

* Should different roles have different merge strategies? Should some roles
  simply have no merge strategy-- eg, if they can't be cleanly merged, they're
  pushed to a branch for someone more savvy to do the merge.
  Or perhaps even roles that never have their changes applied automatically--
  their changes are essentially staged as pull requests that an administrator
  accepts into their master copy of the accounts.  Each would, however, need to
  be a branch of the previous push-- cascading, rather the parallel branches.
  

* It should be possible to "edit" a transaction from a closed month.  Obviously,
  this wouldn't actually edit the transation in that period, but it would allow
  later recategorization and (possibly) the addition of further metadata.  New
  metadata would simply be added to the transaction record.  Recategorization would
  involve creating a new transaction in the current month linked to the previous one
  that moves the sum to the other category.

* Reporting
  * What must interactive displays be able to do?
  * What printable forms are required?
  * Check printing?
  * Any form printing?
  * Invoice printing?
    * Invoice customization? Or formatting to print on to preprinted
      letterhead?

---

Import / Export
===============

* Obvious formats:
  * IIF/QIF
    * These are full of ad-hoc kinds of information...
      We should look at actual exports from businesses of the kind that
      we're targeting to get a sense of what is actually important in
      these files, both, primarily, from QuickBooks online and from
      desktop QuickBooks... a sample of an older version and the current
      version.
  * JSON
  * Is anything else actually required for minimal system?
* Integration with payroll systems or an ability to do it internally is
  critical for larger scale use.  This is mostly an import/export and/or
  reporting problem, depending on the approach taken by the end user.

---

Other thoughts
==============

* It seems desirable to me to have fully offline modes of operation.  This
  implies either a native application or a full offline JavaScript version
  of the application.  I would prefer the latter...  this would allow
  offline operation to be the norm rather then exception, with pushes occuring
  whenever a natural save point occurs when online, or upon going online.  This
  confusion over how web-based entries should be reintegrated-- this would now
  always be done in the same way.
* This does imply that we would have to write a pure JavaScript version of
  at least basic git opperations, although non-fastforward merges wouldn't
  necessarily need to be implemented in this layer.
* Other advantages include ensuring that all users always have a full copy
  of their data.
* Disadvantages include startup time on a new computer, while books are
  downloaded-- however that said, existing, uncompressed formats, books are
  still very small, in git they'll be even smaller.
* Also, this is dependent on browsers ability to handle local storage, which
  is still dubious.  On the other hand, it's likely to firm up in the coming
  year or two.
