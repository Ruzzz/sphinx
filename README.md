# Sphinx server with SQLite supports.

Based on https://github.com/aughey/sphinx-sqlite3
Current version based on Sphinx 2.2.11-release.

# Examle

Config file for Sphinx sphinx.conf:
```
source docs
{
    type                   = sqlite3
    sql_host               = localhost
    sql_user               = 
    sql_pass               =    
    sql_db                 = files.db
    sql_query              = SELECT docs.id AS id, repo_id, lang_id, docs.path as rel_path, \
                               repos.path || docs.path AS path \
                             FROM docs, repos \
                             WHERE (docs.repo_id = repos.id) AND (repos.flags & 1) = 0
    # If (repos.flags & 1) != 0 then repo is disable
    sql_file_field         = path
    sql_attr_string        = rel_path
    sql_attr_uint          = repo_id
    sql_attr_uint          = lang_id
}

index docs
{
    source                 = docs
    path                   = ./index/docs
    min_word_len           = 2
    #wordforms              = ./wordforms.txt
    #exceptions             = ./exceptions.txt
}

indexer
{
    max_file_field_buffer  = 24M    
}

searchd
{
    listen                 = localhost:5000:mysql41
    pid_file               = ./searchd.pid
    log                    = ./log/log.txt
    query_log              = ./log/query_log.txt
    binlog_path            = ./binlog/    
}
```

SQLite database files.db:
```
CREATE TABLE "repos" (
  "id" INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
  "name" TEXT,
  "path" TEXT NOT NULL,
  "flags" INTEGER DEFAULT 0
);

CREATE TABLE "langs" (
  "id" INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
  "name" TEXT NOT NULL
);

CREATE TABLE "docs" (
  "id" INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
  "repo_id" INTEGER NOT NULL,
  "lang_id" INTEGER DEFAULT 1,
  "path" TEXT NOT NULL,
  CONSTRAINT "fkey_lang" FOREIGN KEY ("lang_id") REFERENCES "langs" ("id")
    ON DELETE SET DEFAULT ON UPDATE CASCADE,
  CONSTRAINT "fkey_repo" FOREIGN KEY ("repo_id") REFERENCES "repos" ("id")
    ON DELETE CASCADE ON UPDATE CASCADE,
  UNIQUE ("repo_id" ASC, "path" ASC) ON CONFLICT IGNORE
);

INSERT INTO "langs" VALUES (1, 'Unknown');
INSERT INTO "repos" VALUES (1, null, 'd:\Docs\Repo1\', 0);
INSERT INTO "docs" (repo_id, path) VALUES(1, 'Folder1\Doc100.txt')
INSERT INTO "docs" (repo_id, path) VALUES(1, 'Folder2\Doc200.txt')
INSERT INTO "docs" (repo_id, path) VALUES(1, 'Folder3\Doc300.txt')
```

Run:
```
searchd.exe --config sphinx.conf
indexer.exe --config sphinx.conf --all --rotate
mysql.exe -P 5000
> SELECT * FROM docs WHERE MATCH ('text');
```
