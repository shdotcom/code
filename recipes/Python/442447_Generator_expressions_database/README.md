## Generator expressions for database requests

Originally published: 2005-10-18 12:21:33
Last updated: 2005-10-18 12:21:33
Author: Pierre Quentel

This recipe is a follow-up to #440653, which was easy to implement but very slow because the iteration required to read all the rows of a table\n\nAs suggested by Matteo Dell'Amico in a comment, it would be much better if we could write something like\n\n\tquery(r.name for r in plane_tbl if r.country == "France")\n\nwhere the generator expression is first translated into an SQL select, so that the iteration on the instance of query only reads the rows selected by the SQL statement\n\nThe present recipe is an attempt to achieve this. The first problem is to get the source code of the generator expression. I use information from the stack frame to get the file name and the line number, then the tokenize module to read the elements of the generator expression in the source code\n\nThen, to build the SQL statement, the source code must be parsed : this is done using the compiler package and "visitors" that walk the AST tree returned by compiler.parse and do operations on the nodes, depending on their type\n\nFinally, once the SQL statement is built, the iteration on the query instance can start : for the first one, the SQL statement is executed ; then the iteration yields the selected rows one by one.\n\nThe items can be :\n- objects, with attribute names matching those in the generator expression, except that qualified names (table.name) are converted to table_name\n- dictionaries : the keys are the same as the attribute names above\n- lists\n\nFor instance :\n- iterating on query(name for r in plane_tbl) returns objects with an attribute name\n- iterating on query(r.name for r in plane_tbl) returns objects with an attribute r_name\n\nThis is because of iteration on tables which have the same field names\n\nquery((r.name,c.name) for r in plane_tbl for c in country_tbl if r.speed > 500 )\n\nThe type of the items is set by query.return_type = object, dict or list