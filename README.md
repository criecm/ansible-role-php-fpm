# Role php-fpm

For Freebsd and Debian

Installs php-fpm and one or many pools using 'site' configs.

Calls webserver role (nginx by default) per site to create HTTP confs

## php.ini

php.ini vars can be forced with `php_ini_vars` dict
php-cli.ini additional vars can be set with `php_cli_vars` dict

## Role variables (default value)

* `monitoring_from` ([127.0.0.1]): host(s) or net(s) allowed to query status url's

* `fpm_priority` (12): default processes priority

* `fpm_error_log` (syslog): error log filename or "syslog"

* `fpm_log_level` (notice): log level (alert, error, warning, notice, debug)

* `php_fpm_umask` (): umask (FreeBSD only)

* `www_user` and `www_group` (system dependant)
  webserver user and group

* `php_version` (detected or 7.4): php version to install/use.

* `php_upgrade` (False): if `php_version` differs from installed, do we want to upgrade it ?

* `php_modules` ([]): list of modules to install

* `sites`: list of dicts, see above

* `mysite ('')`: if defined, only this site will be processed
  (use `ansible-playbook ... -e mysite=thisone` where `thisone` is your site's `id`)

## per-site variables (default value)

### mandatory

* `id` (mandatory, no default): Unique identifier on the host for this config, used for user, group, maintainer, dirsnames, …)

* `name` (mandatory, no default): DNS domain for this site

### optional (default value)

* `user (_{{id}})`: System user to run php

* `maintainer` ({{id}}): Maintenance user for the app (with shell and home)

* `maintainer_pass ()`: password for shell user

* `group (_{{id}})`: Common group for user and maintainer

* `home` (/home/{{id}}): Home of 'maintainer' and base for other default values (tmpdir, rootdir, ...)
  if a directory playbook/files/{{ id }}/home exists, it will be used as a template for the homedir
  
* `rootdir` (/home/{{id}}/{{id}}): php and webserver root directory

* `gits` ([]): list of dicts:
  list of git repositories with their details: for each element of the list you MUST provide:
  * `repo`: git URL to the repository
  and MAY provide (default):
  * `dest`: destination directory (`{{rootdir}}`)
  * `umask`: ('0022')
  * `maj`: whether to update existing repo (no)
  * `version`: tag or branch (master)
  * `owner`: owner of files (`{{maintainer}}`)
  * `group`: group for files (`{{group}}`)
  * `mode`: chmod -R (u=rwX,g=rX,o=rX)

* `configfiles` ([]) list of dicts:
  * `src`: template path, relative to playbooks dir (no default, ex:files/mytpl.j2)
  * `dest`: path on dest, absolute or relative to `{{rootdir}}`
  * `mode`: (0640)
  * `owner`: (`{{maintainer}}`)
  * `group`: (`{{group}}`)

* `crons` ([]): List of cron dicts (see ansible cron module for values)

* `webserver_role` (nginx): Which webserver role to use (see webserver's role's README.md for additional vars)

* `tmpdir` (/home/{{id}}/tmp): Base for tmp, sessions, uploads

* `memory_limit` (128): php [[http://php.net/manual/fr/ini.core.php#ini.memory-limit|`memory_limit`]] param, without 'M'

* `limit_openbasedir` (True): activate php `open_basedir` limitation: limits fopen to tmpdir+rootdir+`pear_path`+`open_basedir_add`

* `open_basedir_add` (''): path(es) to add to php `open_basedir` — string, separated by ':'

* `upload_max_meg` (20): base to compute php `post_max_size` and `upload_max_filesize`, in Mo

* `php_ini_vars ([])`: dict of default php.ini file
   * `date.timezone (host_timezone)`

* `php_cli_vars ([])`: dict of default php-cli.ini file (added to `php_ini_vars`)
   * `display_errors: on`
   * `display_startup_errors: on`

* `php_values` ({}): additional php values for php.ini

* `php_flags` ({}): additional php flags for php.ini

* `php_admin_values` ({}): additional forced `php_values`

* `php_admin_flags` ({}): additional forced `php_flags`

* `error_log` (`{{fpm_error_log}}`): error log file for this site

* `tmp_dir` ({{ tmpdir }}/tmp): TMPDIR, php tmpdir, ...

* `sessions_path` (/home/{{id}}/sessions): [[http://php.net/manual/fr/session.configuration.php#ini.session.save-path|session.save-path]]

* `upload_dir` (/home/{{id}}/tmp): [[http://php.net/manual/fr/ini.core.php#ini.upload-tmp-dir|upload-tmp-dir]]

* `pm` (ondemand): process manager model [[http://php.net/manual/en/install.fpm.configuration.php]]

* `pm_max` (200): max processes

* `pm_max_requests` (500): max requests served before restarting process (limits memory leaks)

* `pm_min` (2): for 'dynamic' pm: min processes waiting

* `pm_start` (5): for 'dynamic' pm: num processes started at boot

* `pm_max_spare` (10): for 'dynamic' pm: max idle processes

* `fcgi_status_path` (/fpmstatus.php): path to reach fpm's status report

  (restricted to `monitoring_from` nets by webserver)

* `debug` (False): activate `catch_workers_output` for this site

* `host_timezone ("Europe/Paris") Time zone for php
