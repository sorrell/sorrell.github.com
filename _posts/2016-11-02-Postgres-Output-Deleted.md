---
layout: post
title: Postgres Output Deleted
tags: [SSIS, BIML]
---
# {{ page.title }}

Working on a live, production database is sometimes unavoidable.  The amount of nervousness spent on an UPDATE or DELETE in a production environment, *even if you've already tested the change in a testing environement* is agonizing.

A while back I found out how to prevent this agony when using SQL Server.  You would simply wrap your `UPDATE` in a `BEGIN/ROLLBACK`, and use `OUTPUT DELETED.*, INSERTED.*` to view the changes you would have made.  This is a very safe way to see the effects without committing the changes.  It might look something like:

```
  BEGIN TRAN;
  UPDATE myTable
  SET myCol = 4
  OUTPUT deleted.*, inserted.*
  WHERE myCol = 5;
  ROLLBACK;
```

The output breaks the UPDATE into two steps, first deleting the old row, and inserting the new row.  Remember, we are rolling back so the change isn't committed, but the output allows us to see first the deleted columns, and then the inserted columns.  And the same holds true for a deletion:

```
  BEGIN TRAN;
  DELETED myTable
  OUTPUT deleted.*, inserted.*
  WHERE myCol = 5;
  ROLLBACK;
```

When I started working with PostgreSQL, I was itching for an equivalent.  In the above queries, you can run the the entire query in a single command and see the results thanks to SQL Server Mgmt Studio.  Unfortunately, PGAdmin is no Mgmt Studio, so we have to hack around a little and **separate into two commands**.  First we execute the preview by opening a transaction, and then our second step is to rollback that transaction after previewing our results.  Here's how that might look:

```
  BEGIN;
  UPDATE myTable
  SET myCol = 4
  FROM myTable as old_myTable
  WHERE myCol = 5
  RETURNING old_myTable.*, myTable.*;

  -- Ok, take a breath and review your results before you run this next command!
  ROLLBACK;
```

Thankfully, the `DELETE` preview is a little more compact:

```
  BEGIN;
  DELETE FROM myTable
  WHERE myCol >= 5
  AND myCol <= 10
  RETURNING myTable.*

  -- Ok, take that breather again!  Don't run me until you're ready!
  ROLLBACK;
```

The best part about these queries is that when you feel comfortable with them, just change the `ROLLBACK` to a `COMMIT` and it's all done!