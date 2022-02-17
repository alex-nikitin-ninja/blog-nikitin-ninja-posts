# Simple Queue Service Analog in MySQL

Difficulty level: **medium**

SQS is a service which allows developers to create queues, put messages in them,
and depends on several conditions retrieve them.

One of the use case is to send different tasks between various worker instances

## Objectives

Create a service where we can create described earlier types of messages.

Be able to retrieve messages, see if there's any tables locks and not able to
retrieve messages more that twice within given period of time.

Use MySQL compatible database (Maria DB, AWS Aurora, MemSQL, etc) for simplicity
of implementation, wide support, etc.


For messages we need to be able to make these 3 operations:

- put message in queue
- retrieve latest message
- delete/soft delete message from queue

In this exercise we'll learn how to create tables and stored procedures.

Interested...?

-more-

## Input details

Database/language: **MySQL**/**SQL**

## Implementation

Let's assume we use a database name `main_schema`. We need to create a primary
table, then related stored procedures.

### Our primary table creation
```
DROP TABLE main_schema.main_queue;
CREATE TABLE main_schema.main_queue (
    id BIGINT NOT NULL AUTO_INCREMENT,
    queue_name VARCHAR(64) DEFAULT 'default',
    payload MEDIUMTEXT,
    read_cnt BIGINT DEFAULT 0,
    max_read_cnt BIGINT DEFAULT 1,
    last_read DATETIME(6) DEFAULT CURRENT_TIMESTAMP(6),
    next_read DATETIME(6) DEFAULT CURRENT_TIMESTAMP(6),
    invisibility_time BIGINT DEFAULT 0,
    created_at DATETIME(6) DEFAULT CURRENT_TIMESTAMP(6),
    deleted_at DATETIME(6) DEFAULT NULL,
    PRIMARY KEY(id)
);
```

**Columns explanation**:

- `id` - our primary key in the table.
- `queue_name` - different queues can handle payload for different tasks.
- `payload` - our message to be delivered.
- `read_cnt` - this counter indicates how many times message been read already.
- `max_read_cnt` - limit how many times the message can be delivered. `0` means
no limit on how many times it can be read.
- `last_read` - mark last time message being read.
- `next_read` - timestamp when message is visible for reading.
- `invisibility_time` - number in seconds how long message is not available for
retrieval (helpful if our task needs time to process the message before actual
deletion. `0` means visible immediately.
- `created_at` - timestamp when message was created.

**Note:** number in `DATETIME(6)` means fraction part of seconds to store.

**Indexes** - TBD, but according to the code in the stored procedures, that need
to be the following columns `next_read`, `max_read_cnt`, `read_cnt`,
`queue_name`.

### Stored functions - `put_message_in_queue`
```
USE main_schema;
DROP FUNCTION put_message_in_queue;
DELIMITER //
CREATE FUNCTION put_message_in_queue (_queue_name VARCHAR(64), _payload MEDIUMTEXT, _max_read_cnt BIGINT, _invisibility_time BIGINT)
RETURNS BIGINT
DETERMINISTIC
BEGIN
    SET @queue_name = _queue_name;
    SET @payload = _payload;
    SET @max_read_cnt = _max_read_cnt;
    SET @invisibility_time = _invisibility_time;

    INSERT INTO main_schema.main_queue
        (queue_name, payload, max_read_cnt, invisibility_time)
    VALUES
        (@queue_name, @payload, @max_read_cnt, @invisibility_time);

    SET @LID = LAST_INSERT_ID();

    RETURN @LID;
END //
DELIMITER ;
-- SELECT put_message_in_queue ('default', '{}', 1, 0) AS _message_id;
```

### Stored procedures - `get_message_from_queue`
```
USE main_schema;
DROP PROCEDURE get_message_from_queue;
DELIMITER //
CREATE PROCEDURE get_message_from_queue (IN _queue_name VARCHAR(64))
BEGIN
    START TRANSACTION;
    
    SET @queue_name = _queue_name;
    SET @_now = NOW(6);
    SET @id = (
        SELECT
            id
        FROM
            main_schema.main_queue
        WHERE
            next_read < @_now
            AND (max_read_cnt = 0 OR (max_read_cnt != 0 AND max_read_cnt > read_cnt))
            AND queue_name=@queue_name
            AND deleted_at IS NULL
        ORDER BY id
        LIMIT 1
        FOR UPDATE);
    
    UPDATE
        main_schema.main_queue
    SET
        last_read = @_now,
        read_cnt = read_cnt + 1,
        next_read = DATE_ADD(@_now, INTERVAL invisibility_time SECOND)
    WHERE id=@id;
    
    SELECT * FROM main_schema.main_queue WHERE id = @id FOR UPDATE;
    
    COMMIT;
END //
DELIMITER ;
-- CALL main_schema.get_message_from_queue('default');
```

### Stored procedures - `delete_message_from_queue`
```
USE main_schema;
DROP PROCEDURE delete_message_from_queue;
DELIMITER //
CREATE PROCEDURE delete_message_from_queue (IN _message_id BIGINT)
BEGIN
    START TRANSACTION;
    
    SET @_id = _message_id;
    SET @_now = NOW(6);
    
    UPDATE
        main_schema.main_queue
    SET
        deleted_at = @_now
    WHERE id=@_id AND deleted_at IS NULL;

    COMMIT;
END //
DELIMITER ;
-- CALL main_schema.delete_message_from_queue(1);
```

## Usage

Basic data flow would look like described below:

Step 1 - put message in queue with certain parameters.  
Step 2 - certain processing system gets message from there.  
Step 3 - with given details (id) from Step 2 message gets deleted by the
processing system.

## Example

Assuming we're familiar with docker.

### Start the MySQL server

```
$ docker run --rm \
    --name main_schema_mysql \
    -p 33061:3306 \
    -e MYSQL_ALLOW_EMPTY_PASSWORD='yes' \
    -e MYSQL_ROOT_PASSWORD='' \
    -e MYSQL_DATABASE='main_schema' \
    -d mysql:8.0
```

Connect to the server with any appropriate way, for example using `mysql-cli`
client:
```
$ mysql --port 33061 -u root -h 127.0.0.1
```

Then run all the sql scripts from above - create table, stored function, and two
stored procedures.

### And let's try it in action

I'm going to use the following pattern - create a simple message with 2 max read
count, then retrieve the message two times and validate that it's not readable
anymore, then soft delete the message and validate that necessary metadata value
was updated in the table directly.

Switch the DB for current session:
```
USE main_schema;
```

We put the message in queue, and receive message id for later use or any kind of
logging:
```
SELECT put_message_in_queue ('default', '{}', 2, 0) AS _message_id;
```

In this call we output message and note that `read_cnt` counter increases its
value:
```
CALL main_schema.get_message_from_queue('default');
CALL main_schema.get_message_from_queue('default');
```

In this call we ensure that there's no messages to report:
```
CALL main_schema.get_message_from_queue('default');
```

And finally delete the message (though it's not necessary in this scenario
because message will not appear anymore because of counter is over limit):
```
CALL main_schema.delete_message_from_queue(1);
```

Then with direct table call we ensure that deleted_at column gets updated
accordingly:
```
SELECT * FROM main_schema.main_queue;
```

**Output:**
```
mysql> USE main_schema;
Database changed

mysql> SELECT put_message_in_queue ('default', '{}', 2, 0) AS _message_id;
+-------------+
| _message_id |
+-------------+
|           1 |
+-------------+
1 row in set (0.01 sec)

mysql> CALL main_schema.get_message_from_queue('default');
+----+------------+---------+----------+--------------+----------------------------+----------------------------+-------------------+----------------------------+------------+
| id | queue_name | payload | read_cnt | max_read_cnt | last_read                  | next_read                  | invisibility_time | created_at                 | deleted_at |
+----+------------+---------+----------+--------------+----------------------------+----------------------------+-------------------+----------------------------+------------+
|  1 | default    | {}      |        1 |            2 | 2022-02-16 20:59:57.003972 | 2022-02-16 20:59:57.003972 |                 0 | 2022-02-16 20:59:50.915183 | NULL       |
+----+------------+---------+----------+--------------+----------------------------+----------------------------+-------------------+----------------------------+------------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

mysql> CALL main_schema.get_message_from_queue('default');
+----+------------+---------+----------+--------------+----------------------------+----------------------------+-------------------+----------------------------+------------+
| id | queue_name | payload | read_cnt | max_read_cnt | last_read                  | next_read                  | invisibility_time | created_at                 | deleted_at |
+----+------------+---------+----------+--------------+----------------------------+----------------------------+-------------------+----------------------------+------------+
|  1 | default    | {}      |        2 |            2 | 2022-02-16 21:00:01.527457 | 2022-02-16 21:00:01.527457 |                 0 | 2022-02-16 20:59:50.915183 | NULL       |
+----+------------+---------+----------+--------------+----------------------------+----------------------------+-------------------+----------------------------+------------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

mysql> CALL main_schema.get_message_from_queue('default');
Empty set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

mysql> CALL main_schema.delete_message_from_queue(1);
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM main_schema.main_queue;
+----+------------+---------+----------+--------------+----------------------------+----------------------------+-------------------+----------------------------+----------------------------+
| id | queue_name | payload | read_cnt | max_read_cnt | last_read                  | next_read                  | invisibility_time | created_at                 | deleted_at                 |
+----+------------+---------+----------+--------------+----------------------------+----------------------------+-------------------+----------------------------+----------------------------+
|  1 | default    | {}      |        2 |            2 | 2022-02-16 21:00:01.527457 | 2022-02-16 21:00:01.527457 |                 0 | 2022-02-16 20:59:50.915183 | 2022-02-16 21:00:09.191801 |
+----+------------+---------+----------+--------------+----------------------------+----------------------------+-------------------+----------------------------+----------------------------+
1 row in set (0.00 sec)

```

## Conclusion

In this exercise we learned how to create stored procedures and got familiar
with messaging queue systems.

When I was working on this tool - most difficult part was to make sure that
Race Conditions event does not happen when several tasks workers query the DB
simultaneously. I was able to achieve that using transactions and proper rows
locks with `SELECT ... FOR UPDATE` statement ([dev.mysql.com/.../innodb-locking-reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html)).

 #sqs #queue #service #research #mysql #howto #databases