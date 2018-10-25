# pgRouting RFC 3: Positional and named parameters on pgRouting functions

| Item | value
| --- | ---
| RFC 3 | Handling releases and branches on pgRouting
| Author(s) | Vicky Vergara
| Status | Adopted
| Date | October 24, 2018

Table of Contents
=================

* [pgRouting RFC 3: Positional and named parameters on pgRouting functions](#pgrouting-rfc-3-positional-and-named-parameters-on-pgrouting-functions)
* [Summary](#summary)
* [Details](#details)
   * [PostgreSQL](#postgresql)
* [pgRouting](#pgrouting)
   * [Proposal:](#proposal)
   * [With this proposal, from the user point of view](#with-this-proposal-from-the-user-point-of-view)
      * [These calls are valid:](#these-calls-are-valid)
      * [These calls are invalid:](#these-calls-are-invalid)
   * [Developers steps to take:](#developers-steps-to-take)


# Summary

This document proposes how to proceed on:

* Handling parameters names on functions.

Enforcing from: pgRouting version 3.0

# Details
## PostgreSQL
PostgreSQL provides:
- Positional notation
  - a function call is written with its argument values in the same order as they are defined in the function declaration
- Named notation
  - the arguments are matched to the function parameters by name and can be written in any order
- Mixed notation
  - which combines positional and named notation.
  - The positional parameters are written first and named parameters appear after them.

Starting from PostgreSQL v9.4 the named notation is with `=>` instead of `:=` and
PostgreSQL v9.3 end of life is on November 8, 2018


**references**
* https://www.postgresql.org/docs/current/static/sql-syntax-calling-funcs.html
* https://www.postgresql.org/support/versioning

# pgRouting

The majority of pgRouting functions have the following structure:
```
pgr_<function_name>(<edges_sql>, [<other_sql>], start_vid(s), end_vid(s), <optional_parameters>)
```
also most of our functions have optional parameters

## Proposal:

- Use positional parameters on the compulsory parameters
- Use named parameters on the optional parameters

## With this proposal, from the user point of view
For this signatures
```
pgr_aStar(edges_sql, from_vid,  to_vid  [, directed] [, heuristic] [, factor] [, epsilon])
pgr_aStar(edges_sql, from_vid,  to_vids [, directed] [, heuristic] [, factor] [, epsilon])
```
Where the defaults of the optional parameters are:
```
directed => true
heuristic => 5
factor => 1
epsilon => 1
```

### These calls are valid:

**Example 1** The optional parameters have their default value

Automatically its the "One to One" signature
```
SELECT * FROM pgr_aStar(
    'SELECT id, source, target, cost, reverse_cost, x1, y1, x2, y2 FROM edge_table',
    2, 12);
```
**Example 2** The optional parameter `directed` can be used with positional notation

Automatically its the "One to Many" signature
```
SELECT * FROM pgr_aStar(
    'SELECT id, source, target, cost, reverse_cost, x1, y1, x2, y2 FROM edge_table',
    2, ARRAY[3, 12], true, heuristic => 2);
```

**Example 3** The optional parameter `directed` of position 4 is skipped using the default value `true`
```
SELECT * FROM pgr_aStar(
    'SELECT id, source, target, cost, reverse_cost, x1, y1, x2, y2 FROM edge_table',
    2, ARRAY[3, 12], heuristic => 2);
```
**Example 4** The optional parameters `directed` and `heuristic` changed position
```
SELECT * FROM pgr_aStar(
    'SELECT id, source, target, cost, reverse_cost, x1, y1, x2, y2 FROM edge_table',
    2, ARRAY[3, 12], heuristic => 2, directed => false);
```
### These calls are invalid:
**Example 5** Mix compulsory parameters with optional parameters

Compare with example 4:
```
SELECT * FROM pgr_aStar(
    directed => false, to_vids => ARRAY[3, 12], heuristic => 2,
    edges_sql => 'SELECT id, source, target, cost, reverse_cost, x1, y1, x2, y2 FROM edge_table',
    from_vid => 2);
```

**Example 6** Over use of named parameters

Compare with example 1
```
SELECT * FROM pgr_aStar(
    edges_sql => 'SELECT id, source, target, cost, reverse_cost, x1, y1, x2, y2 FROM edge_table',
    from_vid => 2, to_vids => ARRAY[3, 12]);
```
**Example 7** Difficult to debug
```
SELECT * FROM pgr_aStar(
    edges_sql => 'SELECT id, source, target, cost, reverse_cost, x1, y1, x2, y2 FROM edge_table',
    from_vid => 2, to_vid => ARRAY[3, 12], directed => true, heuristic => 2);
```
Error: The user is has a contradiction:
`from_vid, to_vid` combination is for the "One to One" signature But is using ARRAY that belongs to the "One to Many" signature.



## Developers steps to take:

Taking as example:
```
CREATE OR REPLACE FUNCTION pgr_aStarCost(
    edges_sql TEXT,
    start_vid BIGINT,
    end_vid BIGINT,
    directed BOOLEAN DEFAULT true,
    heuristic INTEGER DEFAULT 5,
    factor FLOAT DEFAULT 1.0,
    epsilon FLOAT DEFAULT 1.0,

    OUT start_vid BIGINT,
    OUT end_vid BIGINT,
    OUT agg_cost FLOAT)

RETURNS SETOF RECORD AS
$BODY$
    SELECT a.start_vid, a.end_vid, a.agg_cost
    FROM _pgr_aStar(_pgr_get_statement($1), ARRAY[$2]::BIGINT[],  ARRAY[$3]::BIGINT[], $4, $5, $6::FLOAT, $7::FLOAT, true) AS a
    ORDER BY  a.start_vid, a.end_vid;
$BODY$
LANGUAGE sql VOLATILE STRICT
COST 100
ROWS 1000;
```
**Note:** Internally its not using names, because of name collision of the input parameters with the output parameters.

```
CREATE OR REPLACE FUNCTION pgr_aStarCost(
    ---------------- remove the names on compulsory parameters
    TEXT,
    BIGINT,
    BIGINT,
    directed BOOLEAN DEFAULT true,
    heuristic INTEGER DEFAULT 5,
    factor FLOAT DEFAULT 1.0,
    epsilon FLOAT DEFAULT 1.0,

    OUT start_vid BIGINT,
    OUT end_vid BIGINT,
    OUT agg_cost FLOAT)

RETURNS SETOF RECORD AS
$BODY$
    ---------------- Because name collision is not there any more remve the  `a.` and the `AS a`
    SELECT start_vid, end_vid, agg_cost
    FROM _pgr_aStar(_pgr_get_statement($1), ARRAY[$2]::BIGINT[],  ARRAY[$3]::BIGINT[], $4, $5, $6::FLOAT, $7::FLOAT, true)
    ORDER BY start_vid, end_vid;
$BODY$
LANGUAGE sql VOLATILE STRICT
COST 100
ROWS 1000;
```








