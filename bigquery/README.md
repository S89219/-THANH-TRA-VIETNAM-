# Google BigQuery + Github Archive

[Google BigQuery](https://developers.google.com/bigquery/) is a web service that lets you do interactive analysis of massive datasets—up to billions of rows.

The Github Activity stream is automatically uploaded to BigQuery sevice to enable interactive analysis.

## Sample Queries

```sql
/* distribution of different events on GitHub */
SELECT type, count(type) as cnt
FROM [github.events]
GROUP BY type
ORDER BY cnt DESC

/* distribution of different events on GitHub for Ruby */
SELECT type, count(type) as cnt
FROM [github.events]
WHERE repository_language="Ruby"
GROUP BY type
ORDER BY cnt DESC

/* watches for a specific language + date range */
SELECT repository_name, count(repository_name) as watches, repository_description, repository_url
FROM github.events
WHERE type="WatchEvent"
	AND repository_language="Ruby"
	AND PARSE_UTC_USEC(created_at) >= PARSE_UTC_USEC('2012-04-01 00:00:00')
GROUP BY repository_name, repository_description, repository_url
ORDER BY watches DESC

/* top 100 repos for Ruby by number of pushes */
SELECT repository_name, count(repository_name) as pushes, repository_description, repository_url
FROM github.events
WHERE type="PushEvent"
	AND repository_language="Ruby"
	AND PARSE_UTC_USEC(created_at) >= PARSE_UTC_USEC('2012-04-01 00:00:00')
GROUP BY repository_name, repository_description, repository_url
ORDER BY watches DESC
LIMIT 100

/* push events by language */
SELECT repository_language, count(repository_language) as pushes
FROM github.events
WHERE type="PushEvent"
	AND PARSE_UTC_USEC(created_at) >= PARSE_UTC_USEC('2012-04-01 00:00:00')
GROUP BY repository_language
ORDER BY pushes DESC

/* show recent push events for Go, sorted by time */
SELECT repository_name, repository_watchers, url, PARSE_UTC_USEC(created_at) as date
FROM github.events
WHERE type="PushEvent"
	AND repository_language="Go"
	AND repository_watchers > 1
	AND PARSE_UTC_USEC(created_at) >= PARSE_UTC_USEC('2012-04-01 00:00:00')
ORDER BY date DESC
```

For full schema of available fields to select, order, and group by, see schema.js.

## Manually loading the data

If you want to load the archive data into your own BigQuery project:

```bash
$> wget http://data.githubarchive.org/2012-03-11-15.json.gz
$> ruby transform.rb -i 2012-03-11-15.json.gz
$> python bq.py --apilog true load github.events 2012-03-11-15.json.gz-out.csv.gz schema.js
```