---

layout: post
title: csv to sqlalchemy_model & sql
category : Python
tags : [Python]

---

很久以前写的脚本，今天再用的时候差点没找到。。所以贴出来备份下。。

解析固定格式的csv表格，生成对应的sql语句和sqlalchemy model的py文件。

Python脚本:
```py
#!/usr/bin/env python
# encoding: utf-8


FIELD = 0
TYPE = 1
LENGTH = 2
PRIMARY = 3
NULL = 4
COMMENT = 5

TYPE_MAP = {
    'VARCHAR': 'String',
    'DATETIME': 'DateTime',
    'TINYINT': 'Integer',
    'DOUBLE': 'Float',
    'TEXT': 'Text',
    'INT': 'Integer',
}

HEAD = """ #!/usr/bin/env python
# encoding: utf-8

import _env  # noqa

import uuid
import datetime
import time

from sqlalchemy import Column, String, DateTime, Integer, Text, Float

from z42.config import IMG_HOST as FS_HOST, BP_PATH

from zapp.ANGELCRUNCH.models.mysql_base import Base
from zapp.ANGELCRUNCH.models.common_utils import Dict


"""

BASE = """class {class_name}(Base):
    \"\"\"{doc}\"\"\"
    __tablename__ = "{table_name}"

{columns}\n
"""

COLUMNS = """    {field} = Column({_type}{primary_key})  {comment}"""


SQL = """
TRUNCATE TABLE {table_name};
DROP TABLE IF EXISTS {table_name};
CREATE TABLE {table_name}
    (
{columns}
    ) ENGINE=InnoDB CHARSET=utf8;


"""

SQL_COLUMNS = """       {field} {_type}{length} {primary} {comment_new}"""


OUTPUT = 'models'
INPUT = 'data.csv'


def generate_file(all_tables):
    with open('%s.py' % OUTPUT, 'w') as f, open('%s.sql' % OUTPUT, 'w') as f1:
        f.write(HEAD)
        for table in all_tables:
            columns_str = ''
            sql_str = ''
            columns = table['columns']
            column = {}
            table_name = table['table_name'].split('_')
            class_name = []
            start = 0
            if table_name[0] == 'proj':
                class_name.append('Project')
                start = 1
            for name in table_name[start:]:
                new = name[0].upper()
                new += name[1:]
                class_name.append(new)
            table['class_name'] = ''.join(class_name)
            for c in columns:
                print c
                column['field'] = c[FIELD]
                column['_type'] = TYPE_MAP[c[TYPE].upper()]
                primary_key = c[PRIMARY]
                primary = c[PRIMARY]
                if primary_key:
                    primary_key = ', primary_key=True'
                    primary = 'PRIMARY KEY'
                column['primary_key'] = primary_key
                comment = c[COMMENT]
                comment_new = c[COMMENT]
                if comment:
                    comment_new = "COMMENT '%s'" % comment
                    comment = '# %s' % comment
                column['comment'] = comment
                columns_str += COLUMNS.format(**column)
                columns_str = columns_str.rstrip() + '\n'

                column.pop('comment')
                column.pop('primary_key')
                length = c[LENGTH]
                if length:
                    length = '(%s)' % length
                column['length'] = length
                column['_type'] = c[TYPE].upper()
                column['primary'] = primary
                column['comment_new'] = comment_new
                sql_str += SQL_COLUMNS.format(**column)
                sql_str = sql_str.rstrip() + ',\n'

            table['columns'] = columns_str
            f.write(BASE.format(**table))

            table['columns'] = sql_str.rstrip()[:-1]
            f1.write(SQL.format(**table))


def main():
    with open(INPUT) as f:
        Lines = f.readlines()
        breaks = [0]
        for i, line in enumerate(Lines):
            if not line.split(',')[0]:
                breaks.append(i)
        print breaks
        all_tables = []
        while(breaks):
            start = breaks.pop(0)
            if not breaks:
                lines = Lines[start:]
            else:
                lines = Lines[start: breaks[0]]
            i = 0
            table_name = ''
            doc = ''
            columns = []
            for line in lines:
                if not line.split(',')[0]:
                    i = 0
                    continue
                if i == 0:
                    tmp = line.split(',')
                    table_name = tmp[0]
                    doc = tmp[1]
                    i += 1
                    continue
                comment = None
                if '"' in line:
                    line, comment, _ = line.split('"')
                    print line
                    print comment
                line = line.split(',')
                field = line[FIELD]
                _type = line[TYPE]
                length = line[LENGTH]
                primary = line[PRIMARY]
                null = line[NULL]
                if not comment:
                    comment = line[COMMENT]
                columns.append([field, _type, length, primary, null, comment.strip()])
                i += 1

            print 'table_name', table_name
            print 'doc', doc
            all_tables.append({
                'table_name': table_name,
                'doc': doc,
                'columns': columns
            })

        print all_tables
        print '+++++++++++++++'
        generate_file(all_tables)

if __name__ == "__main__":
    import sys
    argv = sys.argv
    if len(argv) < 3:
        print '\033[34mUsage:\033[0m'
        print '''\tparser \033[34mInput \033[32mOutput\033[0m
\033[34mInput:\033[0m the .csv file you want to parse
\033[32mOutput:\033[0m the .py and .sql file you want to save

\033[32mExample:\033[0m parser data.csv models
\t will parse the data.csv file and output models.py and models.sql
        '''
    else:
        INPUT = argv[1]
        OUTPUT = argv[2]
        main()
```

用法：

```bash
marcoqin@marcoqin-begin  ~/Codes/csv2table  
╰─$ ./parser.py
Usage:
    parser Input Output
Input: the .csv file you want to parse
Output: the .py and .sql file you want to save

Example: parser data.csv models
     will parse the data.csv file and output models.py and models.sql
```

csv格式如下:

| table_name | table_description |
|:-----------|:-----------------:|--:|--:|--:|--:|
| column_name | column_type | column_length | is_primary_key | is_null | command |
|=====
| column_name | column_type | column_length | is_primary_key | is_null | command |
|=====
| ...
|=====
| (enpty line) |
|=====
| table_name | table_description |
|=====
| column_name | column_type | column_length | is_primary_key | is_null | command |
|=====
| column_name | column_type | column_length | is_primary_key | is_null | command |
|=====
| ...
|=====
{: rules="groups"}

example:

| proj_intention_order | 投资意向订单 |
|:-----------:|:-----------------:|:--:|:--:|:--:|:--:|
| order_id | VARCHAR | 64 | Y |  | 订单号 |
|=====
| target_id | VARCHAR | 64 |  | Y |  |
| ...
|=====
{: rules="groups"}
