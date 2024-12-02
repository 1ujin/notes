# 三范式

数据库的三范式（Three Normal Forms）是一种设计规范，用于规范化关系型数据库中的数据结构，以减少数据冗余和提高数据的一致性。

1. 第一范式（1NF）：要求关系表中的每个属性都是原子的，不可再分。也就是说，每个属性不能包含多个值或多个属性。通过将多值属性拆分为单值属性，可以消除数据冗余和复杂性。
2. 第二范式（2NF）：在满足 1NF 的基础上，要求表中的非主键属性完全依赖于主键。换句话说，非主键属性必须完全依赖于候选键（主键）。如果存在部分依赖，即一个非主键属性依赖于候选键的一部分属性，就需要将其拆分为独立的关系表。
3. 第三范式（3NF）：在满足 2NF 的基础上，要求表中的非主键属性之间没有传递依赖关系。如果存在传递依赖，即一个非主键属性依赖于另一个非主键属性，就需要将其拆分为独立的关系表。

# 基本语法

### 删除操作：

1. `DROP`删除整个表，DDL 语句，不能被回滚；
2. `TRUNCATE`删除全部数据不删除表，比`DELETE`快，DDL 语句，不能被回滚；
3. `DELETE`删除部分数据，DML 语句，会触发行级触发器。

### 配置

查看配置：

```mysql
SELECT @@global.character_set_database;
SELECT @@session.character_set_database;
SELECT @@character_set_database; -- 等价于会话

SHOW global variables LIKE 'character_set_database';
SHOW session variables LIKE 'character_set_database';
SHOW variables LIKE 'character_set_database'; -- 等价于会话
```

修改配置：

```shell
SET global transaction isolation level read committed;
SET session transaction isolation level read committed;
SET transaction isolation level read committed;

SET global transaction_isolation='repeatable-read';
SET session transaction_isolation='repeatable-read';
SET transaction_isolation='repeatable-read';

SET @@global.transaction_isolation='repeatable-read';
SET @@session.transaction_isolation='repeatable-read';
SET transaction_isolation='repeatable-read';
```



# 索引

适合作索引的字段特点：

1. 常用作条件查询；
2. 常用作排序；
3. 常用作连接；
4. 高区分度、分布均匀；
5. 不会频繁改动。

### 创建索引

三种方式：

1. 建表时创建：

   ```mysql
   create table user_index(
       id  int  auto_increment   primary key,
       first_name   varchar(16),
       last_name   varchar(16),
       id_card        varchar(18),
       information   text(225),
       key    name( first_name,last_name),
       fulltext   key(information),
       unique key(id_card)
   );
   ```

2. `create`创建：

   ```mysql
   create index index_name on table_name(column_list);
   ```

3. `alter`添加：

   ```mysql
   alter table table_name add index index_name(column_list);
   ```

### 删除索引

```mysql
alter table table_name drop key idx_name;
```

### 普通索引

也叫非唯一索引，是最普通的索引，没有任何的限制。

### 主键索引

主键索引是所有 InnoDB 表必须的，且一个表中只能有一个主键索引。InnoDB 的数据文件就是按照主键顺序存放的，也就是聚簇索引。主键索引的选择对查询的性能有很大的影响。

### 唯一索引（Unique Index）

唯一索引要求键值不能重复，但可以有空值。另外需要注意的是，主键索引是一种特殊的唯一索引，它还多了一个限制条件，要求键值不能为空。主键索引用 primay key 会自动创建。如果是组合索引，则**组合的值**必须唯一。

### 回表查询

根据数据库行的具体地址来找到对应行数据，一般在非聚簇索引中需要回表查询。

#### 1. **索引选择与执行计划**

在 MySQL 中，查询的执行计划是由 **查询优化器**根据不同的因素来选择的。优化器会评估不同的访问路径，选择最合适的一种来执行查询。这里的“访问路径”就是指如何通过索引或者全表扫描来获取数据。

##### a. **使用索引的情况**

假设我们有一个名为 `employees` 的表，并且在 `name` 列上创建了一个索引 `idx_name`。如果执行以下查询：

```sql
SELECT * FROM employees;
```

在没有 `WHERE` 子句的情况下，MySQL 会考虑不同的执行计划，并且有可能选择使用 `idx_name` 索引进行查询，即使最终返回的数据包含所有列。

#### 2. **为什么会选择 `idx_name` 索引而不是主键索引？**

尽管表的主键索引包含了所有数据（因为它是聚簇索引），但在某些情况下，MySQL 查询优化器可能会选择使用 **非主键索引** 来进行扫描，原因如下：

##### a. **索引的覆盖性**

- 如果查询使用的是 `idx_name` 这样的非主键索引，并且这个索引已经足够覆盖查询的需求（即可以通过索引中的内容来定位行），那么优化器可能会选择使用这个索引。 
- **注意**：在没有条件查询的情况下，使用非主键索引（例如 `idx_name`）并不意味着直接访问所有列，但在某些情况下，优化器仍然可能选择它，因为它是优化器认为成本最优的选择。

##### b. **选择扫描方式的因素**

- **`idx_name` 索引可能比全表扫描或者主键索引更有优势**，因为主键索引的聚簇特性要求它按顺序存储数据，而对于某些查询，使用非主键索引可能会减少扫描的成本，尤其是当有大量列时。
- 使用 `idx_name` 索引时，优化器可能会通过索引扫描找到符合条件的记录（索引中的列和主键值），然后再通过主键回表获取其他列的数据。这是一个典型的“回表”场景。

##### c. **回表操作**

- 如果 MySQL 决定通过 `idx_name` 索引扫描数据，**它会扫描 `idx_name` 索引，首先根据 `name` 列值查找到符合条件的行，然后通过主键值去回表获取其他列的数据**。这个过程是因为 `idx_name` 只包含 `name` 列和主键值，而不包含其他列的数据。

#### 3. **为什么会通过主键值回表？**

假设查询的是 `SELECT * FROM employees;`：

1. 如果查询使用了 `idx_name` 索引，**这个索引仅存储 `name` 列和对应的主键值**。

2. **`idx_name` 不包含所有列的数据**，所以，查询返回所有列时，必须使用主键值回到数据表（通过主键索引）获取其他列的数据。

3. 如果查询完全依赖于 **主键索引**（即你查询的列都在主键索引中），那么就可以直接通过主键索引获取完整的行数据，而不需要回表。

#### 4. **为何可能选择非主键索引而非主键本身？**

有时候，即便主键索引可以提供完整的数据行，MySQL 的优化器可能依然会选择非主键索引，原因可能包括：

- **索引的结构和扫描成本**：如果非主键索引（例如 `idx_name`）提供了较低的扫描成本，优化器可能会选择它，即使这意味着必须回表。
- **数据分布和索引选择性**：索引的选择性可能影响查询计划的选择。如果 `name` 列有更高的选择性，意味着使用 `idx_name` 进行扫描可能更高效。

#### 5. **总结**

- **主键索引**：是聚簇索引，包含了所有数据列，因此对于包含所有列的查询，MySQL 可以直接通过主键索引获取数据，无需回表。

- **非主键索引**：如果使用了像 `idx_name` 这样的非主键索引，它只包含 `name` 列和主键值。当查询需要返回 `*` （所有列）时，MySQL 需要回表，通过主键索引去获取其他列的数据。

- **回表的选择**：即便在查询中没有条件（如 `SELECT *`），如果查询优化器认为使用非主键索引（如 `idx_name`）会更有效率，它也可能选择这样做，然后通过回表操作来获取其他列的数据。

- **回表的原因**：通过非主键索引无法直接获取所有列的数据，只有主键索引才能提供完整的数据行，因此非主键索引查询时必须回表。

### 聚簇索引（Clustered Index）

**聚簇索引**（Clustered Index）是一种**数据存储方式**，它将数据的物理存储顺序与索引顺序相同。也就是说，表中的数据行按照索引键的顺序进行存储。聚簇索引通常用于数据库系统中，以提高数据查询的效率。**非主键索引**在 InnoDB 中大多数情况下就是**非聚簇索引**。非主键索引通常又被称为**二级索引**（Secondary Index）。

特点：

1. 物理存储顺序与索引顺序相同：

   - 在聚簇索引中，数据行按照索引键的顺序存储在磁盘上。也就是说，数据的物理位置是按照索引的顺序排列的。

   - 这意味着索引中的每个键值对应的数据是实际的数据行，而不是指向数据行的指针。

2. 每张表只能有一个聚簇索引：

   - 由于聚簇索引要求数据按照某个键的顺序物理存储，因此每个表只能有一个聚簇索引。因为数据只能按照一种顺序存储。

   - 其他索引可以是**非聚簇索引**（Non-clustered Index），即索引和数据存储是分开的。

3. 提高查询效率，特别是范围查询：
   - 聚簇索引特别适合范围查询（例如 `BETWEEN`、`ORDER BY` 等），因为数据是连续存储的。数据库可以只进行一次磁盘读取，然后顺序地读取数据，而不是像非聚簇索引那样每次都要根据索引指针跳转到不同的存储位置读取数据。

4. 插入和更新的代价较高：
   - 由于数据的物理存储顺序要和索引顺序保持一致，因此在插入新数据或更新数据时，可能需要重新排列数据的物理存储顺序。这使得聚簇索引在频繁插入和更新的情况下性能较低。

聚簇索引与非聚簇索引的区别：

| 特性           | 聚簇索引                         | 非聚簇索引                       |
| -------------- | -------------------------------- | -------------------------------- |
| 数据存储方式   | 数据按照索引的顺序存储           | 索引存储与数据存储分离           |
| 每表的索引数量 | 每个表只能有一个聚簇索引         | 可以有多个非聚簇索引             |
| 叶子节点       | 叶子节点存储的是实际的数据行     | 叶子节点存储的是指向数据行的指针 |
| 查询性能       | 对于范围查询特别高效             | 查询性能相对较低                 |
| 插入/更新性能  | 插入和更新可能会导致数据重新排列 | 插入和更新不会影响数据的物理顺序 |

适用场景：

- **聚簇索引** 适合读多写少、且需要频繁进行范围查询的场景，如数据分析、日志查询等。
- **非聚簇索引** 更适合频繁的随机读写操作，尤其是在没有明确的排序要求时。

### 覆盖索引

在一次查询中，如果一个索引包含或者说覆盖**所有**需要查询的字段的值，我们就称之为覆盖索引，而不再需要**回表查询**。而要确定一个查询是否是覆盖索引，我们只需要`EXPLAIN`语句看`Extra`列的结果是否是`Using index`即可。

### 全文索引（Fulltext Index）

在 MySQL 5.6 版本以前，只有 MyISAM 存储引擎支持全文引擎。在 5.6 版本中，InnoDB 加入了对全文索引的支持，但是不支持中文全文索引。在 5.7.6 版本，MySQL 内置了 ngram 全文解析器，用来支持亚洲语种的分词。但是，InnoDB 的全文索引在功能和性能上与 MyISAM 存在差距，如需对全文索引的性能要求较高，或者对全文索引的更高级功能有所要求，建议使用MyISAM存储引擎。

建表时创建：

```mysql
CREATE TABLE articles (
    id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
    title VARCHAR (200),
    body TEXT,
    FULLTEXT (title, body) WITH PARSER ngram
) ENGINE = INNODB DEFAULT CHARSET=utf8mb4 COMMENT='文章表';
INSERT INTO articles (title, body) VALUES ('弘扬正能量', '贯彻党的18大精神');
INSERT INTO articles (title, body) VALUES ('北京冬奥会', '2022年北京冬奥会于2022年2月20日闭幕啦');
INSERT INTO articles (title, body) VALUES ('MySQL Tutorial', 'DBMS stands for Database');
INSERT INTO articles (title, body) VALUES ('IBM History', 'DB2 history for IBM');
```

现有表字段添加：

```mysql
ALTER TABLE articles ADD FULLTEXT INDEX title_body_index (title,body) WITH PARSER ngram;
```

使用全文索引：

```mysql
-- 创建一个包含文章的表
CREATE TABLE articles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255),
    body TEXT,
    FULLTEXT(title, body)  -- 在标题和正文字段上创建全文索引
) ENGINE=InnoDB;

-- 插入一些数据
INSERT INTO articles (title, body) VALUES
('MySQL Fulltext Search Example', 'This is an example of MySQL FULLTEXT search.'),
('Understanding Fulltext Search', 'Learn how to use MySQL FULLTEXT search for text-based queries.'),
('Fulltext and Inverted Indexes', 'This article compares FULLTEXT search with inverted indexes.');

-- 使用全文搜索进行查询
SELECT * FROM articles
WHERE MATCH(title, body) AGAINST ('MySQL search' IN NATURAL LANGUAGE MODE);
```

解释：

- **`FULLTEXT(title, body)`**：在 `title` 和 `body` 字段上创建全文索引。
- **`MATCH ... AGAINST`**：用于执行全文搜索，`NATURAL LANGUAGE MODE` 是 MySQL 支持的一种全文搜索模式。
- 在上述例子中，MySQL 使用全文索引来搜索包含关键词 "MySQL" 和 "search" 的文章。

### 倒排索引（Inverted Index）

MySQL **没有内置**的倒排索引，不过可以通过一些技巧来模拟倒排索引。通常，我们可以将关键词提取并存储在一个独立的表中，然后通过该表实现类似倒排索引的功能。

示例：模拟倒排索引

假设我们有一个包含文档的表 `documents`，并且我们希望为每个文档创建一个倒排索引来加速关键词搜索。

```mysql
-- 创建包含文档的表
CREATE TABLE documents (
    id INT AUTO_INCREMENT PRIMARY KEY,
    content TEXT
);

-- 插入一些文档
INSERT INTO documents (content) VALUES
('This is an example of a document.'),
('This document discusses MySQL and Inverted Indexes.'),
('Fulltext search and inverted indexes are important.');

-- 创建倒排索引表，记录每个关键词与文档的关系
CREATE TABLE inverted_index (
    word VARCHAR(255),
    document_id INT,
    PRIMARY KEY (word, document_id),
    FOREIGN KEY (document_id) REFERENCES documents(id)
);

-- 手动向倒排索引表插入数据（通常由文本分析工具生成）
INSERT INTO inverted_index (word, document_id) VALUES
('example', 1),
('document', 1),
('document', 2),
('mysql', 2),
('inverted', 2),
('index', 2),
('fulltext', 3),
('inverted', 3),
('index', 3);

-- 查询包含关键词 'document' 的文档
SELECT d.id, d.content FROM documents d
JOIN inverted_index i ON d.id = i.document_id
WHERE i.word = 'document';
```

解释：

- **`inverted_index` 表**：这张表充当倒排索引，记录每个关键词与文档 ID 的对应关系。
- **`JOIN inverted_index ON document_id`**：通过倒排索引表来查询包含特定关键词的文档。
- 虽然这种方式是模拟倒排索引，但它可以高效处理特定关键词的查询。

### 空间索引（Spatial Index）

R 树。略。

### 底层数据结构

#### B+ 树（加强多路平衡查找树）

![B+树](./assets/7c271d0d4c0bbf9901797a01c7339c0d.jpeg)

大部分 MySQL 存储引擎的默认索引类型。B+Tree 是一种**平衡多路查找树**，可以保证数据的**有序性**，并且有较高的查找效率。比如 InnoDB 存储引擎就采用的 B+Tree 索引。在 B+Tree 索引中，索引项是按照顺序排列并分布在树上的，这样对范围查询和排序就有了很大的优势。**叶子节点才真正存放数据，非叶子节点只存放键**，键存的更多，高度更低，减少磁盘IO；叶子节点形成**链表**，可以快速遍历。

[MySQL 索引原理详解](https://blog.csdn.net/weixin_41716183/article/details/126089315)

分表：

#### Hash

![Hash索引](./assets/2cdceeafe45789bb1e0c8635ff3b1011.png)

Memory 存储引擎的索引就采用了 Hash 索引，适用于**等值查询**，但不支持**范围查询和排序**等操作。Hash 索引的查询速度非常快，但是索引的**维护成本较高**，而且 Hash 冲突的存在也会影响查询性能。

#### 为什么不用 AVL 树（平衡二叉树）或红黑树？

![AVL树](./assets/a5d77b87403c6b4cb0418058ebc3320e.png)

AVL 树**左右子树深度差绝对值不能超过 1**。

红黑树的种种约束保证的是什么？**最长路径不超过最短路径的二倍**。不太适合于数据库索引。适合内存的数据机构，例如实现一致性哈希：

- **节点度与磁盘访问**：AVL 树和红黑树的每个节点**只包含两个子节点**（即度为 2），这意味着树的**高度相对较大**。由于数据库中的数据通常存储在磁盘上，每次从磁盘读取数据都会引发一次 I/O 操作。树的高度越高，意味着进行查询时需要访问的磁盘页面越多，导致**磁盘 I/O 次数增加**，查询效率下降。（==子节点少 → 高度大 → I/O多==）
- **平衡开销**：AVL 树为了保持严格的平衡，插入和删除操作需要**频繁的旋转**操作，这增加了维护平衡的开销。红黑树虽然对平衡的要求没有 AVL 树严格，但仍然需要进行旋转操作，尤其是在频繁插入和删除的情况下，**维护成本较高**。（==频繁旋转 → 维护成本高==）

因为 B Tree 和 B+Tree 的特性，它们广泛地用在文件系统和数据库中，例如 Windows 的 HPFS 文件系统，Oracel、MySQL、SQLServer 数据库。

#### 为什么不用 B 树（多路平衡查找树）

![B树](./assets/dc6bbc131841b6df77234c70f5b5b704.jpeg)

B 树相比二叉树更适合数据库索引，原因是它是**多叉平衡树**（度数 > 2），它**减少了树的高度**，从而**减少了磁盘 I/O**。然而，B 树与 B+ 树相比仍存在一些劣势：

- **数据存储**：B 树的**每个节点都存储键和数据**，而 B+ 树的**非叶子节点只存储键**，所有**数据都存储在叶子节点上**。因此，B+ 树的非叶子节点可以存储更多的键（InnoDB 页大小默认 16 KB / 每个键大小 = 每个节点的键数量），每个节点有更多的子节点，从而使树的**高度更低**，进一步**减少了磁盘 I/O**。（==叶子存数据 → 非叶子存更多键 → 增加 Degree，降低高度 → 减少IO==）

  > 一般来说，索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储的磁盘上。这样的话，索引查找过程中就要产生磁盘I/O消耗，相对于内存存取，I/O存取的消耗要高几个数量级，所以评价一个数据结构作为索引的优劣最重要的指标就是在查找过程中磁盘I/O操作次数的渐进复杂度。换句话说，索引的结构组织要尽量减少查找过程中磁盘I/O的存取次数。
  >

- **范围查询**：B+ 树的所有叶子节点通过指针相连，形成一个链表结构，因此在做范围查询时，可以通过叶子节点之间的链表快速遍历数据。而 B 树由于数据分布在各个节点，进行范围查询时需要在树上进行频繁的查找和遍历，效率较低。（==叶子节点链表==）

### 索引失效

1. 使用`!=`或`<>`操作符：查询的结果集大于整个表的大约 30%，索引对期待**扫描全部**数据的查询通常没有帮助，尤其是不等式查询。
2. 对索引列进行**计算或函数**操作：如果对一个索引列进行函数操作，那么引擎将无法使用索引，因为它必须对每个行执行函数操作后才能比较结果。例如：`SELECT * FROM table WHERE YEAR(date_column) = 2022;`
3. 使用`LIKE`操作符**以`%`开头的模糊查询**：当`LIKE`的模式值以通配符`%`开头时，无法使用索引，因为查询引擎无法知道搜索结果在何处开始或结束。例如：`SELECT * FROM table WHERE column LIKE '%Z';`
4. 联合索引中使用**最左前缀原则**。像 `(col1, col2, col3)` 这样的联合索引只有在查询条件在索引树左侧时才能够被用到。比如查询 `col1` 或 `col1, col2`，索引是起效的。但当查询 `col2` 或者 `col3` 或者 `col2, col3`这样，该索引就不起作用了。
5. **数据类型不一致**：如果查询中的数据类型与索引中的数据类型不一致，MySQL 将无法使用索引。

### 索引提示（Index Hint）

`index hint`：索引提示，是一种优化手段，通过嵌入 sql 中告知 MySQL 如何选择索引：

- `use index`：指定索引。如果优化器认为全表扫描更快，会使用全表扫描，而非指定的索引；
- `force index`：强制指定索引。即使优化器认为全表扫描更快，也不会使用全表扫描，而是用指定的索引；
- `ignore index`：忽略指定索引。

#### 指定索引（Use Index）

```sql
SELECT cols_list FROM table_name USE INDEX(index_list) WHERE condition;
```

在 MySQL 中，使用`USE INDEX`语句的主要使用场景如下：

1. 强制使用特定的索引：当查询优化器选择了不太高效的索引，或者根本没有选择索引时，可以使用`USE INDEX`语句来指定要使用的索引。特别适用于分析和调试查询性能问题。
2. 测试不同索引的性能：在有多个可选的索引时，可以使用`USE INDEX`语句来临时替换不同的索引，以评估它们对查询性能的影响，从而选择最佳的索引策略。
3. 绕过索引选择器：当 MySQL 选择了不适合的索引时，可能会引起性能问题。使用`USE INDEX`语句可以绕过索引选择器，强制使用指定的索引。

`USE INDEX`的限制和注意事项：

- 语法限制：`USE INDEX`语句的语法是使用在**`SELECT`**语句中，用于指定要使用的索引。例如：`SELECT * FROM table_name USE INDEX (index_name) WHERE condition;`需要确保语法正确并且正确指定了要使用的索引。
- 仅适用于单表查询：`USE INDEX`语句只适用于**单表查询**，无法用于涉及多个表的联接查询。如果查询涉及多个表，可以考虑使用`FORCE INDEX`语句。
- 仅适用于当前查询：`USE INDEX`语句只适用于当前查询，不会对表的全局索引使用产生影响。它仅在当前查询中强制使用指定的索引。
- 索引必须存在：使用`USE INDEX`语句时，需要确保指定的索引存在于表中。如果指定了不存在的索引，将会**抛出错误**。
- 超过索引的列不能使用：`USE INDEX`语句只能指定索引的使用，不能指定查询中其他列的使用。如果查询需要使用的列不在指定的索引中，优化器可能会选择其他索引或执行全表扫描。
- 潜在性能问题：尽管`USE INDEX`可以用于指定索引，但过度使用`USE INDEX`可能会导致查询性能下降。优化器通常能够更好地评估和选择索引，使用`USE INDEX`可能会限制其优化能力，导致不必要的索引使用和性能损失。
- 谨慎使用：使用`USE INDEX`语句时需要谨慎评估和测试。建议在实际场景中进行性能测试和比较，确保使用`USE INDEX`确实能够提供更好的性能，而不是盲目使用。
- 即使建议使用索引，它也**完全取决于查询优化器**根据该索引的使用情况决定是否使用给定索引。

#### 强制索引（Force Index）

```sql
SELECT * FROM table_name FORCE INDEX(index_name) WHERE condition;

UPDATE table_name FORCE INDEX(index_name) SET column_name = value WHERE condition;
```

在 MySQL 中，`FORCE INDEX`是一种查询提示，用于强制查询优化器使用特定索引来执行查询。查询优化器在执行查询时，会根据统计信息和查询条件等来选择最优的执行计划，包括选择哪个索引来提高查询性能。但有时候查询优化器可能会选择非最优的索引，或者无法识别最适合的索引，这时可以使用`FORCE INDEX`来指定使用某个索引。

使用`FORCE INDEX`需要提供需要使用的索引的名称，可以是单个索引，也可以是多个索引，用逗号隔开。MySQL 将强制使用指定的索引来执行查询，即使查询优化器可能认为其他索引更加适用。

`FORCE INDEX`可以用于`SELECT`和`UPDATE`语句中，通过在查询语句或更新语句中添加`FORCE INDEX`子句来指定使用的索引。使用`FORCE INDEX`可能会显著提高查询性能，但也有可能导致性能下降，因此需要谨慎使用。

`FORCE INDEX`可以用于以下场景：

1. 优化查询性能：当查询语句的性能较低，查询优化器无法选择最优的索引时，可以使用`FORCE INDEX`来指定使用特定的索引。通过强制使用指定的索引，可以提高查询性能。
2. 跳过不必要的索引扫描：有时候查询语句可能会选择错误的索引，导致不必要的索引扫描。使用`FORCE INDEX`可以确保查询器直接使用指定的索引进行查询，避免了额外的索引扫描。
3. 强制查询器使用覆盖索引：覆盖索引是指查询恰好可以使用索引来获取所需的所有列，而不需要回表查找对应的行记录。如果查询器没有选择使用覆盖索引，可以使用`FORCE INDEX`强制查询器使用覆盖索引，从而提高查询性能。
4. 模拟索引失效的情况：在一些情况下，可能想要测试某个查询在没有某个索引的情况下的性能。可以使用`FORCE INDEX`来模拟索引失效的情况，从而比较有索引和无索引的性能差异。

需要注意的是，`FORCE INDEX`可能会导致查询性能下降，特别是当指定的索引并不是最适合的索引时。因此，在使用`FORCE INDEX`时需要谨慎评估和测试，确保其确实能够提高查询性能。

#### 忽略索引（Ignore Index）

```sql
SELECT * FROM table_name IGNORE INDEX(index_list);
```

提示会禁止查询优化器使用指定的索引。在具有多个索引的查询时，可以用来指定不需要优化器使用的那个索引，还可以在删除不必要的索引之前在查询中禁止使用该索引。

### 跳跃扫描（Index Skip Scan）

索引跳跃扫描是一种优化查询的技术，尤其在联合索引中用于减少扫描的无效行数。它通过“跳跃”式的扫描方式，避免了对索引中无用部分的扫描，从而提升查询效率。这种技术适合特定场景，并有一定的优缺点。

索引跳跃扫描利用的是联合索引中非首列（非最左前缀）的索引列，来提高查询效率。例如，如果你有一个复合索引`(A,B)`，在传统的 B-Tree 索引中，只有当查询条件包含 A 列时，索引才会生效。但在跳跃扫描中，即使 A 没有出现在查询条件中，仍然可以通过扫描 B 列来有效查询。

跳跃扫描会逐步扫描 A 列的每一个可能值，然后在每个 A 值下查找 B 列中符合条件的记录。**以 A 列中出现的所有值划分区域，如果当前 A 列值的“分区”中的后续记录已经确定不可能符合，则跳过整个“分区”。**这样避免了扫描大量无关记录，提升了查询性能。

**优点**：

- 提高查询效率：对于联合索引，如果查询条件只涉及非最左前缀列，跳跃扫描能够提高查询效率，减少全表扫描的次数。
- 减少 I/O 操作：通过避免扫描无效的索引行，跳跃扫描减少了对数据页的访问，从而节省了 I/O 操作。
- 降低索引空间要求：在某些场景下，可以减少为查询额外建立索引的需求，因为即使只使用了非首列，跳跃扫描也能利用现有的复合索引。

**缺点**：

- 不适合高基数列：跳跃扫描对低基数列（值不多但重复率高的列）有较好的效果。但如果参与跳跃扫描的列基数高，可能需要大量跳跃，反而影响效率。
- 无法替代覆盖索引：对于那些经常查询的列，跳跃扫描并不能代替为每个列创建单独的索引。对于常用列，覆盖索引的效果会更好。
- 不适用于所有查询类型：跳跃扫描仅在某些查询模式下有效，特别是当查询条件中不包含索引的最左前缀列时。如果最左列经常被查询，跳跃扫描无法发挥作用。

**适用场景**，索引跳跃扫描通常适用于以下场景：

- 联合索引查询：当查询条件不包括索引的最左前缀列，而仅包括后面的列时，可以使用跳跃扫描。
- 低基数列查询：对于列值种类少、重复率高的列，跳跃扫描可以减少扫描无效记录的时间。
- 避免额外索引：当现有的联合索引足够支持查询，而不想为特定列额外创建索引时，跳跃扫描是一种权衡。

**触发条件**，索引跳跃扫描并不是总能触发，通常需要满足以下条件：

- 有一个复合索引：该复合索引需要包含多个列，例如`(a,b)`或`(a,b,c)`。
- 查询条件不包含最左列：查询中使用了联合索引的非最左列。例如，查询中只使用了 b 列，而没有 a 列。
- 索引的区分度较低：跳跃扫描往往适用于索引的最左列的重复值较多的情况，因为这时跳过部分记录的开销较低。

**触发技巧**，尽管 MySQL 会自动决定是否使用索引跳跃扫描，但有一些 SQL 编写和索引设计的技巧可以引导 MySQL 更好地使用这种优化：

- 设计联合索引：创建适合查询的联合索引，例如 (a, b)，这样在查询条件不包含 a 但包含 b 时，MySQL 可能会使用跳跃扫描。

- 避免使用最左列：如果你希望 MySQL 使用跳跃扫描，查询中不应该使用联合索引的最左列。例如：

  ```sql
  SELECT * FROM table WHERE b = 'value';
  ```

- 使用 EXPLAIN 查看执行计划：可以通过 EXPLAIN 查看 MySQL 的执行计划，看看是否触发了索引跳跃扫描优化。

  ```sql
  EXPLAIN SELECT * FROM table WHERE b = 'value';
  ```


  在执行计划中，如果看到索引部分显示使用了联合索引，并且查询条件没有最左列，说明可能触发了跳跃扫描。

# 事务

## 隔离级别

“魔高一尺，道高一丈”：读未提交 < 脏读 < 读已提交 < 不可重复读 < 可重复读 < 幻读 < 可串行化

<img src="./assets/561d2fc32dc3b3dd8d87c9a985918875.png" alt="MySQL Innodb 啥时候用表锁，啥时候用行锁？" style="zoom: 33%;" />

### 读未提交（Read Uncommitted）

**可以读取到其他事务还没提交的数据。** 在这个隔离级别下，由于可以读取到未提交的值，因此会产生「脏读」问题。此隔离存在排他锁，两个事务无法同时修改，后修改的会阻塞。此隔离**读取数据不需要加锁**，所以可以一边写，另一边读，可能造成脏读。

```mysql
-- 设置事务隔离级别为 READ UNCOMMITTED
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

### 脏读（Dirty Read）

未提交 + 回滚/修改

举个例子：A 事务更新了 price 为 30，但还**未提交**。此时 B 事务读取到了 price 为 30，但后续 A 事务**回滚**了，那么 B 事务读取到的 price 就是错的（脏的）。

事务 2 可以读取事务 1 还没提交的数据，一旦后者**回滚**或再次修改就会造成错误：

<img src="./assets/1633379-20230924102826381-2084756780.png" alt="img" style="zoom:50%;" />

### 读已提交（Read Committed）

读写混合操作较多时推荐该级别。

**只能读到其他事务已经提交的数据。** 这个隔离级别解决了脏读的问题，不会读到未提交的值，但是却会产生「不可重复读」问题。

同样是记录锁，但是

```mysql
-- 设置事务隔离级别为 READ COMMITTED
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### 不可重复读（Non-Repeatable Read）

一边读 + 另一边改

指的是在**同一个事务**范围内，前后两次读取到的数据不一样。举个例子：A 事务第 1 次读取了 price 为 10。随后 B 事务将 price 更新为 20，接着 A 事务再次读取 price 为 30。A 事务前后两次读取到的数据是不一样的，这就是不可重复读。

事务 1 的两次查询之间，夹杂了事务 2 的**针对本行的修改**，导致两次查询的结果不一样：

<img src="./assets/1633379-20230924111051437-903161221.png" alt="img" style="zoom: 67%;" />

### 可重复读 （Repeatable Read）

**指的是同一事务范围内读取到的数据是一致的。** 这个隔离级别解决了「不可重复读」的问题，只要是在同一事务范围内，那么读取到的数据就是一样的。对于 MySQL Innodb 来说，其实通过 MVCC 来实现的。但「可重复读」隔离级别会产生幻读问题。

```mysql
-- 设置事务隔离级别为 REPEATABLE READ
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

### 幻读（Phantom Read）

一边读范围 + 另一边改范围

对于某个**范围**的数据读取，前后两次可能读取到不同的结果。

举个例子：数据库中有 price 为 1、3、5 三个商品，此时 A 事务查询 price < 10 的商品，查询到了 3 个商品。随后 B 事务插入了一条 price 为 7 的商品。接着 A 事务继续查询 price < 10 的商品，这次却查询到了 4 个商品。

可以看到「幻读」与「不可重复读」是有些类似的，只是「不可重复读」更多指的是**某一条**记录，而「幻读」指的则是**某个范围**数据。对于 MySQL Innodb 来说，其通过**行级锁**级别的 Gap Lock 解决了幻读的问题。Next-key Lock，间隙锁 + 记录锁，间隙锁锁定记录行之间的间隙（左开右开区间），记录锁锁定行记录，合二为一（左开右闭区间）锁定整个范围。

事务 1 的两次查询之间，夹杂了事务 2 的**针对其他行的插入**，导致两次查询的结果不一样：

<img src="./assets/1633379-20230924113157563-1287688304.png" alt="img" style="zoom: 67%;" />

### 可串行化（Serializable）

**所有事务串行执行**。

```mysql
-- 设置事务隔离级别为 SERIALIZABLE
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### MVCC

MVCC（Multi-Version Concurrency Control）是一种并发控制机制，用于在数据库系统中处理并发读写操作的一致性问题。

在传统的并发控制机制中，比如锁机制，读操作会对数据加锁，写操作会对数据加排他锁，以防止并发操作导致的数据不一致性。但是锁机制会带来死锁、性能下降等问题。

而 MVCC 机制则采用了更加灵活的方式来处理并发问题。它基于**时间戳**的概念，为每个数据版本都分配了一个唯一的时间戳。当一个事务开始时，它会读取当前的数据版本，并将该版本的时间戳作为自己的”读取时间戳”。在事务执行期间，它只能看到在该时间戳之前已经提交的数据版本。对于写操作，事务会创建一个新的数据版本，并将其时间戳设置为当前时间戳。

这样，不同事务之间的读写操作可以同时进行，不会相互阻塞，提高了并发性能。同时，MVCC 也保证了事务之间的隔离性，避免了不可重复读、幻读等问题。

MVCC 在许多数据库系统中得到了广泛的应用，比如 MySQL 的 InnoDB 存储引擎就采用了 MVCC 机制来处理并发控制。

# 日志

![img](./assets/v2-1cceff1e162d155afd524c30f4fd8448_720w.webp)

**Redo log** 和 **bin log**（binary log）是 MySQL 中两个不同的日志类型，它们的作用和使用场景各不相同。下面将详细对比两者的区别：

### 1. **作用和目的**
- **Redo log（重做日志）**：
  - **作用**：InnoDB 引擎专用，用于保证数据库的**崩溃恢复**。当事务被提交时，InnoDB 会将操作记录写入 redo log，即使数据尚未写入磁盘，崩溃后也可以通过 redo log 恢复数据。
  - **目的**：在 MySQL 崩溃或宕机时，利用 redo log 将数据恢复到一致的状态。
  
- **Bin log（二进制日志）**：
  - **作用**：MySQL Server 级别的日志，记录所有**修改数据库内容**的操作，包括 DML（`INSERT`、`UPDATE`、`DELETE` 等）和 DDL（`CREATE`、`ALTER` 等）。它用于**数据恢复**和**主从复制**。
  - **目的**：
    - **数据恢复**：可以使用 bin log 重做某些操作，恢复数据到某个时间点（基于备份 + bin log 的组合）。
    - **主从复制**：用于 MySQL 主从复制中，将主库上的数据更改传递到从库。

### 2. **日志层级**
- **Redo log**：InnoDB 存储引擎级别的日志，**仅适用于 InnoDB 引擎**。其他存储引擎（如 MyISAM）不使用 redo log。
- **Bin log**：MySQL Server 级别的日志，**适用于所有存储引擎**（如 InnoDB、MyISAM 等），记录的是 SQL 语句或基于行的更改。

### 3. **记录内容**
- **Redo log**：
  - 记录的是**物理日志**，具体是数据页的更改（即对磁盘页的修改），而不是 SQL 语句。
  - 日志量较小，记录的数据针对具体的物理操作，主要用于事务提交后的崩溃恢复。

- **Bin log**：
  - 记录的是**逻辑日志**，可以是基于 SQL 语句的日志（statement-based bin log，SBR）或基于行的日志（row-based bin log，RBR）。
  - 适合用于数据的逻辑恢复或数据的复制。

### 4. **写入时机**
- **Redo log**：
  - 在事务执行的过程中，数据变更会先写入 redo log。即使事务尚未提交，数据修改也可能会提前写入 redo log。事务提交时，再刷入 redo log 文件。
  - 因此，redo log 的写入是**提前**的，确保数据的安全性。

- **Bin log**：
  - 只有当事务提交后，MySQL 才会将事务的操作记录写入 bin log。因此，bin log 的写入是在**事务提交后**。

### 5. **数据恢复机制**
- **Redo log**：
  - 用于**崩溃恢复**。当数据库异常宕机后，InnoDB 会通过 redo log 恢复未完全写入磁盘的数据，将数据库恢复到一致性状态。
  - Redo log 只能将数据恢复到最近一次**提交**的状态，不能用于恢复到某个时间点。

- **Bin log**：
  - 用于**时间点恢复**或基于日志的恢复。通过备份数据和 bin log，可以将数据库恢复到某个特定的时间点。
  - 可用于**增量备份**，结合全备份和 bin log，可以恢复到崩溃前的某个时间点。

### 6. **日志文件的大小和循环使用**
- **Redo log**：
  - Redo log 文件是**固定大小**的，由 `innodb_log_file_size` 和 `innodb_log_files_in_group` 参数决定。
  - Redo log 是循环使用的，即写满后会从头开始覆盖旧的日志（类似环形缓冲区）。因此，它主要用于短期的崩溃恢复。

- **Bin log**：
  - Bin log 的大小是可配置的（通过 `max_binlog_size` 参数控制），可以不断生成新的 bin log 文件。
  - Bin log **不会覆盖**旧日志，旧的 bin log 文件会一直保留，直到手动清理或设置自动清理机制（如 `expire_logs_days` 设置 bin log 过期时间）。

### 7. **使用场景**
- **Redo log**：
  - 用于**事务提交后的数据恢复**，在数据库崩溃后恢复到一致的状态，确保事务的持久性。
  - 主要是为了保证数据库的安全性和完整性，不用于数据备份或复制。

- **Bin log**：
  - 用于**主从复制**，通过 bin log 将主库上的操作同步到从库。
  - 用于**数据恢复**，结合全量备份和 bin log，可以恢复数据库到某个特定时间点，适合做增量备份。

### 8. **总结**
| 特性         | Redo Log                       | Bin Log                 |
| ------------ | ------------------------------ | ----------------------- |
| 作用         | 崩溃恢复                       | 数据恢复、主从复制      |
| 日志类型     | 物理日志，记录数据页的变化     | 逻辑日志，记录 SQL 操作 |
| 写入时机     | 事务执行过程中（提前写入）     | 事务提交后写入          |
| 覆盖机制     | 循环覆盖                       | 不覆盖，生成新文件      |
| 应用场景     | 数据库崩溃恢复                 | 数据恢复、主从复制      |
| 使用层级     | InnoDB 引擎级别                | MySQL Server 级别       |
| 文件大小     | 固定大小，循环使用             | 动态增长，不循环        |
| 数据恢复能力 | 仅恢复到最近一次事务提交的状态 | 恢复到任意时间点        |
| 存储引擎     | 仅适用于 InnoDB                | 适用于所有存储引擎      |

*有 binlog 为什么还要 redo log ？*

1. binlog 不知道数据库究竟是在**哪一时刻**丢失了哪部分数据，只能从备份点开始对 binlog 记录重放来恢复数据，比较**耗时**。
2. binlog 恢复是需要我们**手动执行**的，而 redo log 可以在服务器重启后**自动恢复**数据。
3. WAL + 先写缓冲 + 异步刷脏页有效提升了磁盘的 IO 效率。

*有 redo log 为什么还要 binlog？*

1. binlog 是**服务器层面**的功能，redo log 是 **innoDB** 的功能。redo log 帮助 InnoDB 实现了性能提升、自动恢复。但其他存储引擎是**无法使用** redo log 的能力的。
2. 我们也可以关闭 binlog，但大多数情况下我们都会开启，因为开启的好处更多。比如，主从模式需要订阅 binlog 进行**主从复制**，以及可以通过 binlog 进行数据库的增量备份和恢复。

### 总结

**Redo log** 主要用于**崩溃恢复**，是 InnoDB 引擎保证事务持久性的关键机制，而 **bin log** 是 MySQL 层面的日志，主要用于**主从复制**和**数据恢复**。两者在工作原理、写入时机和使用场景上都有明显不同，redo log 专注于保证数据一致性和崩溃恢复，而 bin log 则为数据的备份和复制提供支持。

# 引擎

MySQL的存储引擎是用来存储和管理数据的组件，不同的存储引擎提供了不同的存储机制、索引技巧、锁定水平等功能。MySQL最常见的引擎主要有以下几种：

1. InnoDB：这是 MySQL 的默认存储引擎，支持事务处理和行级锁定，提供了提交、回滚、崩溃恢复能力，支持外键，可以进行外键和非空约束。拥有 redo log 崩溃恢复。
2. MyISAM：这是 MySQL 的传统存储引擎，**不支持事务和行级锁**，只支持表级锁。MyISAM 的优点是插入数据速度快，占用的磁盘空间相对较小。但是，由于不支持事务，安全性不如 InnoDB，一般用于只读或者小型应用。
3. MEMORY：所有的数据都在内存中，数据的处理速度快，但是安全性不高，如果数据库重启，所有的数据都会消失。一般用于存储临时数据。
4. Archive：只支持 INSERT 和 SELECT 操作，适合存储和检索大量的历史数据。
5. BLACKHOLE：黑洞引擎，它不存储数据，插入的数据会被丢弃，但是可以被用在复制的场景，如主从复制。
6. Federated：联邦存储引擎，可以把一些远程的数据表映射为本地的一张表，使用这张表时实际上访问的是远程的数据。

# 主从数据库

<img src="./assets/640.webp" alt="图片" style="zoom:50%;" />

## 设置半同步模式：

1. 加载 lib，所有主从节点都要配置

　　主库&从库（windows 下是 dll 文件）：

```
install plugin rpl_semi_sync_master soname 'semisync_master.so';
install plugin rpl_semi_sync_slave soname 'semisync_slave.so';
```

2. 查看，确保所有节点都成功加载：

    ```
    mysql> show plugins;
    +---------------------------------+----------+--------------------+--------------------+---------+
    | Name                            | Status   | Type               | Library            | License |
    +---------------------------------+----------+--------------------+--------------------+---------+
    | rpl_semi_sync_master            | ACTIVE   | REPLICATION        | semisync_master.so | GPL     |
    +---------------------------------+----------+--------------------+--------------------+---------+
    | rpl_semi_sync_slave             | ACTIVE   | REPLICATION        | semisync_slave.so  | GPL     |
    +---------------------------------+----------+--------------------+--------------------+---------+
    ```

3. 启用半同步：

   1. 先启用从库上的参数，最后启用主库的参数：

      从库：

        ```
        set global rpl_semi_sync_slave_enabled = 1; # 1：启用，0：禁止
        ```
      主库：

        ```
        set global rpl_semi_sync_master_enabled = 1;     # 1：启用，0：禁止
        set global rpl_semi_sync_master_timeout = 60000; # 60秒，时间长些便于实验
        ```

   2. 从库重启 io_thread：

        ```
        stop slave io_thread;
        start slave io_thread;
        ```
        
   3. 查看主库参数：
   
        ```mysql
        show global variables like "%sync%";
        show global status like "%sync%";
        ```



`rpl_semi_sync_master_wait_for_slave_count`

1. master 提交后所需的应答数量，如果 slave clients 数量大于等于这个值，那么 master 会一路畅行无阻；如果低于这个值，master 可能会在事务提交阶段发生一次超时等待，当等待超过参数`rpl_semi_sync_master_timeout`设定时，master 就转为异步模式（原理见下一个参数）。
2. master 将这个参数值作为标杆，用来和`rpl_semi_sync_master_clients`参数做比较。

`rpl_semi_sync_master_wait_no_slave`

1. 为 OFF 时，只要 master 发现`rpl_semi_sync_master_clients`小于`rpl_semi_sync_master_wait_for_slave_count`，则master 立即转为异步模式。
2. 为 ON 时，空闲时间（无事务提交）里，即使 master 发现`rpl_semi_sync_master_clients`小于`rpl_semi_sync_master_wait_for_slave_count`，**也不会做任何调整**。只要保证在事务超时之前，master 收到大于等于`rpl_semi_sync_master_wait_for_slave_count`值的 ACK 应答数量，master 就一直保持在半同步模式；如果在事务提交阶段（master 等待 ACK）超时，master 才会转为异步模式。

无论`rpl_semi_sync_master_wait_no_slave`为 ON 还是 OFF，当 slave 上线到`rpl_semi_sync_master_wait_for_slave_count`值时，master 都会自动由异步模式转为半同步模式。

# 分库与分表

InnoDB 默认每页大小`innodb_page_size`为 16KB，其中页头页尾约占 200B，可供存储的部分约为 16184B。三层 B+ 树数据量计算：
$$
({16184 \over \text{主键字段大小}}) ^ 2 * {16184 \over \text{每行数据大小}}
$$
超过该数据量时建议分表。	

# 优化

1. 禁止使用`SELECT *`：不走覆盖索引，会产生回表，除非预估全表效率更高；

2. 小表驱动大表：`SELECT ... FROM '小表' JOIN '大表'`，因为需要先查出前者中的全部数据，所以前者应为小表、索引完备的表。可以通过`FORCE INDEX`和`STRAIGHT_JOIN`强制指定。

   > `STRAIGHT_JOIN`只适用于内连接，因为`left join`、`right join`已经知道了哪个表作为驱动表，哪个表作为被驱动表，比如`left join`就是以左表为驱动表，`right join`反之，而`STRAIGHT_JOIN`就是在内连接中使用，而强制使用左表来当驱动表，所以这个特性可以用于一些调优，强制改变 mysql 的优化器选择的执行计划。

   其底层为 Join Buffer 缓冲区，其默认大小`innodb_buffer_pool_size`为 0x800000 = 8388608：

   > `innodb_buffer_pool_size`
   >
   > - 定义：这个参数定义了 InnoDB 用于缓存数据和索引的总内存大小。它是 InnoDB 的主要内存结构，决定了可以缓存的数据量，从而影响数据库的性能。
   > - 单位：以字节为单位，通常设置为系统内存的 70% ~ 80%（对于数据库服务器）。
   >
   > `innodb_buffer_pool_chunk_size`
   >
   > - 定义：这个参数定义了缓冲池的分块大小。在 `innodb_buffer_pool_size` 被设置为大于 1GB 的值时，可以将缓冲池分成多个块，每个块的大小由 `innodb_buffer_pool_chunk_size` 定义。
   > - 单位：以字节为单位，默认值通常是 128MB。

3. 连接查询代替子查询；

4. 提升`GROUP BY`的效率：被分组的列建立索引； 

5. 批量插入：单条插入每条都要连接数据库的 connection，SQL——`VALUES`，Mybatis——`<foreache>`，`ExecutorType.BATCH`。默认最大单次插入条数`max_allowed_packet`为 0x400000 = 4194304；

6. 使用`LIMIT`：查询时，最后几页不需要，避免深度分页；

7. 深度分页优化，用条件查询代替分页查询偏移，或者用子查询先查出主键：

   - 记录上次查询最大的 id，并以此为起点进行下一次查询。这种方法需要有连续的、唯一的列（如自增 ID）以用于分页：

      ```mysql
      SELECT * FROM table WHERE id > last_id ORDER BY id ASC LIMIT page_size;
      ```

   - 通过子查询的方式，仅选择需要记录的 id，然后再通过这些 id 获取完整记录：

      ```mysql
      SELECT a.* FROM table AS t
      JOIN (
          SELECT id FROM table ORDER BY id ASC LIMIT page_size OFFSET pass
      ) AS b WHERE a.id = b.id;
      ```

8. `UNION ALL`代替`UNION`：后者会去重，非要用则考虑提升查询本身的效率；

9. 尽量少关联表：

   > 阿里巴巴：不加索引的字段表连接不允许超过 3 张

   - 反范式化；
   - 数据冗余：存储公共字段；
   - 分而治之：复杂查询拆分成多个简单查询，在应用层组合数据；
   - 预先计算：通过 ETL 作业预先计算和存储结果；
   - 使用 NoSQL 数据库：在诸如 ES 等数据库中建立宽表，关联查询时通过字段获取主键，再回到主数据库中通过主键获取最终数据结果。

10. 除小型、明确、简单数据库外，不推荐使用外键：

   - 影响插入和删除的性能；
   - 影响数据迁移和重构；
   - 事务管理变得复杂；
   - 难以应用于分布式数据库；
   - 尽量在应用层通过代码维护；
   - 高并发时锁的争抢。

# Spring 中的数据源动态切换

通过继承 `AbstractRoutingDataSource` 实现数据源动态切换是一个常见的做法。

本质上是通过在上下文的线程中，在操作数据库前切换不同的数据源中选出对应的数据源，往往通过切面配合方法注解的方式。

以下是详细的实现步骤：

### 1. **创建 DynamicDataSource 类**

首先，你需要创建一个继承自 `AbstractRoutingDataSource` 的类，用于决定当前使用哪个数据源。

```java
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

public class DynamicDataSource extends AbstractRoutingDataSource {
    
    @Override
    protected Object determineCurrentLookupKey() {
        // 返回当前数据源的标识
        return DynamicDataSourceContextHolder.getDataSourceType();
    }
}
```

### 2. **创建上下文持有者**

使用一个上下文持有者类来存储和获取当前线程的数据源类型。

```java
public class DynamicDataSourceContextHolder {
    private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();

    public static void setDataSourceType(String dataSourceType) {
        contextHolder.set(dataSourceType);
    }

    public static String getDataSourceType() {
        return contextHolder.get();
    }

    public static void clearDataSourceType() {
        contextHolder.remove();
    }
}
```

### 3. **配置数据源**

在 Spring 的配置类中，配置多个数据源并使用 `DynamicDataSource`。

```java
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class DataSourceConfig {

    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.slave")
    public DataSource slaveDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    public DynamicDataSource dynamicDataSource(
            @Qualifier("masterDataSource") DataSource master,
            @Qualifier("slaveDataSource") DataSource slave) {

        Map<Object, Object> targetDataSources = new HashMap<>();
        targetDataSources.put("master", master);
        targetDataSources.put("slave", slave);

        DynamicDataSource dynamicDataSource = new DynamicDataSource();
        dynamicDataSource.setTargetDataSources(targetDataSources);
        dynamicDataSource.setDefaultTargetDataSource(master); // 默认使用主库

        return dynamicDataSource;
    }
}
```

### 4. **使用 AOP 切换数据源**

可以使用 AOP 来在方法调用前后设置和清除数据源类型。

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.After;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class DataSourceAspect {

    @Before("@annotation(ReadOnly)")
    public void setReadDataSource() {
        DynamicDataSourceContextHolder.setDataSourceType("slave");
    }

    @Before("@annotation(WriteOnly)")
    public void setWriteDataSource() {
        DynamicDataSourceContextHolder.setDataSourceType("master");
    }

    /**
     * 织出时清空 ThreadLocal，避免数据库事务传播行为而影响主从切换错误
     */
    @After("@annotation(ReadOnly) || @annotation(WriteOnly)")
    public void clearDataSource() {
        DynamicDataSourceContextHolder.clearDataSourceType();
    }
}
```

### 5. **定义注解**

定义自定义注解来标识读写操作，也可以换成传参获取数据源名。

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ReadOnly {}

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface WriteOnly {}
```

### 6. **使用注解**

在服务层中使用自定义注解来标识读写方法。

```java
public class UserService {

    @ReadOnly
    public User getUserById(Long id) {
        // 查询操作，使用从库
    }

    @WriteOnly
    public void createUser(User user) {
        // 写入操作，使用主库
    }
}
```

### 总结

通过继承 `AbstractRoutingDataSource` 和结合 AOP，可以实现动态数据源切换，从而实现读写分离。这样在运行时，根据方法的注解来决定使用哪个数据源，提高了代码的灵活性和可维护性。