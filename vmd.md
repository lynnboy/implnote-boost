# Boost.VMD

* lib: `boost/libs/vmd`
* repo: `boostorg/vmd`
* commit: `26b39eb`, 2025-05-03

------
### Generic

#### Data types

* identifier: `anyname`
* number: `47`
* type: `BOOST_VMD_TYPE_NUMBER`
* array: `(4,(ident,145,TYPE_IDENTIFIER))`
* list: `(78,(ident,(TYPE_TYPE,PP_NIL)))`
* seq: `(ident)(89)(245)`
* tuple: `(id,175,TYPE_LIST,hanny,21)`
* sequence: `tree 59 (56,TYPE_SEQ)(128)(fire)(clown)(47,(TYPE_TUPLE,PP_NIL))`

#### Functional groups

* Emptiness
* Identifiers
* Numbers
* Types
* Boost PP data types (array, list, seq, tuple)
* Sequences
* Helpers

------
### Macros

* `ARRAY_TO_SEQ(array)`, `ARRAY_TO_TUPLE(array)`
* `ASSERT(...)`, `ASSERT_IS_ARRAY(seq)`, `ASSERT_IS_EMPTY(...)`, `ASSERT_IS_IDENTIFIER(...)`,
    `ASSERT_IS_LIST(seq)`, `ASSERt_IS_NUMBER(seq)`, `ASSERT_IS_SEQ(seq)`, `ASSERT_IS_TUPLE(seq)`, `ASSERT_IS_TYPE(seq)`
* `ELEM(elem,...)`
* `EMPTY(...)`
* `ENUM(...)`
* `EQUAL(seq,...)`, `NOT_EQUAL(seq,...)`
* `GET_TYPE(...)`
* `IDENTITY(item)`
* `IS_ARRAY(seq)`, `IS_EMPTY(...)`, `IS_EMPTY_ARRAY(seq)`, `IS_EMPTY_LIST(seq)`, `IS_GENERAL_IDENTIFIER(...)`,
    `IS_IDENTIFIER(...)`, `IS_LIST(seq)`, `IS_MULTI(seq)`, `IS_NUMBER(seq)`, `IS_PARENS_EMPTY(seq)`,
    `IS_SEQ(seq)`, `IS_TUPLE(seq)`, `IS_TYPE(seq)`, `IS_UNARY(seq)`
* `LIST_TO_SEQ(list)`, `LIST_TO_TUPLE(list)`
* `IS_VMD_SEQ(seq)`, `SEQ_POP_BACK(seq)`, `SEQ_POP_FRONT(seq)`, `SEQ_PUSH_BACK(seq,elem)`, `SEQ_PUSH_FRONT(seq,elem)`,
    `SEQ_REMOVE(seq,index)`, `SEQ_SIZE(seq)`, `SEQ_TO_ARRAY(seq)`, `SEQ_TO_LIST(seq)`, `SEQ_TO_TUPLE(seq)`
* `SIZE(seq)`
* `TO_ARRAY(...)`, `TO_LIST(...)`, `TO_SEQ(...)`, `TO_TUPLE(...)`
* `IS_VMD_TUPLE(seq)`, `TUPLE_POP_BACK(tuple)`, `TUPLE_POP_FRONT(tuple)`, `TUPLE_PUSH_BACK(tuple,elem)`, `TUPLE_PUSH_FRONT(tuple,elem)`,
    `TUPLE_REMOVE(tuple,index)`, `TUPLE_SIZE(tuple)`, `TUPLE_TO_ARRAY(tuple)`, `TUPLE_TO_LIST(tuple)`, `TUPLE_TO_TUPLE(tuple)`

------
### Dependency

#### Boost.Preprocessor

* `<boost/preprocessor/**.hpp>`

------
### Standard Facilities
