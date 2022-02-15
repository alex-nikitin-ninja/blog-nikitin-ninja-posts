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

### Tables creation
```
DROP TABLE main_schema.main_queue;
CREATE TABLE main_schema.main_queue (
    id BIGINT NOT NULL AUTO_INCREMENT,
    queue_name VARCHAR(64) DEFAULT 'default',
    payload MEDIUMTEXT,
    read_cnt BIGINT DEFAULT 0,
    max_read_cnt BIGINT DEFAULT 1,
    last_read DATETIME(6) DEFAULT '2020-01-01 00:00:00',
    next_read DATETIME(6) DEFAULT '2020-01-01 00:00:00',
    invisibility_time BIGINT DEFAULT 0,
    created_at DATETIME(6) DEFAULT CURRENT_TIMESTAMP,
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


**Indexes** - TBD, but according to the code in the stored procedures, that need
to be the following columns `next_read`, `max_read_cnt`, `read_cnt`,
`queue_name`.

### Stored procedures - `put_message_in_queue`
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
-- SELECT put_message_in_queue ('default', '{}', 1, 0);
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
            AND (max_read_cnt = 0 || (max_read_cnt != 0 AND max_read_cnt > read_cnt))
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
-- CALL main_schema.get_message_from_queue('default')
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
-- CALL main_schema.delete_message_from_queue(1)
```

## Usage

Basic data flow would look like described below:

Step 1 - put message in queue with certain parameters.  
Step 2 - certain processing system gets message from there.  
Step 3 - with given details (id) from Step 2 message gets deleted by the
processing system.

## Examples


