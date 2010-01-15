
======================
Akshell Query Language
======================

Grammar
=======

.. productionlist::
   relation: 'for' rangevars relation | union | select
   rangevars: '(' NAME  (',' NAME)* 'in' relation ')'
   union: 'union' '(' relation (',' relation)* ')'
   select: ('{' [prototype (',' prototype)*] '}' | field_expr | NAME)
         : ['where' expression]
   prototype: NAME ':' expr | field_expr | NAME
   expr: ('forsome' | 'forall') (rangevars | '(' NAME (',' NAME)* ')') expr |
       : cond_expr
   cond_expr: log_or_expr ['?' expr ':' cond_expr]
   log_or_expr: log_and_expr ('||' log_and_expr)*
   log_and_expr: eq_expr ('&&' eq_expr)*
   eq_expr: rel_expr (('==' | '!=') rel_expr)*
   rel_expr: add_expr (('<=' | '>=' | '<' | '>') add_expr)*
   add_expr: mul_expr (('+' | '-') mul_expr)*
   mul_expr: unary_expr (('*' | '/' | '%') unary_expr)*
   unary_expr: ['+' | '-' | '!'] prim_expr
   prim_expr: NUMBER | STRING | BOOLEAN |
            : '(' expr ')' | '$' DIGIT* | field_expr | NAME
   field_expr: NAME ('.' NAME | '[' NAME (',' NAME)* ']')
             : ('->' (NAME | '[' NAME (',' NAME)* ']'))*
