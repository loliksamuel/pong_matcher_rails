# CF example app: ping-pong matching server

This is an app to match ping-pong players with each other. It's currently an
API only, so you have to use `curl` to interact with it.

It has an [acceptance test suite][acceptance-test] you might like to look at.

**Note**: We highly recommend that you use the latest versions of any software required by this sample application. 

## Running on [Pivotal Web Services][pws]

Log in.

```bash
cf login -a https://api.run.pivotal.io
```

Target your org / space. An empty space is recommended, to avoid naming collisions.

```bash
cf create-space spc1 -o allocation

cf target -o allocation -s spc1
```

Sign up for a cleardb instance.

```bash
cf create-service cleardb spark mysql
```

Push the app. Its manifest assumes you called your ClearDB instance 'mysql'.

```bash
cf push -n mysubdomain4
```

Export the test host
wait and see long logs
see the route
http://mysubdomain4.cfapps.io/all
```bash
export HOST=http://mysubdomain4.cfapps.io
```

Now follow the [interaction instructions](#interaction-instructions).

## Running locally

The following assumes you have a working, recent version of [Ruby][ruby]
installed.

Install and start MySQL:

```bash
brew install mysql
mysql.server start
mysql -u root
```

Create a database user and table in the MySQL REPL you just opened:

```sql
CREATE USER 'railspong'@'localhost' IDENTIFIED BY 'railspong';
CREATE DATABASE pong_matcher_rails_development;
GRANT ALL ON pong_matcher_rails_development.* TO 'railspong'@'localhost';
exit
```

Install the project's dependencies:

```bash
bundle install --binstubs
```

Create and migrate the database:

```bash
bin/rake db:create db:migrate
```

Start the application server:

```bash
bin/rails s
```

Export the test host

```bash
export HOST=http://localhost:3000
```

Now follow the [interaction instructions](#interaction-instructions).

## Interaction instructions

Start by clearing the database from any previous tests.
You should get a 200.

```bash
curl -v -X DELETE $HOST/all
```

Then request a match, providing both a request ID and player ID. You should get
a 204.

```bash
curl -v -H "Content-Type: application/json" -X PUT $HOST/match_requests/firstrequest -d '{"player": "andrew"}'
```

Now pretend to be someone else requesting a match:

```bash
curl -v -H "Content-Type: application/json" -X PUT $HOST/match_requests/secondrequest -d '{"player": "navratilova"}'
```

Let's check on the status of our first match request:

```bash
curl -v -X GET $HOST/match_requests/firstrequest
```

The bottom of the output should show you the match_id. You'll need this in the
next step.

Now pretend that you've got back to your desk and need to enter the result:

```bash
curl -v -H "Content-Type: application/json" -X POST $HOST/results -d '
{
    "match_id":"thematchidyoureceived",
    "winner":"andrew",
    "loser":"navratilova"
}'
```

You should get a 201 Created response.

Future requests with different player IDs should not cause a match with someone
who has already played. The program is not yet useful enough to
allow pairs who've already played to play again.

[acceptance-test]:https://github.com/cloudfoundry-samples/pong_matcher_acceptance
[pws]:https://run.pivotal.io
[ruby]:https://www.ruby-lang.org/
