# What CalDAV-server to choose?

NOTE: this was written in February 2023, and may not be completely up-to-date.  At some point I should consider to go through it and check if I can come up with new recommendations.

In November 2024 I started writing a tool for testing caldav server compatibility and checking how the servers handles various corner cases (curerntly it's living in a git branch in the caldav project).  I'm intending to bug at least some of the server maintainers with my findings.

**TLDR-summary**:

To use plann heavily both to manage calendars and do heavy task management, none of the calendar servers out there seems to be good enough.

Xandikos seems to be the best - it works out of the box, the developer is quick to fix things when filing bug reports, everything works well ... except for one tiny thing, recurrances.  Well, I do consider recurrances to be important!

I ended up with DAViCal.  It has been out there for a decade or two and is heavily used by a large community - still, first thing that happened when I tried to run plann towards it was ... 500 internal server error!  I did have to make some workarounds both in my CalDAV installation and in plann.

I keep pushing to get things fixed upstream by the various calendar server developers, hopefully I can come with a more solid recommendation soon enough :-)

## Requirements

To be able to efficiently use `plann` version 1.0 (no, it has not been released yet - work in progress) for planning, you will need a calendar server that supports ...

* Tasks (VTODO component type)
* Searching on text fields
* Searching on categories
* Relatively complex searches
* Recurring events
* Recurring tasks
* Relationships (RELATED-TO property type)

I have on the long term list to also make it easy to add journal entries after a meeting has been held or a task has been done - then the calendar should also support the VJOURNAL component type.

Perhaps at some point in the future, I will also add support for "scheduling" (basically, sending meeting invites and respond to them).

## Different servers tested (self-hosting)

### DAViCal

DAViCal seems to be one of the most standard-compliant caldav servers out there, and it has been around for quite a while.  It supports everything we need, even scheduling.  It has a basic WebUI for administrating access rights, users, etc.  It also supports address books (VCARD).

It's written in PHP, needs a database backend (PostgreSQL) and a webserver frontend (Apache is probably easiest - though I went for php-fpm and nginx).

I installed a version a decade ago, didn't upgrade it ever, and it worked out well enough.  Revisiting it with the newest version of plann in 2023, things didn't work out very well - to get it to play well with plann I needed to write up pull request https://gitlab.com/davical-project/davical/-/merge_requests/101 and https://gitlab.com/davical-project/davical/-/merge_requests/102, and I still need to find a workaround for the last issue at https://gitlab.com/davical-project/davical/-/issues/281

### Radicale

https://radicale.org/

A relatively new server, written in Python.  It's included in the test suite of the python caldav library, so it's well-tested.  It's also very easy to set up.

It does not support the scheduling RFC, but it does support multiple users and some kind of permission settings.

Radicale seems to be less maintained than Xandikos.  Category search was broken with a simple traceback, this has been an open issue for more than two years, I proposed a one-line workaround in https://github.com/Kozea/Radicale/issues/1125 with no feedback for several months.  Someone has submitted a more proper fix in https://github.com/Kozea/Radicale/pull/1280 (... and I've added test code in pull request #1281).  Hence, my recommendation is to patch up Radicale if going for this option - but still, it fails as it's unable to do date-searches for uncompleted tasks.  I gave up on Radicale due to this.

Radicale also supports address books (VCARD).

### Xandikos

https://www.xandikos.org/

Relatively new server, written in Python, stores all data in a local git-repository.  Super-easy to set up and get started with.  Quite standard-compliant.  Compability is ensured, as it's included in the python caldav library test suite, and hence tested almost on every commit.  All features in plann as of 2023-01 has been tested and works flawlessly against Xandikos.  I would definitely recommend Xandikos if/when it will support recurrances - the missing support for recurring events and task is a big show-stopper for me.

Xandikos does not support multiple users and access controls (I would still use it for my family - one calendar for each person, and no personal secrets stored in the calendars).  The lack of multi-user support also means support for scheduling is missing.

Xandikos also supports address books (VCARD).

### SOGo

https://www.sogo.nu/

This is "Groupware" supporting mail and featuring a rich web interface.

SOGo has been around for as long as the python caldav library.

SOGo does not support searches on text fields and other advanced searches (and this is needed for advanced task management in plann).

SOGo did not support journals last time I checked.

SOGo is probably very good if you want a web based email and calendaring tool which supports syncing towards a large amount of devices and software, but it's not good enough when a "thin" client such as plann wants search for some particular events or tasks in a calendar.

### Robur

https://github.com/roburio/caldav

Robur is written in ... eh ... what?

It does not support relationship attributes, text search is not working, it does not support journals and some other problems.

### Baikal

https://sabre.io/baikal/

PHP-based application.

I haven't revisited it for a while, but Baikal does not support recurring tasks.  I think recurring tasks are important, but if you disagree you may probably try Baikal.

Baikal has problems with synchronizing (through sync tokens) breaking if something got deleted from the calendar.

### Nextcloud

https://www.nextcloud.com/

Nextcloud has a lot of components, including calendar.

I believe the calendar is based on Sabre, at list my list of problems is roughly the same as for Baikal.

### Zimbra

"Groupware", supporting mail and featuring a rich web interface.

If you're already using Zimbra, then by all means - you may try, and you may be lucky.  I do use plann against Zimbra myself - but it's not my primary calendar, and it also seems like they're breaking their CalDAV support little by little for every release.

On my list (some of it is old, I haven't bothered to recheck):

* Zimbra does not support journal entries
* Setting the "display name" of a calendar through CalDAV does not work
* It's not possible to do a simple GET towards a calendar component resource
* Zimbra has a very clear distinction between a "task list" and a "calendar", one cannot combine tasks and events on the same calendar.
* Sync tokens not supported
* Category search yields nothing
* Relationships cannot be set
* Quite some other minor issues

### Bedework

"Bedework Enterprise Calendar System" is the title of https://www.bedework.org/ - but the web site there looks like it was written in 1990.  It does point to more enterprise-looking web pages though.  Java-based.

I haven't revisited this one, but when testing it long long time ago it didn't support tasks nor journals nor recurrances, so it is (or was?) a no-go.

## Calendar-as-a-service providers

(This section should probably be improved and expanded)

* Robur is offered as a cloud service - https://calendar.robur.coop/
* Fastmail does not support tasks and some other problems.  They also did not allow me to have a dedicated permanent test account for free, so I haven't done more testing.
* ecloud.global offers NextCloud
* Google had quite limited CalDAV support, and did not support tasks last time I checked
* iCloud does not officially support CalDAV - unofficially they do have some quite broken CalDAV-support, with no support for recurrances nor tasks and many other things are broken.

## Non-free solutions

* Synology does not support recurring tasks

## Things I should consider testing for

The current test suite is sort of a by-product of the Python CalDAV library.  Originally the idea was to just verify that the library would interoperate with different clients.  Eventually it has grown into a test-suite that verifies how compliant the different CalDAV servers are.  I would eventually like to fork it out into a separate product - a CalDAV compliance checker.  The current code does not go far enough:

* It does not check if it's possible to add tasks and events (VTODO and VEVENT) to the same calendar.  I know most servers supports it and that Zimbra does not support it, but should explicitly test this on all the servers.
* The possibility to ask for events and tasks in the same report query.  This is important for plann, as this is the default search mode.  For xandikos, sometimes the search requests come out empty unless one explicitly adds --todo or --event.  I should check if this is the same for other implementations.
* Expansion support - I should write up dedicated tests to catch weather the server supports expansion or not (since the caldav library now does expansion on the client side, the earlier tests does not show this anymore)
* Compatibility matrix - currently I'm editing it by hand all until all the tests are passing - but I should write code that autodetects what kind of flaws the various servers have.
* Compatibility matrix should not be "hidden" in the test folder, it should be made available both to the caldav library and to plann, allowing the tools to do workarounds if needed.

## Conclusion

All I wanted to do was to set up a personal calendar and access it easily through the command line ... and I fell down a rabbit hole and have been fighting with the CalDAV protocol and iCalendar format now for almost ten years.  Initially I hoped not to have to deal with the CalDAV standard itself, I hoped that I could just use other python libraries for interfacing towards the calendar server, but it didn't work out very well - soon enough I ended up with the maintainer hat for the python CalDAV client library.

Let me be completely honest about it - I'm not happy, neither about the CalDAV standard nor the iCalendar standard.  The protocols seems to be not very well thought-through, there are quite some problems with them.  While the iCalendar standard is thriving, things are apparently not working out that well for the CalDAV standard.  It's needed to install extra software on Android to support CalDAV - but yet, as far as I know, that's the only standard we have for two-way server/client calendaring communication.  The alternative to using CalDAV is to sync the clients by downloading a read-only ics feed, copying the whole calendar every time it's synced, or to keep a local copy of all the calendar events and send/receive calendaring data on event-basis over other protocols, like email attachments.  Neither of the solutions are good.

The CalDAV protocol seems to be designed so that the client can be relatively simple while most of the work is done on the server side (expansions, complex searches, etc).  Unfortunately the server support for this is often broken - like expansion, there is quite a list of servers that supports recurrences but doesn't support server-side expansion of the events.  My idea for calendar-cli and plann was always to do as little work as possible at the client side and leave all the complexity for the smart people creating the server software.  Unfortunately ... it doesn't always work out that well.  Even when using the CalDAV protocol, to fully support all servers, the best thing one can do is probably to download the full calendar frequently and do the various logics on the client side.

It may be a minor problem when performing a search query and the server returns too much.  This may be handled on the client side - I'm considering to add code to filter away the extra data.  However, the bigger problem is when a report query returns nothing, or returns only a fraction of the components it should return.  That may cause one to miss appointments and forget about important tasks.  Not good!
