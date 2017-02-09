# Flow

If your recipe based on *common* recipe or one of frameworks recipe shiped with Deployer, then you are using one of our default flows.
Each flow described as group task of other tasks in `deploy` name space. Common deploy flow may look like this:

```php
task('deploy', [
    'deploy:prepare',
    'deploy:lock',
    'deploy:release',
    'deploy:update_code',
    'deploy:shared',
    'deploy:writable',
    'deploy:vendors',
    'deploy:clear_paths',
    'deploy:symlink',
    'deploy:unlock',
    'cleanup',
    'success'
]);
```

Frameworks recipes may diff in flow, but basic structure is same. You can create your own flow by overriding `deploy` task, but better slution is to using cache. 
For example if you want to run some task before symlink new release:

```php
before('deploy:symlink', 'deploy:build');
```

Or to do some notifications after success deploy:

```php
after('success', 'notify');
```

In next section i will give short overview of each task. 

### deploy:prepare

Preeparation for deployment. Checks if `deploy_path` exists, otherwise create it. Also checks for existing of next paths:

* `releases` – in this dir will be stored releases.
* `shared` – shread files across all releases.
* `.dep` – metadata used by Deployer.

### deploy:lock

Locks deployment. So only one concurent deployment can running. To lock deployment, task check of existing `.dep/deploy.lock` file. In deploy process was canceld by Ctrl+C, run `dep deploy:unlock` t odelete this file. In case if deployment was filed `deploy:unlock` task will be triggered automatically. 

### deploy:release

Create new release folder based on `release_name` config. Also reading `.dep/releases` to get list of releases what was created before.

### deploy:update_code

Upload new version of code using git. If using git version 2.0 and `git_cache` config is turned on, this task will be using files from previuos release, so only changed files will be downloaded.

### deploy:shared

Creates shared files and dirs from `shared` dir to `release_path`. You can specify shared dirs and files in `shared_dirs` and `shared_files` config. Process splitted into a few steps:

* Copy dir from `release_path` to `shared` if doesn't exists,
* delete dir from `release_path`,
* symlink dir from `shared` to `release_path`.

Same steps for shared files. If your system support relative symlinks them will be used, otherwise absolutle symlinks wil be used.