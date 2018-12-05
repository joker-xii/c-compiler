# compiler

A simple C compiler with limited grammar supported


## Frontend 

### grammar
```
program : decl_list ;
decl_list : decl decl_list
        | ;
decl : type_spec id {push_id} var_or_func 
        | struct_decl 
        | ';' ;
var_or_func : '(' {func_begin}  params ')' func_follow {func_end}
        | var_def_follow var_def_list_follow ;
func_follow : ';' 
        | code_block ;
struct_decl : 'struct' id {push_id} '{' {struct_begin}  def_list '}' ';' {struct_end} ; 
exp_or_str : exp  
        | string {push_str} ;
type_spec : int_type 
        | float_type 
        | char_type 
        | 'void' {void} 
        | 'struct' id {push_id} {struct_var} ;
int_type : int16_type {int16}
        | uint16_type {uint16}
        | int32_type {int32}
        | uint32_type {uint32}
        | int64_type {int64}
        | uint64_type {uint64} ;
int16_type : 'short' ;
uint16_type : 'unsigned' 'short' ;
int32_type : 'int' 
        | 'long' 
        | 'long' 'int' ;
uint32_type : 'unsigned' 'int' 
        |  'unsigned' 
        | 'unsigned' 'long' ;
int64_type :  'long' 'long' 'int' 
        | 'long' 'long' ;
uint64_type :  'unsigned' 'long' 'long' 
        |  'unsigned' 'long' 'long' 'int' ;

float_type : 'float' {float32}
        | 'double' {float64} ;
char_type : 'char' {char8}
        | 'unsigned' 'char' {uchar8} ;
params : param_list 
        | ;
param_list : param  param_follow ;
param_follow : ',' param param_follow 
        | ;
param :  type_spec  id {push_id} param_suffix ;
param_suffix :  '[' ']' {param_array}
        | {param_var} ;
code_block :  '{' {code_block} def_list code_list  '}' {code_block_end} ;
def_list : var_def def_list 
        | ;
var_def : type_spec var_def_list ;
var_def_list :  id {push_id} var_def_follow var_def_list_follow  ;
var_def_list_follow : ',' id {push_id} var_def_follow  var_def_list_follow 
        | ';' {var_def_list_end} ;
var_def_follow : '[' int_const {def_array} ']'  array_init {def_array_end}
        | '=' exp {var_init_end}
        | {var_def_end} ;
array_init : '='  array_init_follow  
        | ;
array_init_follow :  '{' {init_list_begin}  initializer_list '}' {init_list_end}
        | string {push_str} {arr_init_str} ;
initializer_list : initializer  initializer_follow 
        | ;
initializer_follow : ',' initializer initializer_follow 
        | ;
initializer : exp {init_list_item} ;

code_list : code code_list 
        | ;
code : normal_stmt 
        | branch_stmt 
        | iteration_stmt 
        | return_stmt 
        | code_block ;
normal_stmt : ';' 
        | exp_or_str ';' {pop_top}
        | 'break' ';' {break}
        | 'continue' ';' {continue} ;
call_func : '(' call_params ')' ;
call_params : call_param_list 
        | ;
call_param_list : exp_or_str {call_func_push} call_param_follow ;
call_param_follow : ',' exp_or_str {call_func_push} call_param_follow 
        | ;
branch_stmt : if_stmt
        | switch_stmt ;
switch_stmt : 'switch' {switch_begin} '(' exp ')' '{' case_list default '}' {switch_end} ;
i_c : int_const {push_int}
        | char_const {push_char} ;
case_list : case case_list 
        | ;
case : 'case' i_c {case_begin} ':' code {case_end} ;
default : 'default' ':' code ;
if_stmt :  'if' {if_begin} '(' exp_or_str {if_check} ')' code {if_end} else_stmt {else_end} ;
else_stmt : 'else'  code
        | ;
iteration_stmt : 'while' {while_begin} '(' exp_or_str {while_check} ')' code {while_end} 
        | 'for' '(' for_exp {for_init} ';'  for_exp {for_check} ';' for_exp {for_suf} ')' code {for_end} ;
for_exp : exp_or_str ; 
return_stmt : 'return' return_follow ;
return_follow : ';' {return} 
        | exp_or_str {return@} ';'  ;
exp	:  term_assign exp_assign ;
exp_assign :  '=' term_assign {@=@} exp_assign 
        |   '+=' term_assign {@+=@} exp_assign 
        |   '-=' term_assign {@-=@} exp_assign 
        |   '*=' term_assign {@*=@} exp_assign 
        |   '/=' term_assign {@/=@} exp_assign 
        |   '%=' term_assign {@%=@} exp_assign 
        |   '<<=' term_assign {@<<=@} exp_assign 
        |   '>>=' term_assign {@>>=@} exp_assign 
        |   '&=' term_assign {@&=@} exp_assign 
        |   '^=' term_assign {@^=@} exp_assign 
        |   '|=' term_assign {@|=@} exp_assign 
        |  ;
term_assign : term_log_or exp_log_or ;
exp_log_or :  '||' term_log_or {@||@} exp_log_or 
        |  ;
term_log_or : term_log_and exp_log_and ;
exp_log_and :  '&&' term_log_and {@&&@} exp_log_and 
        |  ;
term_log_and : term_bit_or exp_bit_or ;
exp_bit_or : '|' term_bit_or {@|@} exp_bit_or 
        |  ;
term_bit_or : term_bit_xor exp_bit_xor ;
exp_bit_xor : '^' term_bit_xor {@^@} exp_bit_xor 
        |  ;
term_bit_xor : term_bit_and exp_bit_and ;
exp_bit_and : '&' term_bit_and {@&@} exp_bit_and 
        |  ;
term_bit_and : term_eq exp_eq ;
exp_eq : '==' term_eq {@==@} exp_eq 
        | '!=' term_eq {@!=@} exp_eq 
        |  ;
term_eq : term_cmp exp_cmp ;
exp_cmp : '>=' term_cmp {@>=@} exp_cmp 
        | '<=' term_cmp {@<=@} exp_cmp 
        | '>' term_cmp {@>@} exp_cmp 
        | '<' term_cmp {@<@} exp_cmp 
        |  ;
term_cmp : term_shift exp_shift ;
exp_shift : '<<' term_shift {@<<@} exp_shift 
        | '>>' term_shift {@>>@} exp_shift
        | ;
term_shift : term_add exp_add ;
exp_add : '+' term_add {@+@} exp_add 
        | '-' term_add {@-@} exp_add 
        | ;
term_add : term_mul exp_mul ;
exp_mul : '*' term_mul {@*@} exp_mul 
        | '/' term_mul {@/@} exp_mul 
        | '%' term_mul {@%@} exp_mul 
        | ;
term_mul : term_prefix ;
term_prefix : term_suffix 
        | '++' term_prefix {++@}
        | '--' term_prefix {--@}
        | '!' term_prefix {!@}
        | '~' term_prefix {~@}
        | '+' term_prefix {+@}
        | '-' term_prefix {-@} ;
term_suffix : factors inc_or_dec ;
inc_or_dec : '++' {@++} 
        | '--' {@--}
        | ;
factors : int_const {push_int}
        | float_const {push_float}
        | char_const {push_char}
        | id {push_id} id_suffix
        | '(' exp ')' ;

id_suffix : '[' exp ']'  {@[@]} 
        | call_func {call_func} 
        | '.' id {@.@} 
        | ;


```

### intermediate representation 
|QUAT TYPE | VAL1 | VAL2 | VAL3 | OPERATION|
|----------|------|------|------|----------|
|label | pos | 0 | 0|定义一个叫做pos的label|
|jmp | pos| 0 | 0|无条件跳转到名叫pos的label|
|btrue | pos| pred | 0|若pred为真 跳转到pos|
|bfalse | pos| pred | 0|若pred为假 跳转到pos|
|newvar | id | type | array |新建一个type类型的名字叫id的变量 若array为0是变量，array不为0是长度为array的数组|
|func | id | type | 0|名字为id 返回值为type的函数|
|funcparam | id | type | array|  func的参数，类型是type，array不为零为数组|
|funcend | id | 0| 0 | 函数的结束|
|structdef | id | 0 | 0 |    名字为id的struct|
|structend| id |  0 | 0   |   名字为id的struct定义结束 |
|cblock | 0 |  0  |   0|  标记一个代码块的开始|
|cend | 0 | 0  |  0|      ~~~~~~~~~~~~~~结束|        
|push | val | 0 |0  |     将val push进堆栈|
|call | id | 0 | to   |  调用id函数，返回值存在to中|
|ret| 0 | 0 |0 |   函数返回|
|retval| 0| 0 |to |  函数返回，返回值存在to中|
|op | lhs| rhs| to | 双目运算符，运算结果存入to中|
|op | val| 0| to |单目运算符，运算结果存入to中|
|assign | lhs| rhs| type | lhs ?= rhs  |
|initlst|id|0|0 |  初始化列表|
|initend|id|0|0| 初始化列表结束|
|initlstitem | id | 0 |0 | 初始化列表元素|