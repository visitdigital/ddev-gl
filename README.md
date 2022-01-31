# DDEV pull wp scripts

🧘&nbsp; *Keep calm and pull your site.* 🧘

Open collection of DDEV pull scripts for happy local WordPress development: Pull in the latest site content to your local dev sever. Test, develop and design in peace without breaking your live site. These scripts require at least [DDEV 1.18.2](https://github.com/drud/ddev/releases/tag/v1.18.2).

Status: *Work in Progress, please use with caution.*

## Scripts

- ⚡️&nbsp; `ddev pull ssh` - *pull a site with just one command*<br>
- 💾 &nbsp; `ddev pull backup` - *download and import a BackWpUp .zip-file*

## Helpful resources

- Online config generator: [DDEV pull wp generator](https://mandrasch.github.io/ddev-pull-wp-generator/) ([Source code](https://github.com/mandrasch/ddev-pull-wp-generator/))
- 🎥  Screencasts: [ddev pull ssh](https://www.youtube.com/watch?v=lEGL65H-hts)
- [DDEV discord community](https://discord.gg/kDvSFBSZfs)

## Usage

## ⚡️&nbsp;  ddev pull ssh

Pull in your live WordPress site via SSH (and rsync). Your webspace needs support for mysqldump and you need to be able to connect to your SSH webspace via SSH key authentication (without password).

**First time project setup**

1. Copy `.ddev/example.config.yaml` to `.ddev/config.yaml`
   (You can as well use the [Online Generator](https://mandrasch.github.io/ddev-pull-wp-generator/))
1. Configure SSH host, user and WordPress path on server in `.ddev/config.yaml`
1. Run `ddev start`
1. Run `ddev auth ssh`
1. Test ssh connection with `ddev ssh production'` (custom command), use `exit` to leave ssh terminal afterwards. Make sure there is no message `No such file or directory` and use `pwd` to check, that you're in the correct directory.

Nice! You ready to pull!

**(Optional) Child theme setup**

*Skip these steps, if you don't have a child theme on your live site which you want to manage via git*

1. Configure Child theme folder name in `.ddev/config.yaml` 
1. Run `ddev restart`
1. Adjust child theme folder name in `.gitignore`
1. Download your current child theme to the repository (use the custom command `ddev ssh-production-download-child-theme` or use [plugin](https://de.wordpress.org/plugins/download-plugins-dashboard/), now you can git manage it. 
1. For pushing it see [WPPusher - Github Install Theme documentation](https://docs.wppusher.com/article/17-setting-up-a-plugin-or-theme-on-github)

**Pull in your latest site content**

1. Run `ddev pull ssh`

That's it, run `ddev launch` to open your site in the web browser. If you experience issues, see [Troubleshooting](#troubleshooting).

Source code: [.ddev/providers/ssh.yaml](https://github.com/mandrasch/ddev-pull-wp-scripts/blob/main/.ddev/providers/ssh.yaml)

If you want to clean and delete all pulled filles, you can use `git clean -fdx -e .ddev`. 

**Technical concept**

![](README_ddev_pull_ssh.png)


## 💾 &nbsp;ddev pull backup

Create and (manually) download a BackWpUp backup-file from your live site, import it to your local DDEV project. No SSH required, just download your backup file and import it as .zip-file.

**First time project setup**

1. Copy `.ddev/example.config.yaml` to `.ddev/config.yaml`
1. Configure Child theme folder name in `.ddev/config.yaml` (optional)
3. Adjust child theme folder name in `.gitignore` (optional)
4. Run `ddev start` and `ddev auth ssh`

**Import a backup:**

1. Create a backup with [BackWPup – WordPress Backup Plugin](https://wordpress.org/plugins/backwpup/) (Open Source) on your live site
1. Download BackWpUp .zip file to root of local project folder
1. Rename backup file to `backup.zip`
1. `ddev pull backup`
1. Open imported website in browser: `ddev launch`

Source code: [.ddev/providers/backup.yaml](https://github.com/mandrasch/ddev-pull-wp-scripts/blob/main/.ddev/providers/backup.yaml)

## Child theme deployment

You can optionally manage a child theme via git and deploy it with a tool like [WPPusher](https://wppusher.com/) (no git on server required). This repository already contains an example child theme. 

## Technical Documentation

Pull scripts are import steps for database and files which are stored as .yaml-files in `.ddev/providers`. They can be executed via `ddev pull`. See [DDEV pull (providers)](https://ddev.readthedocs.io/en/stable/users/providers/provider-introduction/) documentation for more insights.

 Design, test  and develop sites in local isolated [DDEV](https://ddev.readthedocs.io/en/stable/) project environments without breaking the live site.

### Reset / delete

- delete pulled files: `git clean -fdx -e .ddev`
    - removes all untracked files and directories (pulled files), but without deleting the DDEV folder
- reset DDEV project: `ddev delete -O`
    - deletes DDEV project with database and containers (files will be kept)
- `git clean -fdx`
    - removes all untracked files and directories
    - if .ddev/config.yaml is untracked in git, it will be removed as well (!)
    - you need to run 'ddev start' again afterwards to init

## Troubleshooting

### DDEV version correct?

Please check with `ddev -v` that you are using [DDEV 1.18.2](https://github.com/drud/ddev/releases/tag/v1.18.2) or newer.

### Database dump doesn't work?

If you use double quotes or single + double quotes mixed in wp-config.php, the script can't figure out the database credentials.

Bad: `define('DB_NAME', "db75994");`
Good - use only single quotes: `define('DB_NAME', 'db75994');`

(Debug this with a) connect to ssh `ddev ssh-production` and b) run `echo (cat wp-config.php | grep DB_NAME | cut -d \' -f 4)`, if this output is empty the bash parsing doesn't work properly for your site.) 

### ERR_TOO_MANY_REDIRECTS (apache)

If .htaccess has a https-only rule with something like 'RewriteCond %{HTTPS} !=on', it will result in 'ERR_TOO_MANY_REDIRECTS'. Just remove these rules from .htaccess. https://twitter.com/m_andrasch/status/1481290725694349316

### WP Super Cache warnings

```
/wp-super-cache/wp-cache-base.php): failed to open stream: No such file or directory in /var/www/html/wp-content/plugins/wp-super-cache/wp-cache.php on line 99`
Warning: include_once(/home/sites/site100026635/web/matthias-andrasch.eu/blog/wp-content/plugins/wp-super-cache/wp-cache-phase1.php): failed to open stream: No such file or directory in /var/www/html/wp-content/advanced-cache.php on line 22
```
Just login into wp-admin/-dashboard, WP Super Cache will repair the path or remove the `define('WPCACHEHOME',..` line from pulled wp-config.php manually after import. 

### Emojis get lost after import and are replaced with ????

This is a charset issue I wasn't able to fix automatically yet. Please check your live sites `wp-config.php` and look for `DB_CHARSET`. Set this value in config.yaml for `REMOTE_DB_CHARSET=`, run `ddev restart` and pull via ssh again (`ddev pull ssh`).

(Technical infos: It is related to DB_CHARSET set to something other than default 'utf8' in wp-config.php. For utf8mb4 the export needs to done with `mysqldump --default-character-set=utf8mb4` to be correct (https://stackoverflow.com/a/45301470/809939). But WordPress docs state DB_CHARSET setting may be not existent in wp-config.php, so we can't just read it from wp-config.php via bash, needs to be a conditional bash parsing from wp-config.php (See: https://wordpress.org/support/article/editing-wp-config-php/#database-character-set).)

## TODOs

- [ ] Use "- REMOTE_DB_CHARSET=utf8" as optional parameter, only use it in provider script if it is set
- [ ] Test SSH with password-based auth, should be working as well?
- [ ] Only single quotes are currently support with getting db values via bash (because of cut -d = delimiter is `'`)
- [ ] PHP 8.1 support? WP-CLI threw some errors, maybe fixed in new release? https://make.wordpress.org/cli/2022/01/26/wp-cli-v2-6-0-release-notes/

## Acknowledgements

Thanks to 
- DDEV maintainer [@randyfay](https://github.com/rfay) for helpful advice
- [@dahaupt](https://github.com/dahaupt) for awesome [advice on db sync](https://github.com/drud/ddev/discussions/2940#discussioncomment-1665163),
- [@BokuNoMaxi](https://github.com/BokuNoMaxi) for testing and collaboration on this
- All people in the WordPress & [DDEV community](https://discord.gg/kDvSFBSZfs) for sharing their knowledge online
- My colleagues at [gugler* MarkenSinn](https://www.gugler.at/markensinn).

## License

My code is released as [CC0 Public Domain](https://tldrlegal.com/license/creative-commons-cc0-1.0-universal), feel free to use it with or without attribution.
