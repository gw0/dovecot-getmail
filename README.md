docker-dovecot-getmail
======================

***docker-dovecot-getmail*** is a [*Docker*](http://www.docker.com/) image based on *Debian 8* implementing a private email gateway with [*dovecot*](http://en.wikipedia.org/wiki/Dovecot_(software)) and [*getmail*](http://en.wikipedia.org/wiki/Getmail) for gathering emails from multiple accounts on a private server (IMAP), but using a public email infrastructure for sending (SMTP).

It is a *Docker* container realizing a similar architecture to:

- <http://joel.porquet.org/wiki/hacking/getmail_dovecot/>

```
+-----------+              +-----------+               +--------------+
| ISP       |              | DOCKER    |               | LAPTOP       |
|           |              |           |           +-->|--------------|
| +-------+ | push/delete  | +-------+ | push/sync |   |  MAIL CLIENT +---+
| | IMAPS +----------------->| IMAPS +<------------+   +--------------+   |
| +-------+ |              | +-------+ |           |   +--------------+   |
| +-------+ |              |           |           |   | ANDROID      |   |
| | SMTP  |<-------+       |           |           +-->|--------------|   |
| +-------+ |      |       |           |               |  MAIL CLIENT +---+
+-----------+      |       +-----------+               +--------------+   |
                   +------------------------------------------------------+
```

Open source project:

- <i class="fa fa-fw fa-home"></i> home: <http://gw.tnode.com/docker/dovecot-getmail/>
- <i class="fa fa-fw fa-github-square"></i> github: <http://github.com/gw0/docker-dovecot-getmail/>
- <i class="fa fa-fw fa-laptop"></i> technology: *debian*, *dovecot*, *getmail*
- <i class="fa fa-fw fa-database"></i> docker hub: <https://hub.docker.com/r/gw000/dovecot-getmail/>


Usage
=====

Requirements:

- `/home`: mounted users directories (`Maildir` in fs layout, `sieve`, `.getmail`)
- `/etc/cron.d`: mounted crontabs for executing all getmail accounts
- `/etc/ssl/private`: mounted SSL/TLS certificates (`dovecot.crt`, `dovecot.key`)

Prepare your getmailrc account configurations per user (`/srv/mail/home/user/.getmail/getmailrc-user@email.invalid`):

```
# ~/.getmail/getmailrc-*: getmailrc email configuration

[retriever]
type = SimpleIMAPSSLRetriever
server = imap.email.invalid
username = user@email.invalid
port = 993
password = password
mailboxes = ("INBOX", "Sent", "Spam")

[destination]
type = MDA_external
path = /usr/lib/dovecot/deliver
arguments = ("-e",)

[options]
read_all = false
delete_after = 30
delivered_to = false
received = true
verbose = 1
```

If you are using Sieve filters and want a `Refilter` mailbox to trigger their refiltering, create a refilter configuration per user (`/srv/mail/home/user/.getmail/getmailrc-refilter`):

```
# ~/.getmail/getmailrc-*: getmailrc refilter configuration

[retriever]
type = SimpleIMAPRetriever
server = localhost
port = 143
username = user
password = password
mailboxes = ("Refilter",)

[destination]
type = MDA_external
path = /usr/lib/dovecot/deliver
arguments = ("-e",)

[options]
read_all = false
delete = true
delivered_to = false
received = false
verbose = 1
```

Prepare crontab file (`/srv/mail/cron.d/getmail`) for periodically checking for new mail for each user and account:

```
# /etc/cron.d/getmail: system-wide crontab for getmail
SHELL=/bin/sh

# m h dom mon dow user  command
*/20 *  *   *   * user  ACC="user-refilter" && (date; flock -n ~/.getmail/lock-$ACC getmail --rcfile="getmailrc-$ACC" --idle Refilter) >>"/var/log/getmail/$ACC.log" 2>&1
*/20 *  *   *   * user  ACC="user@email.invalid" && (date; flock -n ~/.getmail/lock-$ACC getmail --rcfile="getmailrc-$ACC" --idle INBOX) >>"/var/log/getmail/$ACC.log" 2>&1
```

Do not forget to place your SSL certificates as `/srv/mail/ssl/dovecot.crt` and `/srv/mail/ssl/dovecot.key`. SSL is required!

And finally start it with *docker*:

```bash
$ docker run -d -v /srv/mail/home:/home -v /srv/mail/cron.d:/etc/cron.d -v /srv/mail/ssl:/etc/ssl/private:ro -p 143 -p 993 -p 4190 --name mail gw000/dovecot-getmail
```

Or use *docker-compose* (check out `docker-compose.example.yml`).

Users are created automatically with default password (`replaceMeNow`) on first start. To reset user passwords (of a running container):

```bash
$ docker exec -it mail passwd user
```


Feedback
========

If you encounter any bugs or have feature requests, please file them in the [issue tracker](http://github.com/gw0/docker-dovecot-getmail/issues/) or even develop it yourself and submit a pull request over [GitHub](http://github.com/gw0/docker-dovecot-getmail/).


License
=======

Copyright &copy; 2016 *gw0* [<http://gw.tnode.com/>] &lt;<gw.2016@tnode.com>&gt;

This library is licensed under the [GNU Affero General Public License 3.0+](LICENSE_AGPL-3.0.txt) (AGPL-3.0+). Note that it is mandatory to make all modifications and complete source code of this library publicly available to any user.
