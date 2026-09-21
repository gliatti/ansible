# cron

Installation of crons

## Tasks

Everything is in the `tasks/cron_manager.yml` file (included by `tasks/main.yml`).
Files are written to `/etc/cron.d/` and always belong to `root:root` (mode 0644),
as required by cron; the `user` field only sets the account the job runs as.

## Available variables

* `cron_schedules`: List of lines by crontab
  *  `user`: Account the job runs as (the file itself stays owned by root)
  *  `name`: Commentary
  *  `cron_file`: The file name under `/etc/cron.d/`
  *  `job`: The line to add
  *  `state`: The state of the line absent or present
  *  `disabled`: Tells if the line should be commented or not by true or false
  *  `minute`: The minute
  *  `hour`: The hour
  *  `day`: The day
  *  `month`: The month
  *  `weekday`: The day
