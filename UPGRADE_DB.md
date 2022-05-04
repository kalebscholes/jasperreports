1. Create the ddl in the jasperserver (if required)
```
cd <js-install>/buildomatic/install_resources/sql/postgresql
psql -U postgres -W
postgres=#create database jasperserver encoding=’utf8’;
postgres=#\c jasperserver;
postgres=#\i js-create.ddl
postgres=#\i quartz.ddl
postgres=#\q
```

Themes 
```bash
cd <js-install>/buildomatic
js‑ant import-minimal-ce
```

3. Run any migrations
Migrate the database 
```
js-ant upgrade-7.9-8.0-ce
```
