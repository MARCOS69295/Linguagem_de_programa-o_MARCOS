CÓDIGO COMENTADO

import ply.lex as lex

# Definição dos tokens que serão reconhecidos pelo lexer
tokens = (
    'NAME', 'NUMBER',
    'PLUS', 'MINUS', 'TIMES', 'DIVIDE', 'EQUALS',
    'LPAREN', 'RPAREN',
)

# Regras de expressão regular para tokens simples
t_PLUS    = r'\+'  # Token para o operador de adição (+)
t_MINUS   = r'-'    # Token para o operador de subtração (-)
t_TIMES   = r'\*'  # Token para o operador de multiplicação (*)
t_DIVIDE  = r'/'    # Token para o operador de divisão (/)
t_EQUALS  = r'='    # Token para o operador de igualdade (=)
t_LPAREN  = r'\('  # Token para o parêntese esquerdo (\()
t_RPAREN  = r'\)'  # Token para o parêntese direito (\))

# Regra para identificar variáveis e nomes
# Esta regra identifica palavras que começam com letras ou underscore e podem ter números
# Exemplo: "x", "var_1", "_nome"
def t_NAME(t):
    r'[a-zA-Z_][a-zA-Z_0-9]*'
    return t

# Regra para identificar números inteiros
def t_NUMBER(t):
    r'\d+'
    t.value = int(t.value)    # Converte o valor do token para inteiro
    return t

# Regra para contar número de linhas
# Atualiza o número de linhas do lexer a cada nova linha encontrada
def t_newline(t):
    r'\n+'
    t.lexer.lineno += len(t.value)

# Define caracteres que devem ser ignorados (espaços e tabulações)
t_ignore  = ' \t'

# Regra para tratar erros de caracteres ilegais
def t_error(t):
    # Imprime o caractere ilegal encontrado e pula para o próximo caractere
    print(f"Caractere ilegal '{t.value[0]}'")
    t.lexer.skip(1)

# Construção do lexer
lexer = lex.lex()

import ply.yacc as yacc

# Definição da precedência dos operadores
precedence = (
    ('left', 'PLUS', 'MINUS'),  # '+' e '-' têm a mesma precedência, associam da esquerda para a direita
    ('left', 'TIMES', 'DIVIDE'), # '*' e '/' têm precedência maior que '+' e '-', associam da esquerda para a direita
    ('nonassoc', 'UMINUS'),      # '-' unário não é associativo
)

# Dicionário para armazenar variáveis e seus valores
names = {}

# Regra para atribuição de variáveis
# Exemplo: x = 3 + 5
def p_statement_assign(p):
    'statement : NAME EQUALS expression'
    # Armazena o valor da expressão na variável especificada
    names[p[1]] = p[3]

# Regra para expressões que são apenas avaliadas e imprimidas
# Exemplo: 3 + 4
def p_statement_expr(p):
    'statement : expression'
    # Imprime o resultado da expressão avaliada
    print(p[1])

# Regras para operações binárias: soma, subtração, multiplicação e divisão
def p_expression_binop(p):
    '''expression : expression PLUS expression
                  | expression MINUS expression
                  | expression TIMES expression
                  | expression DIVIDE expression'''
    # Avalia a operação correspondente com base no operador
    if p[2] == '+':
        p[0] = p[1] + p[3]
    elif p[2] == '-':
        p[0] = p[1] - p[3]
    elif p[2] == '*':
        p[0] = p[1] * p[3]
    elif p[2] == '/':
        if p[3] == 0:
            print("Erro: divisão por zero")
            p[0] = 0
        else:
            p[0] = p[1] / p[3]

# Regra para o operador unário menos (ex: -5)
def p_expression_uminus(p):
    "expression : MINUS expression %prec UMINUS"
    # Aplica o operador unário menos ao valor da expressão
    p[0] = -p[2]

# Regra para expressões entre parênteses
def p_expression_group(p):
    "expression : LPAREN expression RPAREN"
    # Retorna o valor da expressão dentro dos parênteses
    p[0] = p[2]

# Regra para valores numéricos
def p_expression_number(p):
    "expression : NUMBER"
    # Retorna o valor numérico
    p[0] = p[1]

# Regra para valores de variáveis
def p_expression_name(p):
    "expression : NAME"
    # Tenta obter o valor da variável pelo nome
    try:
        p[0] = names[p[1]]
    except LookupError:
        # Se a variável não foi definida, exibe uma mensagem de erro e retorna 0
        print(f"Nome indefinido '{p[1]}'")
        p[0] = 0

# Regra para tratar erros de sintaxe
def p_error(p):
    # Imprime uma mensagem de erro informando o local do erro
    print("Erro de sintaxe em '%s'" % p.value if p else "Erro de sintaxe no final do arquivo")

# Construção do parser
parser = yacc.yacc()

# Teste do lexer com uma expressão de exemplo
data = '''
3 + 4 * 10
  + -20 * 2
'''

# Alimenta o lexer com a entrada de dados
lexer.input(data)

# Tokeniza e imprime cada token encontrado
while True:
    tok = lexer.token()
    if not tok: 
        break      # Sem mais tokens
    # Imprime o token encontrado
    print(tok)

# Teste do parser com a mesma entrada
parser.parse(data)




CÓDIGO NA INTEGRA

import ply.lex as lex
tokens = (
    'NAME', 'NUMBER',
    'PLUS', 'MINUS', 'TIMES', 'DIVIDE', 'EQUALS',
    'LPAREN', 'RPAREN',
)
# Regular expression rules for simple tokens
t_PLUS    = r'\+'
t_MINUS   = r'-'
t_TIMES   = r'\*'
t_DIVIDE  = r'/'
t_EQUALS  = r'='
t_LPAREN  = r'\('
t_RPAREN  = r'\)'
# A regular expression rule with some action code
def t_NAME(t):
    r'[a-zA-Z_][a-zA-Z_0-9]*'
    return t
def t_NUMBER(t):
    r'\d+'
    t.value = int(t.value)    
    return t
# Define a rule so we can track line numbers
def t_newline(t):
    r'\n+'
    t.lexer.lineno += len(t.value)
# A string containing ignored characters (spaces and tabs)
t_ignore  = ' \t'
# Error handling rule
def t_error(t):
    print(f"Illegal character '{t.value[0]}'")
    t.lexer.skip(1)
# Build the lexer
lexer = lex.lex()
import ply.yacc as yacc
# Definição da precedência dos operadores
precedence = (
    ('left','PLUS','MINUS'),
    ('left','TIMES','DIVIDE'),
    ('nonassoc','UMINUS'),
)
# Dicionário de nomes (para armazenar variáveis)
names = {}
def p_statement_assign(p):
    'statement : NAME EQUALS expression'
    names[p[1]] = p[3]
def p_statement_expr(p):
    'statement : expression'
    print(p[1])
def p_expression_binop(p):
    '''expression : expression PLUS expression
                  | expression MINUS expression
                  | expression TIMES expression
                  | expression DIVIDE expression'''
    if p[2] == '+': p[0] = p[1] + p[3]
import ply.lex as lex
tokens = (
    'NAME', 'NUMBER',
    'PLUS', 'MINUS', 'TIMES', 'DIVIDE', 'EQUALS',
    'LPAREN', 'RPAREN',
)
# Regular expression rules for simple tokens
t_PLUS    = r'\+'
t_MINUS   = r'-'
t_TIMES   = r'\*'
t_DIVIDE  = r'/'
t_EQUALS  = r'='
t_LPAREN  = r'\('
t_RPAREN  = r'\)'
# A regular expression rule with some action code
def t_NAME(t):
    r'[a-zA-Z_][a-zA-Z_0-9]*'
    return t
def t_NUMBER(t):
    r'\d+'
    t.value = int(t.value)    
    return t
# Define a rule so we can track line numbers
def t_newline(t):
    r'\n+'
    t.lexer.lineno += len(t.value)
# A string containing ignored characters (spaces and tabs)
t_ignore  = ' \t'
# Error handling rule
def t_error(t):
    print(f"Illegal character '{t.value[0]}'")
    t.lexer.skip(1)
# Build the lexer
lexer = lex.lex()
import ply.yacc as yacc
# Definição da precedência dos operadores
precedence = (
    ('left','PLUS','MINUS'),
    ('left','TIMES','DIVIDE'),
    ('nonassoc','UMINUS'),
)
# Dicionário de nomes (para armazenar variáveis)
names = {}
def p_statement_assign(p):
    'statement : NAME EQUALS expression'
    names[p[1]] = p[3]
def p_statement_expr(p):
    'statement : expression'
    print(p[1])
def p_expression_binop(p):
    '''expression : expression PLUS expression
                  | expression MINUS expression
                  | expression TIMES expression
                  | expression DIVIDE expression'''
    if p[2] == '+': p[0] = p[1] + p[3]
    elif p[2] == '-': p[0] = p[1] - p[3]
    elif p[2] == '*': p[0] = p[1] * p[3]
    elif p[2] == '/': p[0] = p[1] / p[3]
def p_expression_uminus(p):
    "expression : MINUS expression %prec UMINUS"
    p[0] = -p[2]
def p_expression_group(p):
    "expression : LPAREN expression RPAREN"
    p[0] = p[2]
def p_expression_number(p):
    "expression : NUMBER"
    p[0] = p[1]
def p_expression_name(p):
    "expression : NAME"
    try:
        p[0] = names[p[1]]
    except LookupError:
        print(f"Undefined name '{p[1]}'")
        p[0] = 0
def p_error(p):
    print("Syntax error at '%s'" % p.value if p else "Syntax error at EOF")
parser = yacc.yacc()
# Testando o lexer
data = '''
3 + 4 * 10
  + -20 *2
'''
# Give the lexer some input
lexer.input(data)
# Tokenize
while True:
    tok = lexer.token()
    if not tok: 
        break      # No more input
    print(tok)
# Testando o parser
parser.parse(data)

Created by PLY version 3.11 (http://www.dabeaz.com/ply)
Grammar
Rule 0     S' -> statement
Rule 1     statement -> NAME EQUALS expression
Rule 2     statement -> expression
Rule 3     expression -> expression PLUS expression
Rule 4     expression -> expression MINUS expression
Rule 5     expression -> expression TIMES expression
Rule 6     expression -> expression DIVIDE expression
Rule 7     expression -> MINUS expression
Rule 8     expression -> LPAREN expression RPAREN
Rule 9     expression -> NUMBER
Rule 10    expression -> NAME
Terminals, with rules where they appear
DIVIDE               : 6
EQUALS               : 1
LPAREN               : 8
MINUS                : 4 7
NAME                 : 1 10
NUMBER               : 9
PLUS                 : 3
RPAREN               : 8
TIMES                : 5
error                : 
Nonterminals, with rules where they appear
expression           : 1 2 3 3 4 4 5 5 6 6 7 8
statement            : 0
Parsing method: LALR
state 0
    (0) S' -> . statement
    (1) statement -> . NAME EQUALS expression
    (2) statement -> . expression
    (3) expression -> . expression PLUS expression
    (4) expression -> . expression MINUS expression
    (5) expression -> . expression TIMES expression
    (6) expression -> . expression DIVIDE expression
    (7) expression -> . MINUS expression
    (8) expression -> . LPAREN expression RPAREN
    (9) expression -> . NUMBER
    (10) expression -> . NAME
    NAME            shift and go to state 2
    MINUS           shift and go to state 4
    LPAREN          shift and go to state 5
    NUMBER          shift and go to state 6
    statement                      shift and go to state 1
    expression                     shift and go to state 3
state 1
    (0) S' -> statement .
state 2
    (1) statement -> NAME . EQUALS expression
    (10) expression -> NAME .
    EQUALS          shift and go to state 7
    PLUS            reduce using rule 10 (expression -> NAME .)
    MINUS           reduce using rule 10 (expression -> NAME .)
    TIMES           reduce using rule 10 (expression -> NAME .)
    DIVIDE          reduce using rule 10 (expression -> NAME .)
    $end            reduce using rule 10 (expression -> NAME .)
state 3
    (2) statement -> expression .
    (3) expression -> expression . PLUS expression
    (4) expression -> expression . MINUS expression
    (5) expression -> expression . TIMES expression
    (6) expression -> expression . DIVIDE expression
    $end            reduce using rule 2 (statement -> expression .)
    PLUS            shift and go to state 8
    MINUS           shift and go to state 9
    TIMES           shift and go to state 10
    DIVIDE          shift and go to state 11
state 4
    (7) expression -> MINUS . expression
    (3) expression -> . expression PLUS expression
    (4) expression -> . expression MINUS expression
    (5) expression -> . expression TIMES expression
    (6) expression -> . expression DIVIDE expression
    (7) expression -> . MINUS expression
    (8) expression -> . LPAREN expression RPAREN
    (9) expression -> . NUMBER
    (10) expression -> . NAME
    MINUS           shift and go to state 4
    LPAREN          shift and go to state 5
    NUMBER          shift and go to state 6
    NAME            shift and go to state 13
    expression                     shift and go to state 12
state 5
    (8) expression -> LPAREN . expression RPAREN
    (3) expression -> . expression PLUS expression
    (4) expression -> . expression MINUS expression
    (5) expression -> . expression TIMES expression
    (6) expression -> . expression DIVIDE expression
    (7) expression -> . MINUS expression
    (8) expression -> . LPAREN expression RPAREN
    (9) expression -> . NUMBER
    (10) expression -> . NAME
    MINUS           shift and go to state 4
    LPAREN          shift and go to state 5
    NUMBER          shift and go to state 6
    NAME            shift and go to state 13
    expression                     shift and go to state 14
state 6
    (9) expression -> NUMBER .
    PLUS            reduce using rule 9 (expression -> NUMBER .)
    MINUS           reduce using rule 9 (expression -> NUMBER .)
    TIMES           reduce using rule 9 (expression -> NUMBER .)
    DIVIDE          reduce using rule 9 (expression -> NUMBER .)
    $end            reduce using rule 9 (expression -> NUMBER .)
    RPAREN          reduce using rule 9 (expression -> NUMBER .)
state 7
    (1) statement -> NAME EQUALS . expression
    (3) expression -> . expression PLUS expression
    (4) expression -> . expression MINUS expression
    (5) expression -> . expression TIMES expression
    (6) expression -> . expression DIVIDE expression
    (7) expression -> . MINUS expression
    (8) expression -> . LPAREN expression RPAREN
    (9) expression -> . NUMBER
    (10) expression -> . NAME
    MINUS           shift and go to state 4
    LPAREN          shift and go to state 5
    NUMBER          shift and go to state 6
    NAME            shift and go to state 13
    expression                     shift and go to state 15
state 8
    (3) expression -> expression PLUS . expression
    (3) expression -> . expression PLUS expression
    (4) expression -> . expression MINUS expression
    (5) expression -> . expression TIMES expression
    (6) expression -> . expression DIVIDE expression
    (7) expression -> . MINUS expression
    (8) expression -> . LPAREN expression RPAREN
    (9) expression -> . NUMBER
    (10) expression -> . NAME
    MINUS           shift and go to state 4
    LPAREN          shift and go to state 5
    NUMBER          shift and go to state 6
    NAME            shift and go to state 13
    expression                     shift and go to state 16
state 9
    (4) expression -> expression MINUS . expression
    (3) expression -> . expression PLUS expression
    (4) expression -> . expression MINUS expression
    (5) expression -> . expression TIMES expression
    (6) expression -> . expression DIVIDE expression
    (7) expression -> . MINUS expression
    (8) expression -> . LPAREN expression RPAREN
    (9) expression -> . NUMBER
    (10) expression -> . NAME
    MINUS           shift and go to state 4
    LPAREN          shift and go to state 5
    NUMBER          shift and go to state 6
    NAME            shift and go to state 13
    expression                     shift and go to state 17
state 10
    (5) expression -> expression TIMES . expression
    (3) expression -> . expression PLUS expression
    (4) expression -> . expression MINUS expression
    (5) expression -> . expression TIMES expression
    (6) expression -> . expression DIVIDE expression
    (7) expression -> . MINUS expression
    (8) expression -> . LPAREN expression RPAREN
    (9) expression -> . NUMBER
    (10) expression -> . NAME
    MINUS           shift and go to state 4
    LPAREN          shift and go to state 5
    NUMBER          shift and go to state 6
    NAME            shift and go to state 13
    expression                     shift and go to state 18
state 11
    (6) expression -> expression DIVIDE . expression
    (3) expression -> . expression PLUS expression
    (4) expression -> . expression MINUS expression
    (5) expression -> . expression TIMES expression
    (6) expression -> . expression DIVIDE expression
    (7) expression -> . MINUS expression
    (8) expression -> . LPAREN expression RPAREN
    (9) expression -> . NUMBER
    (10) expression -> . NAME
    MINUS           shift and go to state 4
    LPAREN          shift and go to state 5
    NUMBER          shift and go to state 6
    NAME            shift and go to state 13
    expression                     shift and go to state 19
state 12
    (7) expression -> MINUS expression .
    (3) expression -> expression . PLUS expression
    (4) expression -> expression . MINUS expression
    (5) expression -> expression . TIMES expression
    (6) expression -> expression . DIVIDE expression
    PLUS            reduce using rule 7 (expression -> MINUS expression .)
    MINUS           reduce using rule 7 (expression -> MINUS expression .)
    TIMES           reduce using rule 7 (expression -> MINUS expression .)
    DIVIDE          reduce using rule 7 (expression -> MINUS expression .)
    $end            reduce using rule 7 (expression -> MINUS expression .)
    RPAREN          reduce using rule 7 (expression -> MINUS expression .)
  ! PLUS            [ shift and go to state 8 ]
  ! MINUS           [ shift and go to state 9 ]
  ! TIMES           [ shift and go to state 10 ]
  ! DIVIDE          [ shift and go to state 11 ]
state 13
    (10) expression -> NAME .
    PLUS            reduce using rule 10 (expression -> NAME .)
    MINUS           reduce using rule 10 (expression -> NAME .)
    TIMES           reduce using rule 10 (expression -> NAME .)
    DIVIDE          reduce using rule 10 (expression -> NAME .)
    $end            reduce using rule 10 (expression -> NAME .)
    RPAREN          reduce using rule 10 (expression -> NAME .)
state 14
    (8) expression -> LPAREN expression . RPAREN
    (3) expression -> expression . PLUS expression
    (4) expression -> expression . MINUS expression
    (5) expression -> expression . TIMES expression
    (6) expression -> expression . DIVIDE expression
    RPAREN          shift and go to state 20
    PLUS            shift and go to state 8
    MINUS           shift and go to state 9
    TIMES           shift and go to state 10
    DIVIDE          shift and go to state 11
state 15
    (1) statement -> NAME EQUALS expression .
    (3) expression -> expression . PLUS expression
    (4) expression -> expression . MINUS expression
    (5) expression -> expression . TIMES expression
    (6) expression -> expression . DIVIDE expression
    $end            reduce using rule 1 (statement -> NAME EQUALS expression .)
    PLUS            shift and go to state 8
    MINUS           shift and go to state 9
    TIMES           shift and go to state 10
    DIVIDE          shift and go to state 11
state 16
    (3) expression -> expression PLUS expression .
    (3) expression -> expression . PLUS expression
    (4) expression -> expression . MINUS expression
    (5) expression -> expression . TIMES expression
    (6) expression -> expression . DIVIDE expression
    PLUS            reduce using rule 3 (expression -> expression PLUS expression .)
    MINUS           reduce using rule 3 (expression -> expression PLUS expression .)
    $end            reduce using rule 3 (expression -> expression PLUS expression .)
    RPAREN          reduce using rule 3 (expression -> expression PLUS expression .)
    TIMES           shift and go to state 10
    DIVIDE          shift and go to state 11
  ! TIMES           [ reduce using rule 3 (expression -> expression PLUS expression .) ]
  ! DIVIDE          [ reduce using rule 3 (expression -> expression PLUS expression .) ]
  ! PLUS            [ shift and go to state 8 ]
  ! MINUS           [ shift and go to state 9 ]
state 17
    (4) expression -> expression MINUS expression .
    (3) expression -> expression . PLUS expression
    (4) expression -> expression . MINUS expression
    (5) expression -> expression . TIMES expression
    (6) expression -> expression . DIVIDE expression
    PLUS            reduce using rule 4 (expression -> expression MINUS expression .)
    MINUS           reduce using rule 4 (expression -> expression MINUS expression .)
    $end            reduce using rule 4 (expression -> expression MINUS expression .)
    RPAREN          reduce using rule 4 (expression -> expression MINUS expression .)
    TIMES           shift and go to state 10
    DIVIDE          shift and go to state 11
  ! TIMES           [ reduce using rule 4 (expression -> expression MINUS expression .) ]
  ! DIVIDE          [ reduce using rule 4 (expression -> expression MINUS expression .) ]
  ! PLUS            [ shift and go to state 8 ]
  ! MINUS           [ shift and go to state 9 ]
state 18
    (5) expression -> expression TIMES expression .
    (3) expression -> expression . PLUS expression
    (4) expression -> expression . MINUS expression
    (5) expression -> expression . TIMES expression
    (6) expression -> expression . DIVIDE expression
    PLUS            reduce using rule 5 (expression -> expression TIMES expression .)
    MINUS           reduce using rule 5 (expression -> expression TIMES expression .)
    TIMES           reduce using rule 5 (expression -> expression TIMES expression .)
    DIVIDE          reduce using rule 5 (expression -> expression TIMES expression .)
    $end            reduce using rule 5 (expression -> expression TIMES expression .)
    RPAREN          reduce using rule 5 (expression -> expression TIMES expression .)
  ! PLUS            [ shift and go to state 8 ]
  ! MINUS           [ shift and go to state 9 ]
  ! TIMES           [ shift and go to state 10 ]
  ! DIVIDE          [ shift and go to state 11 ]
state 19
    (6) expression -> expression DIVIDE expression .
    (3) expression -> expression . PLUS expression
    (4) expression -> expression . MINUS expression
    (5) expression -> expression . TIMES expression
    (6) expression -> expression . DIVIDE expression
    PLUS            reduce using rule 6 (expression -> expression DIVIDE expression .)
    MINUS           reduce using rule 6 (expression -> expression DIVIDE expression .)
    TIMES           reduce using rule 6 (expression -> expression DIVIDE expression .)
    DIVIDE          reduce using rule 6 (expression -> expression DIVIDE expression .)
    $end            reduce using rule 6 (expression -> expression DIVIDE expression .)
    RPAREN          reduce using rule 6 (expression -> expression DIVIDE expression .)
  ! PLUS            [ shift and go to state 8 ]
  ! MINUS           [ shift and go to state 9 ]
  ! TIMES           [ shift and go to state 10 ]
  ! DIVIDE          [ shift and go to state 11 ]
state 20
    (8) expression -> LPAREN expression RPAREN .
    PLUS            reduce using rule 8 (expression -> LPAREN expression RPAREN .)
    MINUS           reduce using rule 8 (expression -> LPAREN expression RPAREN .)
    TIMES           reduce using rule 8 (expression -> LPAREN expression RPAREN .)
    DIVIDE          reduce using rule 8 (expression -> LPAREN expression RPAREN .)
    $end            reduce using rule 8 (expression -> LPAREN expression RPAREN .)
    RPAREN          reduce using rule 8 (expression -> LPAREN expression RPAREN .)

# parsetab.py
# This file is automatically generated. Do not edit.
# pylint: disable=W,C,R
_tabversion = '3.10'
_lr_method = 'LALR'
_lr_signature = 'leftPLUSMINUSleftTIMESDIVIDEnonassocUMINUSDIVIDE EQUALS LPAREN MINUS NAME NUMBER PLUS RPAREN TIMESstatement : NAME EQUALS expressionstatement : expressionexpression : expression PLUS expression\n                  | expression MINUS expression\n                  | expression TIMES expression\n                  | expression DIVIDE expressionexpression : MINUS expression %prec UMINUSexpression : LPAREN expression RPARENexpression : NUMBERexpression : NAME'
    
_lr_action_items = {'NAME':([0,4,5,7,8,9,10,11,],[2,13,13,13,13,13,13,13,]),'MINUS':([0,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,],[4,-10,9,4,4,-9,4,4,4,4,4,-7,-10,9,9,-3,-4,-5,-6,-8,]),'LPAREN':([0,4,5,7,8,9,10,11,],[5,5,5,5,5,5,5,5,]),'NUMBER':([0,4,5,7,8,9,10,11,],[6,6,6,6,6,6,6,6,]),'$end':([1,2,3,6,12,13,15,16,17,18,19,20,],[0,-10,-2,-9,-7,-10,-1,-3,-4,-5,-6,-8,]),'EQUALS':([2,],[7,]),'PLUS':([2,3,6,12,13,14,15,16,17,18,19,20,],[-10,8,-9,-7,-10,8,8,-3,-4,-5,-6,-8,]),'TIMES':([2,3,6,12,13,14,15,16,17,18,19,20,],[-10,10,-9,-7,-10,10,10,10,10,-5,-6,-8,]),'DIVIDE':([2,3,6,12,13,14,15,16,17,18,19,20,],[-10,11,-9,-7,-10,11,11,11,11,-5,-6,-8,]),'RPAREN':([6,12,13,14,16,17,18,19,20,],[-9,-7,-10,20,-3,-4,-5,-6,-8,]),}
_lr_action = {}
for _k, _v in _lr_action_items.items():
   for _x,_y in zip(_v[0],_v[1]):
      if not _x in _lr_action:  _lr_action[_x] = {}
      _lr_action[_x][_k] = _y
del _lr_action_items
_lr_goto_items = {'statement':([0,],[1,]),'expression':([0,4,5,7,8,9,10,11,],[3,12,14,15,16,17,18,19,]),}
_lr_goto = {}
for _k, _v in _lr_goto_items.items():
   for _x, _y in zip(_v[0], _v[1]):
       if not _x in _lr_goto: _lr_goto[_x] = {}
       _lr_goto[_x][_k] = _y
del _lr_goto_items
_lr_productions = [
  ("S' -> statement","S'",1,None,None,None),
  ('statement -> NAME EQUALS expression','statement',3,'p_statement_assign','main.py',57),
  ('statement -> expression','statement',1,'p_statement_expr','main.py',61),
  ('expression -> expression PLUS expression','expression',3,'p_expression_binop','main.py',65),
  ('expression -> expression MINUS expression','expression',3,'p_expression_binop','main.py',66),
  ('expression -> expression TIMES expression','expression',3,'p_expression_binop','main.py',67),
  ('expression -> expression DIVIDE expression','expression',3,'p_expression_binop','main.py',68),
  ('expression -> MINUS expression','expression',2,'p_expression_uminus','main.py',75),
  ('expression -> LPAREN expression RPAREN','expression',3,'p_expression_group','main.py',79),
  ('expression -> NUMBER','expression',1,'p_expression_number','main.py',83),
  ('expression -> NAME','expression',1,'p_expression_name','main.py',87),
]

[tool.poetry]
name = "python-template"
version = "0.1.0"
description = ""
authors = ["Your Name <you@example.com>"]
[tool.poetry.dependencies]
python = ">=3.10.0,<3.11"
ply = "^3.11"
[tool.pyright]
# https://github.com/microsoft/pyright/blob/main/docs/configuration.md
useLibraryCodeForTypes = true
exclude = [".cache"]
[tool.ruff]
# https://beta.ruff.rs/docs/configuration/
select = ['E', 'W', 'F', 'I', 'B', 'C4', 'ARG', 'SIM']
ignore = ['W291', 'W292', 'W293']
[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
