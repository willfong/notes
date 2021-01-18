# Cron

## crontab

`0 * * * * sh /path/to/script`

Make the cron script as simple as possible. Don't redirect script output to a log file in cron. Handle it in the script.

## script

```
#!/bin/bash

main() {
  date
  echo "hello
  date
}

main 2>&1 |tee /logs/output-$(date +%Y%m%y).log
```

Write the script as a function so log output can be handled. Logging to a file is the most basic way, but preferred to push it to ELK or S3.
