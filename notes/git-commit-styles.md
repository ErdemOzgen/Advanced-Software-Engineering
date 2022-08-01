## Commit Message Guidelines
```
Capitalized, short (50 chars or less) summary

More detailed explanatory text, if necessary.  Wrap it to about 72
characters or so.  In some contexts, the first line is treated as the
subject of an email and the rest of the text as the body.  The blank
line separating the summary from the body is critical (unless you omit
the body entirely); tools like rebase will confuse you if you run the
two together.

Write your commit message in the imperative: "Fix bug" and not "Fixed bug"
or "Fixes bug."  This convention matches up with commit messages generated
by commands like git merge and git revert.

Further paragraphs come after blank lines.

- Bullet points are okay, too

- Typically a hyphen or asterisk is used for the bullet, followed by a
  single space, with blank lines in between, but conventions vary here

- Use a hanging indent
```
### Example for a commit message
```
Add CPU arch filter scheduler support
In a mixed environment of…
```
### A properly formed git commit subject line should always be able to complete the following sentence
If applied, this commit will *\<your subject line here\>*
### Rules for a great git commit message style
* Separate subject from body with a blank line
* Do not end the subject line with a period
* Capitalize the subject line and each paragraph
* Use the imperative mood in the subject line
* Wrap lines at 72 characters
* Use the body to explain what and why you have done something. In most cases, you can leave out details about how a change has been made.
### Information in commit messages
* Describe why a change is being made.
* How does it address the issue?
* What effects does the patch have?
* Do not assume the reviewer understands what the original problem was.
* Do not assume the code is self-evident/self-documenting.
* Read the commit message to see if it hints at improved code structure.
* The first commit line is the most important.
* Describe any limitations of the current code.
* Do not include patch set-specific comments.
Details for each point and good commit message examples can be found on https://wiki.openstack.org/wiki/GitCommitMessages#Information_in_commit_messages
### References in commit messages
If the commit refers to an issue, add this information to the commit message header or body. e.g. the GitHub web platform automatically converts issue ids (e.g. #123) to links referring to the related issue. For issues tracker like Jira there are plugins which also converts Jira tickets, e.g. 
* [Jirafy](https://github.com/square/jirafy)
* [Github Autolink References](https://github.blog/2019-10-14-introducing-autolink-references/)
* [Issue Tracking in Jira](https://deviniti.com/blog/enterprise-software/issue-tracking-jira/)
    
In header:
```
[#123] Refer to GitHub issue…
```
```
CAT-123 Refer to Jira ticket with project identifier CAT…
```
In body:
```
…
Fixes #123, #124
```

## Things to avoid when creating commits
With that in mind, there are some commonly encountered examples of bad things to avoid

Mixing whitespace changes with functional code changes.
The whitespace changes will obscure the important functional changes, making it harder for a reviewer to correctly determine whether the change is correct. Solution: Create 2 commits, one with the whitespace changes, one with the functional changes. Typically the whitespace change would be done first, but that need not be a hard rule.

Mixing two unrelated functional changes.
Again the reviewer will find it harder to identify flaws if two unrelated changes are mixed together. If it becomes necessary to later revert a broken commit, the two unrelated changes will need to be untangled, with further risk of bug creation.

Sending large new features in a single giant commit.
It may well be the case that the code for a new feature is only useful when all of it is present. This does not, however, imply that the entire feature should be provided in a single commit. New features often entail refactoring existing code. It is highly desirable that any refactoring is done in commits which are separate from those implementing the new feature. This helps reviewers and test suites validate that the refactoring has no unintentional functional changes. Even the newly written code can often be split up into multiple pieces that can be independently reviewed. For example, changes which add new internal APIs/classes, can be in self-contained commits. Again this leads to easier code review. It also allows other developers to cherry-pick small parts of the work, if the entire new feature is not immediately ready for merge. Addition of new public HTTP APIs or RPC interfaces should be done in commits separate from the actual internal implementation. This will encourage the author & reviewers to think about the generic API/RPC design, and not simply pick a design that is easier for their currently chosen internal implementation. If patch impacts a public HTTP, use the APIImpact flag (see Including external references).

The basic rule to follow is

>>If a code change can be split into a sequence of patches/commits, then it >>should be split. Less is not more. More is more.

## Examples of good practice
#### Example 1
```
 commit 3114a97ba188895daff4a3d337b2c73855d4632d
  Author: [removed]
  Date:   Mon Jun 11 17:16:10 2012 +0100

    Update default policies for KVM guest PIT & RTC timers

    The default policies for the KVM guest PIT and RTC timers
    are not very good at maintaining reliable time in guest
    operating systems. In particular Windows 7 guests will
    often crash with the default KVM timer policies, and old
    Linux guests will have very bad time drift

    Set the PIT such that missed ticks are injected at the
    normal rate, ie they are delayed

    Set the RTC such that missed ticks are injected at a
    higher rate to "catch up"

    This corresponds to the following libvirt XML

      <clock offset='utc'>
        <timer name='pit' tickpolicy='delay'/>
        <timer name='rtc' tickpolicy='catchup'/>
      </clock>

    And the following KVM options

      -no-kvm-pit-reinjection
      -rtc base=utc,driftfix=slew

    This should provide a default configuration that works
    acceptably for most OS types. In the future this will
    likely need to be made configurable per-guest OS type.

    Closes-Bug: #1011848

    Change-Id: Iafb0e2192b5f3c05b6395ffdfa14f86a98ce3d1f
```
Some things to note about this example commit message

* It describes what the original problem is (bad KVM defaults)
* It describes the functional change being made (the new PIT/RTC policies)
* It describes what the result of the change is (new the XML/QEMU args)
* It describes scope for future improvement (the possible per-OS type config)
* It uses the Closes-Bug notation

#### Example 2
```
 commit 31336b35b4604f70150d0073d77dbf63b9bf7598
  Author: [removed]
  Date:   Wed Jun 6 22:45:25 2012 -0400

    Add CPU arch filter scheduler support

    In a mixed environment of running different CPU architecutres,
    one would not want to run an ARM instance on a X86_64 host and
    vice versa.

    This scheduler filter option will prevent instances running
    on a host that it is not intended for.

    The libvirt driver queries the guest capabilities of the
    host and stores the guest arches in the permitted_instances_types
    list in the cpu_info dict of the host.

    The Xen equivalent will be done later in another commit.

    The arch filter will compare the instance arch against
    the permitted_instances_types of a host
    and filter out invalid hosts.

    Also adds ARM as a valid arch to the filter.

    The ArchFilter is not turned on by default.

    Change-Id: I17bd103f00c25d6006a421252c9c8dcfd2d2c49b
```
Some things to note about this example commit message

* It describes what the problem scenario is (mixed arch deployments)
* It describes the intent of the fix (make the schedular filter on arch)
* It describes the rough architecture of the fix (how libvirt returns arch)
* It notes the limitations of the fix (work needed on Xen)

#### Example 3
```
 commit 71f0e301132a7576f238fc1e51ae0ebc399dce43
 Author: [removed]
 Date:   Wed Jul 21 08:47:13 2021 -0400

    Add parallel option to the collect tool

    The current implementation of collect cycles through
    the specified host list, one after the other.

    This update adds a parallel (-p|--parallel) option to
    collect with the goal to decrease the time it takes to
    collect logs/data from all hosts in larger systems.

    This update does not change any of the current collect
    default options. The collect tool will take advantage
    of this new feature if the -p or --parallel option is
    specified on the command line when starting collect.

    Unless specified, all of the following test cases
    were executed for both serial and parallel collects.

    Test Plan:

    PASS: Verify collect output and logging

    Failure Cases: Failure Handling = FH

    PASS: Verify collect FH for an offline host
    PASS: Verify collect FH for host that recently rebooted
    PASS: Verify collect FH for host that reboots during collect
    PASS: Verify collect FH for host mgmnt network drop during collect
    PASS: Verify collect FH of various bad command line options
    PASS: Verify parallel collect overall timeout failure handling

    Regression:

    PASS: Verify dated collect
    PASS: Verify handling of unknown host
    PASS: Verify ^C|TERM|KILL running collect removes all child processes
    PASS: Verify Single host collect (any host)
    PASS: Verify Listed hosts collect (many different groupings)

    Soak:

    PASS: Verify repeated collects (50+) until after local fs is full

    Change-Id: I91814d14341cdc438a6d5af999b6c12d39c7d97c
```
Some things to note about this example commit message

* It describes what the original limitation is (collect cycles through sequentially)
* It describes the functional change being made (Add parallel option)
* It describes the intent of the change (decrease the time)
* It describes the tests executed (Test Plan)

### Also Take a Look for These Links

* http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html
* https://wiki.openstack.org/wiki/GitCommitMessages
* http://chris.beams.io/posts/git-commit/
* https://www.freecodecamp.org/news/writing-good-commit-messages-a-practical-guide/
* https://cbea.ms/git-commit/
* https://www.git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project#_commit_guidelines
* https://github.com/torvalds/subsurface-for-dirk/blob/master/README.md#contributing
