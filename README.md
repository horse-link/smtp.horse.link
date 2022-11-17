# smtp.horse.link
IRedMail v1.6.2 deployed a on DO droplet, connected to the DO managed PG DB

# Details
[Admin interface](https://mail.horse.link/iredadmin)
[Mailbox](https://mail.horse.link/mail)
Login details in BW

## Install instructions
IRedMail instructions: https://docs.iredmail.org/install.iredmail.on.debian.ubuntu.html
Following the wizard, selecting a PG SQL backend, will install a local PG DB on the droplet.

## Migrate the DB
To migrate to the managed DB, use pg_dumpall from local.
`PGPASSWORD="<password from wizard>" pg_dumpall -h localhost --username postgres > dump.sql`

Then SCP the file down to your local
`scp root@167.172.70.188:/root/iRedMail-1.6.2 dump.sql`

The dump file will need to be altered to remove the commands that require superuser access (DO does not provide supuser access)
* Anything referencing the postgres DB or postgres user need to be removed
* Cmds to create and then alter roles in multiple cmds need to be consolidated into the just create statements (superuser access is required for alter role)

Connect to the DB from your local PGAdmin or equivalent. Then run a PSQL to import the dump
`psql -f "dump.sql" "host='db-postgresql-sgp1-16621-do-user-7279278-0.b.db.ondigitalocean.com' port='25060' dbname='defaultdb' user='doadmin' sslmode='prefer' sslcompression='False' " `

Once the DB is ready the app will need to be updated to use it. There are several places this neesd to be done. There is no official doc for this but the closest is this list of paths on the droplet that will need to be checked: https://forum.iredmail.org/post74151.html#p74151

For each path mentioned search for the default PG SQL port 5432 to pinpoint where updates will need to be applied:
`grep -rnw '<path to dir>' -e '5432'`

Once all the changes have been applied, all the apps that comprise iRedMail will need to be restarted. The easiest way to do this is to restart the droplet.

Once the app is up, test the login and make some changes in the admin interface, e.g. adding a domain or a user, then checking the DB for the associated records.
