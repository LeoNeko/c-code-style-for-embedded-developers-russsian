# Рекомендуемый стиль и правила написания кода на языке С для разработчиков встроенных систем

Этот документ является форком репозитория пользователя `Tilen MAJERLE` с исправлениями и дополнениями `LeoNeko`.

## Содержание документа

  - [Самое важное правило](#самое-важное-правило)
  - [Главные правила](#главные-правила)
  - [Comments](#comments)
  - [Функции](#Функции)
  - [Variables](#variables)
  - [Structures, enumerations, typedefs](#structures-enumerations-typedefs)
  - [Compound statements](#compound-statements)
    - [Switch statement](#switch-statement)
  - [Macros and preprocessor directives](#macros-and-preprocessor-directives)
  - [Документация](#документация)
  - [Заголовочные и исходного кода файлы](#заголовочные-и-исходного-кода-файлы)
  - [Стиль наименования файлов](#стиль-наименования-файлов)

## Самое важное правило

Давайте начнём с цитаты разработчиков с сайта [GNOME developer](https://developer.gnome.org/programming-guidelines/stable/c-coding-style.html.en).

> Единственное наиболее важное правило при написании кода таково: проверяйте окружающий код и старайтесь подстроиться под него.
> Мне как сопровождающему неприятно получать патч, явно отличающийся от окружающего кода стилем кодирования. Это неуважение, как если бы кто-то вошел в безупречно чистый дом в грязных ботинках.
> Итак, кто бы ни рекомендовал этот документ, если код уже написан и вы его исправляете, сохраняйте его текущий стиль, даже если это не ваш любимый стиль.

## Главные правила

Здесь перечислены наиболее очевидные и важные общие правила. Внимательно прочитайте их, прежде чем переходить к другим главам.

- Используй стандарт `C99`.
- Не используй табуляции, вместо этого используй пробелы.
- Используй `4` пробела на каждый уровень доступа (область видимости).
- Используй `1` пробел между ключевым словом и открывающей скобкой.

```c
/* ПРАВИЛЬНО */
if (condition)
while (condition)
for (init; condition; step)
do {} while (condition)

/* НЕ ПРАВИЛЬНО */
if(condition)
while(condition)
for(init;condition;step)
do {} while(condition)
```

- Я рекомендую использовать пробел между именем функции и открывающей скобкой. Это помогает физуально выделить имя функции в потоке текста и сделать на ней акцент, даже без подсветки (в оригинале авор советует наоборот). Также это хорошо сочетается с другими правилами по стилю кода.
- Всегда выравнивайте свой код по линии операторов, имен переменных, комментариев и т.д. Это также упрощает восприятие.

```c
int32_t a = sum (4, 3);            /* ПРАВИЛЬНО  */
int32_t a = sum(4, 3);             /* НЕ ПРАВИЛЬНО */
```

- НИКОГДА не используйте `__` или `_` в начале для переменных/функций/макросов/типов данных. Это зарезервировано языком С для себя.
    - Предпочтительно использовать префикс `prv_` для функций, строго зависящих от модуля.
- Используй только нижний регистр для переменных/функций/макросов с опциональным нижнем подчеркиванием `_`.
- Для пользовательских типво данных предлагается его нарочитое выделение `MODBUS_StatusTypeDef`. Таким образом подчеркивается, что тип данных не относится к стандартным типам какой-либо библиотеки, а также помогает визуально зацепиться глазу за него.
- Открывайте скобку всегда на той же строчке, что и ключевые слова (`for`, `while`, `do`, `switch`, `if`, ...). Это сэкономит строчки кода.
```c
size_t i;
for (i = 0; i < 5; ++i) {           /* ПРАВИЛЬНО */
}

for (i = 0; i < 5; ++i){            /* НЕ ПРАВИЛЬНО */
}

for (i = 0; i < 5; ++i)             /* НЕ ПРАВИЛЬНО */
{
}
```

- Используйте один пробел до и после операторов сравнения и присваивания. Тоже самое и для битовых операций.
```c
int32_t a;
a = 3 + 4;              /* ПРАВИЛЬНО */
for (a = 0; a < 5; ++a) /* ПРАВИЛЬНО */
a = (spr & 0x80);       /* ПРАВИЛЬНО */
a=3+4;                  /* НЕ ПРАВИЛЬНО */
a = 3+4;                /* НЕ ПРАВИЛЬНО */
for (a=0;a<5;++a)       /* НЕ ПРАВИЛЬНО */
```

- Используйте один пробел после каждой запятой:
```c
func_name(5, 4);        /* ПРАВИЛЬНО */
func_name(4,3);         /* НЕ ПРАВИЛЬНО */
```

- Не инициализируейте static и global переменные 0 (или NULL), дайте компилятору сделать это:
```c
static int32_t a;       /* ПРАВИЛЬНО */
static int32_t b = 4;   /* ПРАВИЛЬНО */
static int32_t a = 0;   /* НЕ ПРАВИЛЬНО */

void
my_func(void) {
    static int32_t* ptr;/* ПРАВИЛЬНО */
    static char abc = 0;/* НЕ ПРАВИЛЬНО */
}
```

- Объявите все локальные переменные одного типа в одной строке:
```c
void
my_func(void) {
    char a;             /* ПРАВИЛЬНО */
    char a, b;          /* ПРАВИЛЬНО*/
    char b;             /* НЕ ПРАВИЛЬНО, эта переменная уже существует */
}
```

- Порядок объявления локальных переменных
    1. Пользовательские структуры и перечисления, указатели
    2. Сначала типы Integer, unsigned type
    3. А затем Single/Double floating
```c
int
my_func(void) {
    /* 1 */
    my_struct_t my;     /* Пользовательская структура */
    my_struct_ptr_t* p; /* Указатель */

    /* 2 */
    uint32_t a;
    int32_t b;
    uint16_t c;
    int16_t g;
    char h;
    /* ... */

    /* 3 */
    double d;
    float f;
}
```

- Всегда объявляйте локальные переменные в начале блока перед первым исполняемым оператором.

- Всегда объявляйте счётчик в цикле `for`:
```c
/* ПРАВИЛЬНО  */
for (size_t i = 0; i < 10; ++i)

/* ПРАВИЛЬНО, если вам надо использовать счётчик i вне цикла */
size_t i;
for (i = 0; i < 10; ++i) {
    if (...) {
        break;
    }
}
if (i == 10) {

}

/* НЕ ПРАВИЛЬНО, в любом другом случае */
size_t i;
for (i = 0; i < 10; ++i) ...
```

- Избегайте присваивания переменной с вызовом функции в объявлении, за исключением одиночных переменных:
```c
void
a(void) {
    /* Избегайте вызовов функций при объявлении переменной */
    int32_t a, b = sum(1, 2);

    /* Можно сделать так */
    int32_t a, b;
    b = sum(1, 2);

    /* А лучше так */
    uint8_t a = 3, b = 4;
}
```

- Кроме `char`, `float` или `double`, всегда используйте типы объявленные в бибилиотеке stdint.h, например `uint8_t` для `unsigned 8-bit` и т.д.
- Не используйте бибилиотеку `stdbool.h`. Используйте `1` или `0` для `true` и `false`.
```c
/* ПРАВИЛЬНО */
uint8_t status;
status = 0;

/* НЕ ПРАВИЛЬНО */
#include <stdbool.h>
bool status = true;
```

- Never compare against `true`, eg. `if (check_func() == 1)`, use `if (check_func()) { ... }`
- Always compare pointers against `NULL` value
```c
void* ptr;

/* ... */

/* OK, compare against NULL */
if (ptr == NULL || ptr != NULL) {

}

/* Wrong */
if (ptr || !ptr) {

}
```

- Always use *pre-increment (and decrement respectively)* instead of *post-increment (and decrement respectively)*
```c
int32_t a = 0;
...

a++;            /* Wrong */
++a;            /* OK */

for (size_t j = 0; j < 10; ++j) {}  /* OK */
```

- Always use `size_t` for length or size variables
- Always use `const` for pointer if function should not modify memory pointed to by `pointer`
- Always use `const` for function parameter or variable, if it should not be modified
```c

/* When d could be modified, data pointed to by d could not be modified */
void
my_func(const void* d) {

}

/* When d and data pointed to by d both could not be modified */
void
my_func(const void* const d) {

}

/* Not required, it is advised */
void
my_func(const size_t len) {

}

/* When d should not be modified inside function, only data pointed to by d could be modified */
void
my_func(void* const d) {

}
```

- When function may accept pointer of any type, always use `void *`, do not use `uint8_t *`
    - Function must take care of proper casting in implementation
```c
/*
 * To send data, function should not modify memory pointed to by `data` variable
 * thus `const` keyword is important
 *
 * To send generic data (or to write them to file)
 * any type may be passed for data,
 * thus use `void *`
 */
/* OK example */
void
send_data(const void* data, size_t len) { /* OK */
    /* Do not cast `void *` or `const void *` */
    const uint8_t* d = data;/* Function handles proper type for internal usage */
}

void
send_data(const void* data, int len) {    /* Wrong, not not use int */
}
```

- Always use brackets with `sizeof` operator
- Never use *Variable Length Array* (VLA). Use dynamic memory allocation instead with standard C `malloc` and `free` functions or if library/project provides custom memory allocation, use its implementation
    - Take a look at [LwMEM](https://github.com/MaJerle/lwmem), custom memory management library
```c
/* OK */
#include <stdlib.h>
void
my_func(size_t size) {
    int32_t* arr;
    arr = malloc(sizeof(*arr) * n); /* OK, Allocate memory */
    arr = malloc(sizeof *arr * n);  /* Wrong, brackets for sizeof operator are missing */
    if (arr == NULL) {
        /* FAIL, no memory */
    }

    free(arr);  /* Free memory after usage */
}

/* Wrong */
void
my_func(size_t size) {
    int32_t arr[size];  /* Wrong, do not use VLA */
}
```

- Always compare variable against zero, except if it is treated as `boolean` type
- Never compare `boolean-treated` variables against zero or one. Use NOT (`!`) instead
```c
size_t length = 5;  /* Counter variable */
uint8_t is_ok = 0;  /* Boolean-treated variable */
if (length)         /* Wrong, length is not treated as boolean */
if (length > 0)     /* OK, length is treated as counter variable containing multi values, not only 0 or 1 */
if (length == 0)    /* OK, length is treated as counter variable containing multi values, not only 0 or 1 */

if (is_ok)          /* OK, variable is treated as boolean */
if (!is_ok)         /* OK, -||- */
if (is_ok == 1)     /* Wrong, never compare boolean variable against 1! */
if (is_ok == 0)     /* Wrong, use ! for negative check */
```

- Always use `/* comment */` for comments, even for *single-line* comment
- Always include check for `C++` with `extern` keyword in header file
- Every function must include *doxygen-enabled* comment, even if function is `static`
- Use English names/text for functions, variables, comments
- Use *lowercase* characters for variables
- Use *underscore* if variable contains multiple names, eg. `force_redraw`. Do not use `forceRedraw`
- Never cast function returning `void *`, eg. `uint8_t* ptr = (uint8_t *)func_returning_void_ptr();` as `void *` is safely promoted to any other pointer type
    - Use `uint8_t* ptr = func_returning_void_ptr();` instead
- Always use `<` and `>` for C Standard Library include files, eg. `#include <stdlib.h>`
- Always use `""` for custom libraries, eg. `#include "my_library.h"`
- When casting to pointer type, always align asterisk to type, eg. `uint8_t* t = (uint8_t*)var_width_diff_type`
- Always respect code style already used in project or library

## Comments

- Comments starting with `//` are not allowed. Always use `/* comment */`, even for single-line comment
```c
//This is comment (wrong)
/* This is comment (ok) */
```

- For multi-line comments use `space+asterisk` for every line
```c
/*
 * This is multi-line comments,
 * written in 2 lines (ok)
 */

/**
 * Wrong, use double-asterisk only for doxygen documentation
 */

/*
* Single line comment without space before asterisk (wrong)
*/

/*
 * Single line comment in multi-line configuration (wrong)
 */

/* Single line comment (ok) */
```

- Use `12` indents (`12 * 4` spaces) offset when commenting. If statement is larger than `12` indents, make comment `4-spaces` aligned (examples below) to next available indent
```c
void
my_func(void) {
    char a, b;

    a = call_func_returning_char_a(a);          /* This is comment with 12*4 spaces indent from beginning of line */
    b = call_func_returning_char_a_but_func_name_is_very_long(a);   /* This is comment, aligned to 4-spaces indent */
}
```

## Функции

- Every function which may have access from outside its module, must include function *prototype* (or *declaration*)
- Function name must be lowercase, optionally separated with underscore `_` character
```c
/* OK */
void my_func(void);
void myfunc(void);

/* Wrong */
void MYFunc(void);
void myFunc();
```

- When function returns pointer, align asterisk to return type
```c
/* OK */
const char* my_func(void);
my_struct_t* my_func(int32_t a, int32_t b);

/* Wrong */
const char *my_func(void);
my_struct_t * my_func(void);
```
- Align all function prototypes (with the same/similar functionality) for better readability
```c
/* OK, function names aligned */
void        set(int32_t a);
my_type_t   get(void);
my_ptr_t*   get_ptr(void);

/* Wrong */
void set(int32_t a);
const char * get(void);
```

- Function implementation must include return type and optional other keywords in separate line
```c
/* OK */
int32_t
foo(void) {
    return 0;
}

/* OK */
static const char*
get_string(void) {
    return "Hello world!\r\n";
}

/* Wrong */
int32_t foo(void) {
    return 0;
}
```

## Variables

- Make variable name all lowercase with optional underscore `_` character
```c
/* OK */
int32_t a;
int32_t my_var;
int32_t myvar;

/* Wrong */
int32_t A;
int32_t myVar;
int32_t MYVar;
```

- Group local variables together by `type`
```c
void
foo(void) {
    int32_t a, b;   /* OK */
    char a;
    char b;         /* Wrong, char type already exists */
}
```

- Do not declare variable after first executable statement
```c
void
foo(void) {
    int32_t a;
    a = bar();
    int32_t b;      /* Wrong, there is already executable statement */
}
```

- You may declare new variables inside next indent level
```c
int32_t a, b;
a = foo();
if (a) {
    int32_t c, d;   /* OK, c and d are in if-statement scope */
    c = foo();
    int32_t e;      /* Wrong, there was already executable statement inside block */
}
```

- Declare pointer variables with asterisk aligned to type
```c
/* OK */
char* a;

/* Wrong */
char *a;
char * a;
```

- When declaring multiple pointer variables, you may declare them with asterisk aligned to variable name
```c
/* OK */
char *p, *n;
```

## Structures, enumerations, typedefs

- Structure or enumeration name must be lowercase with optional underscore `_` character between words
- Structure or enumeration may contain `typedef` keyword
- All structure members must be lowercase
- All enumeration members must be uppercase
- Structure/enumeration must follow doxygen documentation syntax

When structure is declared, it may use one of `3` different options:

1. When structure is declared with *name only*, it *must not* contain `_t` suffix after its name.
```c
struct struct_name {
    char* a;
    char b;
};
```
2. When structure is declared with *typedef only*, it *has to* contain `_t` suffix after its name.
```c
typedef struct {
    char* a;
    char b;
} struct_name_t;
```
3. When structure is declared with *name and typedef*, it *must not* contain `_t` for basic name and it *has to* contain `_t` suffix after its name for typedef part.
```c
typedef struct struct_name {
    char* a;
    char b;
    char c;
} struct_name_t;
```

Examples of bad declarations and their suggested corrections
```c
/* a and b must be separated to 2 lines */
/* Name of structure with typedef must include _t suffix */
typedef struct {
    int32_t a, b;
} a;

/* Corrected version */
typedef struct {
    int32_t a;
    int32_t b;
} a_t;

/* Wrong name, it must not include _t suffix */
struct name_t {
    int32_t a;
    int32_t b;
};

/* Wrong parameters, must be all uppercase */
typedef enum {
    MY_ENUM_TESTA,
    my_enum_testb,
} my_enum_t;
```

- When initializing structure on declaration, use `C99` initialization style
```c
/* OK */
a_t a = {
    .a = 4,
    .b = 5,
};

/* Wrong */
a_t a = {1, 2};
```

- When new typedef is introduced for function handles, use `_fn` suffix
```c
/* Function accepts 2 parameters and returns uint8_t */
/* Name of typedef has `_fn` suffix */
typedef uint8_t (*my_func_typedef_fn)(uint8_t p1, const char* p2);
```

## Compound statements

- Every compound statement must include opening and closing curly bracket, even if it includes only `1` nested statement
- Every compound statement must include single indent; when nesting statements, include `1` indent size for each nest
```c
/* OK */
if (c) {
    do_a();
} else {
    do_b();
}

/* Wrong */
if (c)
    do_a();
else
    do_b();

/* Wrong */
if (c) do_a();
else do_b();
```

- In case of `if` or `if-else-if` statement, `else` must be in the same line as closing bracket of first statement
```c
/* OK */
if (a) {

} else if (b) {

} else {

}

/* Wrong */
if (a) {

}
else {

}

/* Wrong */
if (a) {

}
else
{

}
```

- In case of `do-while` statement, `while` part must be in the same line as closing bracket of `do` part
```c
/* OK */
do {
    int32_t a;
    a = do_a();
    do_b(a);
} while (check());

/* Wrong */
do
{
/* ... */
} while (check());

/* Wrong */
do {
/* ... */
}
while (check());
```

- Indentation is required for every opening bracket
```c
if (a) {
    do_a();
} else {
    do_b();
    if (c) {
        do_c();
    }
}
```

- Never do compound statement without curly bracket, even in case of single statement. Examples below show bad practices
```c
if (a) do_b();
else do_c();

if (a) do_a(); else do_b();
```

- Empty `while`, `do-while` or `for` loops must include brackets
```c
/* OK */
while (is_register_bit_set()) {}

/* Wrong */
while (is_register_bit_set());
while (is_register_bit_set()) { }
while (is_register_bit_set()) {
}
```

- If `while` (or `for`, `do-while`, etc) is empty (it can be the case in embedded programming), use empty single-line brackets
```c
/* Wait for bit to be set in embedded hardware unit
uint32_t* addr = HW_PERIPH_REGISTER_ADDR;

/* Wait bit 13 to be ready */
while (*addr & (1 << 13)) {}        /* OK, empty loop contains no spaces inside curly brackets */
while (*addr & (1 << 13)) { }       /* Wrong */
while (*addr & (1 << 13)) {         /* Wrong */

}
while (*addr & (1 << 13));          /* Wrong, curly brackets are missing. Can lead to compiler warnings or unintentional bugs */
```
- Always prefer using loops in this order: `for`, `do-while`, `while`
- Avoid incrementing variables inside loop block if possible, see examples

```c
/* Not recommended */
int32_t a = 0;
while (a < 10) {
    .
    ..
    ...
    ++a;
}

/* Better */
for (size_t a = 0; a < 10; ++a) {

}

/* Better, if inc may not happen in every cycle */
for (size_t a = 0; a < 10; ) {
    if (...) {
        ++a;
    }
}
```

### Switch statement

- Add *single indent* for every `case` statement
- Use additional *single indent* for `break` statement in each `case` or `default`
```c
/* OK, every case has single indent */
/* OK, every break has additional indent */
switch (check()) {
    case 0:
        do_a();
        break;
    case 1:
        do_b();
        break;
    default:
        break;
}

/* Wrong, case indent missing */
switch (check()) {
case 0:
    do_a();
    break;
case 1:
    do_b();
    break;
default:
    break;
}

/* Wrong */
switch (check()) {
    case 0:
        do_a();
    break;      /* Wrong, break must have indent as it is under case */
    case 1:
    do_b();     /* Wrong, indent under case is missing */
    break;
    default:
        break;
}
```

- Always include `default` statement
```c
/* OK */
switch (var) {
    case 0:
        do_job();
        break;
    default:
        break;
}

/* Wrong, default is missing */
switch (var) {
    case 0:
        do_job();
        break;
}
```

- If local variables are required, use curly brackets and put `break` statement inside.
    - Put opening curly bracket in the same line as `case` statement
```c
switch (a) {
    /* OK */
    case 0: {
        int32_t a, b;
        char c;
        a = 5;
        /* ... */
        break;
    }

    /* Wrong */
    case 1:
    {
        int32_t a;
        break;
    }

    /* Wrong, break shall be inside */
    case 2: {
        int32_t a;
    }
    break;
}
```

## Macros and preprocessor directives

- Always use macros instead of literal constants, specially for numbers
- All macros must be fully uppercase, with optional underscore `_` character, except if they are clearly marked as function which may be in the future replaced with regular function syntax
```c
/* OK */
#define MY_MACRO(x)         ((x) * (x))

/* Wrong */
#define square(x)           ((x) * (x))
```

- Always protect input parameters with parentheses
```c
/* OK */
#define MIN(x, y)           ((x) < (y) ? (x) : (y))

/* Wrong */
#define MIN(x, y)           x < y ? x : y
```

- Always protect final macro evaluation with parenthesis
```c
/* Wrong */
#define MIN(x, y)           (x) < (y) ? (x) : (y)
#define SUM(x, y)           (x) + (y)

/* Imagine result of this equation using wrong SUM implementation */
int32_t x = 5 * SUM(3, 4);  /* Expected result is 5 * 7 = 35 */
int32_t x = 5 * (3) + (4);  /* It is evaluated to this, final result = 19 which is not what we expect */

/* Correct implementation */
#define MIN(x, y)           ((x) < (y) ? (x) : (y))
#define SUM(x, y)           ((x) + (y))
```

- When macro uses multiple statements, protect it using `do-while (0)` statement
```c
typedef struct {
    int32_t px, py;
} point_t;
point_t p;                  /* Define new point */

/* Wrong implementation */

/* Define macro to set point */
#define SET_POINT(p, x, y)  (p)->px = (x); (p)->py = (y)    /* 2 statements. Last one should not implement semicolon */

SET_POINT(&p, 3, 4);        /* Set point to position 3, 4. This evaluates to... */
(&p)->px = (3); (&p)->py = (4); /* ... to this. In this example this is not a problem. */

/* Consider this ugly code, however it is valid by C standard (not recommended) */
if (a)                      /* If a is true */
    if (b)                  /* If b is true */
        SET_POINT(&p, 3, 4);/* Set point to x = 3, y = 4 */
    else
        SET_POINT(&p, 5, 6);/* Set point to x = 5, y = 6 */

/* Evaluates to code below. Do you see the problem? */
if (a)
    if (b)
        (&p)->px = (3); (&p)->py = (4);
    else
        (&p)->px = (5); (&p)->py = (6);

/* Or if we rewrite it a little */
if (a)
    if (b)
        (&p)->px = (3);
        (&p)->py = (4);
    else
        (&p)->px = (5);
        (&p)->py = (6);

/*
 * Ask yourself a question: To which `if` statement `else` keyword belongs?
 *
 * Based on first part of code, answer is straight-forward. To inner `if` statement when we check `b` condition
 * Actual answer: Compilation error as `else` belongs nowhere
 */

/* Better and correct implementation of macro */
#define SET_POINT(p, x, y)  do { (p)->px = (x); (p)->py = (y); } while (0)    /* 2 statements. No semicolon after while loop */
/* Or even better */
#define SET_POINT(p, x, y)  do {    \   /* Backslash indicates statement continues in new line */
    (p)->px = (x);                  \
    (p)->py = (y);                  \
} while (0)                             /* 2 statements. No semicolon after while loop */

/* Now original code evaluates to */
if (a)
    if (b)
        do { (&p)->px = (3); (&p)->py = (4); } while (0);
    else
        do { (&p)->px = (5); (&p)->py = (6); } while (0);

/* Every part of `if` or `else` contains only `1` inner statement (do-while), hence this is valid evaluation */

/* To make code perfect, use brackets for every if-ifelse-else statements */
if (a) {                    /* If a is true */
    if (b) {                /* If b is true */
        SET_POINT(&p, 3, 4);/* Set point to x = 3, y = 4 */
    } else {
        SET_POINT(&p, 5, 6);/* Set point to x = 5, y = 6 */
    }
}
```

- Avoid using `#ifdef` or `#ifndef`. Use `defined()` or `!defined()` instead
```c
#ifdef XYZ
/* do something */
#endif /* XYZ */
```

- Always document `if/elif/else/endif` statements
```c
/* OK */
#if defined(XYZ)
/* Do if XYZ defined */
#else /* defined(XYZ) */
/* Do if XYZ not defined */
#endif /* !defined(XYZ) */

/* Wrong */
#if defined(XYZ)
/* Do if XYZ defined */
#else
/* Do if XYZ not defined */
#endif
```

- Do not indent sub statements inside `#if` statement
```c
/* OK */
#if defined(XYZ)
#if defined(ABC)
/* do when ABC defined */
#endif /* defined(ABC) */
#else /* defined(XYZ) */
/* Do when XYZ not defined */
#endif /* !defined(XYZ) */

/* Wrong */
#if defined(XYZ)
    #if defined(ABC)
        /* do when ABC defined */
    #endif /* defined(ABC) */
#else /* defined(XYZ) */
    /* Do when XYZ not defined */
#endif /* !defined(XYZ) */
```

## Документация

Документирование doxygen переобразуется в html/pdf/latex, поэтому очень важно сделать это правильно (Но чаще всего всем всё равно на это. Вы не будете читать кривущие html странички. Проще и быстрее сразу приступить к изучению кода, но для этого тоже надо правильно задокументировать). 

- Используйте doxygen-документацию для variables, functions и structures/enumerations.
- Всегда используйте `\` для doxygen, не используйте `@`. (я счита. это правило не жестким и использую `@` часто. Это хорошо совпадает с документированием стандартных бибилиотек и не выбивается из стиля).
- Всегда используйте `5x4` пробела (`5` табов) для отступа для написания текста.
```c
/**
 * \brief           Текст должен начинаться здесь.
 *                
 */                    
static
type_t* list;
```

- Каждая структура/энумератор должен быть описан в документации.
- Используйте `12x4 отступ` для начала комментаирования (это правило также не советую жестко выполнять. Длинна строчки кода может быть различной. Тут больше советую смотреть на выравнивание стоблца комментариев, чем на правильный "отступ").
```c
/**
 * \brief           Это структура описывающая точку.
 * \note            Эта структура используется для расчета всех вещей, связанных с точками.
 *                      
 */
typedef struct {
    int32_t x;      /*!< X координата точки */
    int32_t y;      /*!< Y координата точки */
    int32_t size;   /*!< Размер точки.
                         Если комментарий слишком большой, 
                         вы можете перейти на новую строчку. */
} point_t;

/**
 * \brief           Энумератор цвета точки.
 */
typedef enum {
    COLOR_RED,      /*!< Red color. This comment has 12x4
                         spaces offset from beginning of line */
    COLOR_GREEN,    /*!< Green color */
    COLOR_BLUE,     /*!< Blue color */
} point_color_t;
```

- Описание функции должно содержать `brief` и все остальные необзодимые ключевые слова документации.
- Кадый параметр должен быть помечен `in` or `out` для *input* и *output* соответсвенно.
- Функция обязательно должна иметь описание возвращаемого объекта. Это можно упустить для функций типа `void`.
- Может содержать дополнительные ключевые слова, такие как `note` или `warning`.
- Используйте двоеточие `:` между именем параметра и его описанием.
- ВАЖНО!!! Здесь я предлагаю отойти от обычного объявления и определения функций. Описание самой функции предлагаю делать в `.h` файле. Во-первых, современные IDE позволяют при наведении на функцию увидеть её описание, а во-вторых, это помогает сократить код. При определении функций в `.c` файл может спокойно уйти за несколько сотен или тысяч строк кода. Таким образом нам приходится проматывать огромные полотнища кода туда-сюда. К заголовочникам мы обращаемся реже и они как правило намного компактнее. Поэтому, чтобы сократить количество строк кода и улучшить восприятие больших файлов, лучше использовать данный подход.
```c
/* file.h ... */
/**
 * \brief           Сумма `2` чисел
 * \param[in]       a: Первое число
 * \param[in]       b: Второе число
 * \return          Сумма `2` переданных чисел
 */
int32_t sum (int32_t a, int32_t b);


/* file.c ... */
int32_t sum (int32_t a, int32_t b) {
    return a + b;
}

/**
 * \brief           Возвращает сумму 2 чисел через указатель.
 * \note            Эта функция не возвращает число, она сохраняет его в указатель.
 * \param[in]       a: Первое число.
 * \param[in]       b: Второе число.
 * \param[out]      result: Выходная переменная, которая используется для получения результата из функции.
 */
void void_sum (int32_t a, int32_t b, int32_t* result) {
    *result = a + b;
}
```

- Если функция возвращает член перечисления, используйте ключевое слово `ref`, чтобы указать какое из них.
```c
/**
 * \brief           Мой энумератор.
 */
typedef enum {
    MY_ERR,         /*!< Ошибочное значение. */
    MY_OK           /*!< Действующее значение. */
} my_enum_t;

/**
 * \brief           Проверяет какое-нибудь значение.
 * \return          \ref MY_OK если удачно, иначе \ref my_enum_t
 */
my_enum_t check_value (void) {
    return MY_OK;
}
```

- Используйте нотацию (\`NULL\` => `NULL`) для констант или чисел.
```c
/**
 * \brief           Получить данные из входного массива.
 * \param[in]       in: Входные данны.
 * \return          Указатель для вывода данных в случае успеха, `NULL` в противном случае.
 */
const void* get_data (const void* in) {
    return in;
}
```

- Документация для макросов должна включать команду doxygen `hideinitializer`:
```c
/**
 * \brief           Получить минимальное значение между `x` и `y`.
 * \param[in]       x: Первое значение.
 * \param[in]       y: Второе значение.
 * \return          Минимальное значение из `x` и `y`
 * \hideinitializer
 */
#define MIN (x, y)       ((x) < (y) ? (x) : (y))
```

## Заголовочные и исходного кода файлы

- Всегда в начале файла используйте `file` and `brief` для описания самого файла. После описания должна быть одна пустая строка.

```c
/**
 * \file            template.h
 * \brief           Описание файла.
 */
                    /* Здесь пустая строка */
```

- Также следует оставлять одну пустую строку в конце файла.

- Каждый файл (*header* or *source*) должен включать лицензию (открывающий комментарий содержит один символ звезды)
- Используйте ту же лицензию, которая уже используется проектом/библиотекой.
```c
/**
 * \file            template.h
 * \brief           Описание файла.
 */

/*
 * Copyright (c) year FirstName LASTNAME
 *
 * Permission is hereby granted, free of charge, to any person
 * obtaining a copy of this software and associated documentation
 * files (the "Software"), to deal in the Software without restriction,
 * including without limitation the rights to use, copy, modify, merge,
 * publish, distribute, sublicense, and/or sell copies of the Software,
 * and to permit persons to whom the Software is furnished to do so,
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be
 * included in all copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
 * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
 * OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE
 * AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
 * HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
 * WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
 * FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
 * OTHER DEALINGS IN THE SOFTWARE.
 *
 * This file is part of library_name.
 *
 * Author:          FirstName LASTNAME <optional_email@example.com>
 */
```

- Прежде всего файл должен содержать в самом начале `#ifndef`.
- Внешние заголовочные файлы бибилиотеки STL C всегда должны предшествовать пользовательским заголовочным файлам.
- Заголовочный файл должен подключать только те заголовочные файлы, которые необходимы для правильной компиляции, но не более того! (на файлы с расширением .c это не распространяется).
- Заголовочный файл должен содержать только общедоступные переменные/типы/функции модуля. Все вспомогательные функции реализуются в `.c`.
- Используйте `extern` для глобальных переменных модуля в заголовочном файле, определите их в исходном файле позже. Для инкапсулированных внутри одного файла используйте `static`.
```
/* file.h ... */
#ifndef ...

extern int32_t my_variable; /* Глобальная переменная, объявленная в заголовочном файле */

#endif

/* file.c ... */
int32_t my_variable;        /* Опережедение в исходном файле */
```
- Никогда не подключайте `.c` файлы в другие `.c` файлы.
- Файл `.c` должен сначала включать соответствующий ему `.h` файл, а затем другие файлы, если иное не сломает вашу программу (старайтесь избегать таких ситуаций изменяя архитектуру программы).
- Не включайте приватные объявления модуля `static` в заголовочный файл.

- Пример заголовочного файла (без лицензии для примера):
```c
/* License comes here */
#ifndef TEMPLATE_HDR_H
#define TEMPLATE_HDR_H

/*******************************************************************************
* INCLUDE FILES:
*******************************************************************************/

/* Подключите заголовочные файлы здесь. */

/*******************************************************************************
* CONSTANTS & MACROS:
*******************************************************************************/

/* Определите константы и макросы здесь. */

/*******************************************************************************
* TYPES:
*******************************************************************************/

/* Определите пользовательские типы здесь. */

/*******************************************************************************
* GLOBAL VARIABLES:
*******************************************************************************/

/* Определите глобальные переменные здесь. */

/*******************************************************************************
* FUNCTIONS:
*******************************************************************************/

/* Определите прототипы функций здесь. */

#endif /* TEMPLATE_HDR_H */
```

## Стиль наименования файлов

Чтобы гарантировать безпроблемное чтение файлов и путей, независимо от операционной системы или инструментов сборки, именуйте файлы строчными буквами, без всяких - и пробелов, т.е. желательно все в одино слово. Если не получается назвать файл в одно слово, то используйте нижнее подчеркивание `_`. Этого же следует придерживаться при наименовании папок в которых лежат собираемые файлы.

```c
syscalls.c          // Правильно
sys_calls.c         // Правильно
sys calls.c         // Не правильно
SysCalls.c          // Не правильно
```

- !!!Внимание!!! Я лично рекомендую всё-таки местами отходить от этих правил только в одном случае, которое не несёт никакой угрозы для сборки проекта, но может облегчить жизнь. Так бывает, что мы используем бибилиотеки кем-то написанные до нас. Особенно в большом проекте потом легко запутаться где чей функционал. Что сожно и следует менять, а куда лучше не лезть, рискуя слмоать логику бибилоитеки. Поэтому для своих файлов рекомендую использовать верхний регистр:

```c
MY_FILE1.h
MY_FILE1.c
MY_FILE2.h            
MY_FILE2.c         
и т.д. 
```