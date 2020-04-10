# Sensu Go Aggregate Check Plugin

An aggregate makes it possible to treat the result of multiple disparate check executions executed across multiple disparate systems as a single result (Event). Aggregates are extremely useful in dynamic environments and/or environments that have a reasonable tolerance for failure. Aggregates should be used when a service can be considered healthy as long as a minimum threshold is satisfied (e.g. are at least 5 healthy web servers? are at least 70% of N processes healthy?).

This plugin allows you to query the Sensu Go Backend API for Events matching certain criteria (labels). This plugin generates a set of counters (e.g. events total, events in an OK state, etc) from the Events query and provides several CLI arguments to evaluate the computed aggregate counters (e.g. --warn-percent=75).

## Configuration

Example Sensu Go definition:

```yaml
api_version: core/v2
type: CheckConfig
metadata:
  namespace: default
  name: dummy-app-aggregate
spec:
  runtime_assets:
  - sensu-aggregate-check
  command: sensu-aggregate-check --api-key foobar-foo-bar-foobar -U https://api --check-labels='aggregate=healthz,app=dummy' --warn-percent=75 --crit-percent=50
  subscriptions:
  - backend
  publish: true
  interval: 30
  handlers:
  - slack
  - pagerduty
  - email
```

## Usage Examples

### Patch notes
In our testing with sensu-go-backend-5.18.1-9930, we found using using a username/password sometimes results in a {"Code":4,"Message":"unauthorized to perform action"} response from the API.  We added API key support and this performs better in our environment.

We also needed HTTPS support, so this patch allows -U/--url to specify an HTTPS (or HTTP) Sensu backend URL.  You can also specify -k to ignore self-signed certs.

New options:
 * -K --api-key
 * -k --insecure trust unknown certs (like Curl)
 * -U --url   specify a URL instead of host and port

Using this patch, which allows an untrusted SSL cert, an API KEY, and a URL specifying HTTPS on port 8443
```
./sensu-aggregate-check --api-key 123456789-a7d3-1234-4567-1234567890 -k --url https://qammon1:8443 -l aggregate=test
```

Help:

```
The Sensu Go Event Aggregates Check plugin

Usage:
  sensu-aggregate-check [flags]

Flags:
  -H, --api-host string        Sensu Go Backend API Host (e.g. 'sensu-backend.example.com') (default "127.0.0.1")
  -K, --api-key string         Sensu Go Backend API Key (use instead of username/password
  -P, --api-pass string        Sensu Go Backend API User (default "P@ssw0rd!")
  -p, --api-port string        Sensu Go Backend API Port (e.g. 4242) (default "8080")
  -u, --api-user string        Sensu Go Backend API User (default "admin")
  -l, --check-labels string    Sensu Go Event Check Labels to filter by (e.g. 'aggregate=foo')
  -C, --crit-count int         Critical threshold - count of Events in critical state
  -c, --crit-percent int       Critical threshold - % of Events in critical state
  -e, --entity-labels string   Sensu Go Event Entity Labels to filter by (e.g. 'aggregate=foo,app=bar')
  -h, --help                   help for sensu-aggregate-check
  -k, --insecure               Allow insecure server connections when using SSL
  -n, --namespaces string      Comma-delimited list of Sensu Go Namespaces to query for Events (e.g. 'us-east-1,us-west-2') (default "default")
  -U, --url string             Sensu Go Backend URL (e.g., https://sensu-backend:8443)
  -W, --warn-count int         Warning threshold - count of Events in warning state
  -w, --warn-percent int       Warning threshold - % of Events in warning state
```
