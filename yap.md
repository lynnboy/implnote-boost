# Boost.YAP

* lib: `boost/libs/yap`
* repo: `boostorg/yap`
* commit: `2d44666`, 2025-03-14

------
### Expression Template Algorithms

* Header `<boost/yap/algorithm.hpp>`

```c++
enum class expr_kind { expr_ref, terminal,
    unary_plus, ... // unary: +, -, *, ~, &, !, ++, --, ++(int), --(int)
    shift_left, ... // binary: <<, >>, *, /, %, +, -, <, >, <=, >=, ==, !=, ||, &&, &, |, ^, ,, ->*, =, <<=, >>=, *=, /=, %=, +=, -=, &=, |=, ^=, []
    if_else, call // ?: and ()
};
struct placeholder<i> : hana::long<l>{};
struct expression<kind,Tuple>;
struct is_expr<Expr> = requires(Expr e) {Expr::kind -> expr_kind; e.elements;} && hana::is_a<hana::tuple_tag, decltype(e.elements)>;
using terminal<exprT<kind,Tup>, T> = exprT<expr_kind::terminal,hana::tuple<T>>;
using expression_ref<exprT<kind,Tup>, T> = exprT<expr_kind::expr_ref, hana::tuple<remove_reference_t<T>*>>;

constexpr decltype(auto) evaluate <Expr,...T> (Expr&& e, T&&...t);
constexpr decltype(auto) transform <Expr,Transform,...Transforms> (Expr&& e, Transform&& tr, Transforms&&... trs);
constexpr decltype(auto) transform_strict <Expr,Transform,...Transforms> (Expr&& e, Transform&& tr, Transforms&&... trs);
constexpr decltype(auto) deref <Expr> (Expr&& x)
{ if constexpr (/*Expr is T&&*/) return std::move(*expr.elements[0_c]); else return *expr.elements[0_c];}
constexpr decltype(auto) value <Expr,termOnly=false> (Expr&& e) {
    if constexpr (is_expr<Expr>::value) { constexpr kind = remove_cv_ref_t<Expr>::kind;
        if constexpr (kind == expr_ref) return value<termOnly>(deref((Expr&&)(x))); // forward deref of x
        else if constexpr (kind == terminal || (!termOnly && arity_of<kind>() == one))
        { if constexpr (/*Expr is T& or elements[0] is T&*/) return expr.elements[0_c] else return std::move(expr.elements[0_c]); }
        else return (T&&)(x); // forward
    } else return (T&&)(x); // forward
}
constexpr auto literals::operator""_p <...c>() // postfix _p for placeholder
{ long long i = /*parse c...*/; return expression<expr_kind::terminal,hana::tuple<placeholder<i>>>{}; }

struct expr_tag<kind> { static const expr_kind kind = kind; };
struct minimal_expr<kind,Tuple> { static const expr_kind kind = kind; Tuple elemens; };

struct detail::static_const<T> { static constexpr T value{}; };
struct detail::partial_decay<T> { using type = ...; }; // std::decay<T>
using detail::operand_value_type_phrase_1<T,U=partial_decay<T>::type>::type
    = (is_same_v<T,U> && !is_const_v<U>) ? U&& : U;
struct detail::expr_ref<ExprT<k,Tup>,T> { using type = expression_ref<ExprT,T>; }; // primary
struct detail::expr_ref<ExprT,ExprT<expr_ref,Tup>[const]&> { using type = ExprT<expr_ref,Tup>; };
using detail::expr_ref_t<ExprT<k,Tup>,T> = expr_ref<ExprT,T>::type;
struct detail::expr_ref_tuple<ExprT<k,Tup>,T>; // primary
struct detail::expr_ref_tuple<ExprT,ExprT<expr_ref,Tup>> { using type = Tup; };
using detail::expr_ref_tuple_t<ExprT<k,Tup>,T> = expr_ref_tuple<ExrT,T>::tupe;
struct detail::operand_type<ExprT<k,Tup>,T>; // primary
struct detail::operand_type<Expr,T> { using U = operand_value_type_phrase_1<T>::type;
    using type = is_expr<T> ?
        is_lvalue_reference_v<T> ? expr_ref_t<ExprT,T> : remove_cv_ref_t<T>
      : is_rvalue_reference_v<U> ? terminal<ExprT,remove_reference_t<U>> : terminal<ExprT,U>;
};
using detail::operand_type_t<ExprT<k,Tup>,T> = operand_type<ExprT,T>::type;
struct detail::make_operand<T> { constexpr auto operator() <U> (U&& u) { return T{(U&&)u}; } };
struct detail::make_operand<ExprT<expr_ref,Tup>> { // wrap value's ptr into expr_ref
    constexpr auto operator() (ExprT<expr_ref,Tup>&& expr) { return expr; }
    constexpr auto operator() <U> (U&& u) { return ExprT<expr_ref,Tup>{Tup{ addressof(u) }}; }
};

struct detail::free_binary_op_result<ExprT<k,Tup>,k,T,U>;
using detail::free_binary_op_result_t<ExprT<k,Tup>,k,T,U> = free_binary_op_result<ExprT,k,T,U>::type;
struct detail::ternary_op_result<ExprT<k,Tup>,T,U,V>;
using detail::ternary_op_result_t<ExprT<k,Tup>,T,U,V> = ternary_op_result<Expr,T,U,V>::type;

struct detail::is_udt_arg<T,Tr<T>> { static const bool value = !is_expr<T>::value && Tr<remove_cv_ref_t<T>>::value; };
struct detail::udt_any_ternary_op_result<ExprT<k,Tup>,T,U,V,Tr<T>>;
using detail::udt_any_ternary_op_result_t<ExprT<k,Tup>,T,U,V,Tr<T>> = udt_any_ternary_op_result<Expr,T,U,V,Tr>::type;
struct detail::udt_unary_op_result<ExprT<k,Tup>,k,T,Tr<T>>;
using detail::udt_unary_op_result_t<ExprT<k,Tup>,k,T,Tr<T>> = udt_unary_op_result<Expr,k,T,Tr>::type;
struct detail::udt_udt_binary_op_result<ExprT<k,Tup>,k,T,U,TrT<T>,TrU<U>>;
using detail::udt_udt_binary_op_result_t<ExprT<k,Tup>,k,T,U,TrT<T>,TrU<U>> = udt_udt_binary_op_result<Expr,k,T,U,TrT,TrU>::type;
struct detail::udt_any_binary_op_result<ExprT<k,Tup>,k,T,U,TrT<T>>;
using detail::udt_any_binary_op_result_t<ExprT<k,Tup>,k,T,U,TrT<T>> = udt_any_binary_op_result<Expr,k,T,U,TrT>::type;

struct detail::copy_or_move<T,U> : is_same<U,T{const&|&|&&}> {};

enum class detail::expr_arity { invalid, one, two, three, n }; // unary, binary, if_else, callable
constexpr expr_arity detail::arity_of<kind>() { switch (kind) ...; }

constexpr decltype(auto) get <Expr,I> get(Expr&& e, I const& i) {
    if constexpr (kind == expr_ref) return get(deref((Expr&&)(e)), i);
    else { if constexpr (ls_lvalue_reference_v<Expr>) return expr.elements[i]; else return std::move(expr.elements[i]); }
}
constexpr decltype(auto) get_c <i,Expr> (Expr && e) { return get((Expr&&)(e), hana::llong_c<i>); }
constexpr decltype(auto) left <Expr> (Expr&& e) { return get((Expr&&)(e), 0_c); } // arity_of<kind>() == two;
constexpr decltype(auto) right <Expr> (Expr&& e) { return get((Expr&&)(e), 1_c); } // arity_of<kind>() == two;
constexpr decltype(auto) cond <Expr> (Expr&& e) { return get((Expr&&)(e), 0_c); } // kind == expr_ref || kind == if_else
constexpr decltype(auto) then <Expr> (Expr&& e) { return get((Expr&&)(e), 1_c); } // kind == expr_ref || kind == if_else
constexpr decltype(auto) else_ <Expr> (Expr&& e) { return get((Expr&&)(e), 1_c); } // kind == expr_ref || kind == if_else
constexpr decltype(auto) callable <Expr> (Expr&& e) { return get((Expr&&)(e), 0_c); } // kind == expr_ref || arity_of<kind>() == n;
constexpr decltype(auto) argument <i,Expr> (Expr&& e, hana::llong<i>) { return get((Expr&&)(e), hana::long_c<i+1>); } // kind == expr_ref || arity_of<kind>() == n;

constexpr auto make_expression <ExprT<kind,Tup>,kind,...T> (T&&...t) { // arity_of<kind> matches sizeof...T
    using tuple_type = hana::tuple<operand_type_t<ExprT,T>...>;
    return ExprT<kind,tuple_type>{tuple_type{ make_operand<operand_type_t<ExprT,Tup>>{}((T&&)(t)...) }};
}
constexpr auto make_terminal <ExprT<kind,Tup>,T> (T&& t) {
    using result_type = operand_type_t<ExprT,T>;
    using tuple_type = decltype(declval<result_type>().elements);
    return result_type{tuple_type{(T&&)(t)}};
}
constexpr decltype(auto) as_expr <ExprT<kind,Tup>,T> (T&& t)
{ if constexpr (is_expr<T>::value) return (T&&)t; else return make_terminal<ExprT>((T&&)t); }

struct expression_function<Expr> { Expr expr;
    constexpr decltype(auto) operator() <...U> (U&&...u) { return evaluate(expr,(U&&)u...); }
};
using detail::expression_function_expr<k,Tup> = minimal_expr<k,Tup>;
constexpr auto make_expression_function <Expr> (Expr&& expr) {
    using stored_type = operand_type_t<expression_function_expr, Expr&&>;
    return expression_function<stored_type>{ make_operand<stored_type>{}((Expr&&)expr) };
}

using detail::nth_element<i,...Ts> = Ts...[i];
struct detail::rvalue_ref_to_value<T> { using type = is_rvalue_reference_v<T> ? remove_reference_t<T> : T; }
using detail::rvalue_ref_to_value_t<T> = rvalue_ref_to_value<T>::type;
struct detail::rvalue_mover<isRref> { constexpr decltype(auto) operator() <T> (T&& t) { return (T&&)(t); } };
struct detail::rvalue_mover<true> { constexpr remove_reference_t<T> operator() <T> (T&& t) { return std::move(t); } };

struct detail::placeholder_transform_t<...PArgs> {
    hana::tuple<rvalue_ref_to_value_t<PArgs>...> placeholder_args_; // wrapped args
    constexpr ctor(PArgs&&... args); // move-in init
    constexpr decltype(auto) operator() <i> (expr_tag<terminal>, placeholder<i>) const
    { return as_expr<minimal_expr>(rvalue_mover<!is_lvalue_reference_v<PArgs...[i-1]>>{}( placeholder_args_[hana::llong<i-1>{}] )); }
};
struct detail::evaluation_transform_t<...PArgs> {
    hana::tuple<rvalue_ref_to_value_t<PArgs>...> placeholder_args_; // wrapped args
    constexpr ctor(PArgs&&... args); // move-in init
    constexpr decltype(auto) operator() <i> (expr_tag<terminal>, placeholder<i>) const
    { return rvalue_mover<!is_lvalue_reference_v<PArgs...[i-1]>>{}( placeholder_args_[hana::llong<i-1>{}] ); }
    constexpr decltype(auto) operator() <T> (expr_tag<terminal>, T&& t) const { return (T&&)(t); }

    constexpr decltype(auto) operator() <T> (expr_tag<unary_plus>, T&& t) const
    { return + transform(as_expr<minimal_expr>((T&&)(t)), *this); }
    // all unary operators: +, -, *, ~, &, !, ++, --, post++, post--
    constexpr decltype(auto) operator() <T,U> (expr_tag<left_shift>, T&& t, U&& u) const
    { return transform(as_expr<minimal_expr>((T&&)(t)), *this) << transform(as_expr<minimal_expr>((U&&)(u)), *this); }
    // all binary operators: >>, *, /, %, +, -, <, >, <=, >=, ==, !=, ||, &&, &, |, ^, ,, ->*, =, <<=, >>=, *=, /=, %=, +=, -=, &=, |=, ^=, []
    constexpr decltype(auto) operator() <T,U,V> (expr_tag<if_else>, T&& t, U&& u, V&& v) const
    { return transform(as_expr<minimal_expr>((T&&)(t)), *this) ? transform(as_expr<minimal_expr>((U&&)(u)), *this) : transform(as_expr<minimal_expr>((V&&)(v)), *this); }
    constexpr decltype(auto) operator() <Callable,...Args> (expr_tag<call>, Callable&& c, Args&&...args) const
    { return transform(as_expr<minimal_expr>((Callable&&)(c)), *this) ( transform(as_expr<minimal_expr>((Args&&)(args)), *this) ...); }
};

struct detail::transform_impl<strict,i,isExprRef>;
constexpr auto detail::make_expr_from_tuple<k,ExprT<k,T>,OldTup,NewTup> (ExprT<k,OldTup> const& expr, NewTup&& tup) { return ExprT<k,NewTup>(std::move(tup)); }
constexpr auto detail::make_expr_from_tuple<k,Expr,NewTup> (Expr const& expr, NewTup&& tup) { return minimal_expr<k,NewTup>(std::move(tup)); }
constexpr decltype(auto) detail::transform_nonterminal<Expr,Tup,TxTup> (Expr const& e, Tup&& tup, TxUp txs) {
    auto tx_tup = hana::transform((Tup&&)tup, [&](auto&& ele) {
        using element_t = decltype(ele); auto const kind = remove_cv_ref_t<element_t>::kind;
        return transform_impl<false,0,kind==expr_ref>{}((element_t&&)ele, txs);
    })
    return make_expr_from_tuple<remove_cv_ref_t<Expr>::kind>(expr, std::move(tx_tup));
}
struct detail::default_transform<isLref,isTerm,strict> {
    constexpr decltype(auto) operator() <Expr,TxTup> (Expr&& expr, TxTup txs) const {
        if constexpr (!isTerm && !strict) {
            if constexpr (isLref) return transform_nonterminal(expr, expr.elements, txs);
            else return transform_nonterminal(expr, std::move(expr.elements), txs);
        } else if constexpr (strict) static_assert(false);
        else return (Expr&&)expr;
    }
};
struct detail::next_or_default_transform<strict,Expr,TxTup,i,hasNext> {
    constexpr decltype(auto) operator() (Expr&& expr, TxTup txs) const {
        if constexpr (hasNext) return transform_impl<strict,i+1,remove_cv_ref_t<Expr>::kind==expr_ref>{} ((Expr&&)expr, txs);
        else return default_transform<is_lvalue_reference_v<Expr>, remove_cv_ref_t<Expr>::kind==terminal, strict>{} ((Expr&&)expr, txs);
    }
};
struct transform_expression_expr<strict,Expr,TxTup,i> {
    constexpr decltype(auto) operator() (Expr&& expr, TxTup txs) const {
        if constexpr (requires{...}) return (*txs[hana::llong<i>{}])((Expr&&)expr);
        else return next_or_default_transform<strict,Expr,TxTup,i,(i+1<hana::size(txs)::value)> ((Expr&&)expr, txs);
    }
};
constexpr decltype(auto) detail::terminal_value<T> (T&& x) { return value<T,true>((T&&)x); }
struct detail::transform_call_unpacker <Expr,Tx> {
    constexpr auto operator() <...i> (Expr&& expr, Tx& tx, integer_sequence<long long,i...>) const -> ...
    { return tx(expr_tag<call>{}, terminal_value(get((Expr&&)expr, hana::llong_c<i>))... ); }
};
constexpr auto indices_for<Expr> (Expr& const& expr) { return make_integer_sequence<long long, hana::size(expr.elements).value>(); }
struct detail::transform_expression_tag<strict,Expr,Tup,i,arity> {
    constexpr decltype(auto) operator() (Expr&& expr, TxTup txs) const {
        using I = hana::llong<i>; auto const kind = remove_cv_ref_t<Expr>::kind;
        if constexpr (arity == one && requires{...})
            return (*txs[I{}])( expr_tag<kind>{}, terminal_value(value((Expr&&)expr)) );
        if constexpr (arity == two && requires{...})
            return (*txs[I{}])( expr_tag<kind>{}, terminal_value(left((Expr&&)expr)), terminal_value(right((Expr&&)expr)) );
        if constexpr (arity == three && requires{...})
            return (*txs[I{}])( expr_tag<kind>{}, terminal_value(cond((Expr&&)expr)), terminal_value(then((Expr&&)expr)), terminal_value(else_((Expr&&)expr)) );
        if constexpr (arity == n && requires{...})
            return transform_call_unpacker<Expr,decltype(*txs[I{}])>( (Expr&&)expr, *trx[I{}], indices_for(expr) );
        return transform_expression_expr<strict,Expr,Tup,i>{}( (Expr&&)(expr), tx );
    }
};
struct detail::transform_impl<strict,i,isExprRef> {
    constexpr decltype(auto) operator() <Expr,Tup> (Expr&& expr, TxTup txs) const {
        if constexpr (isExprRef) return transform_impl<strict,i,false>{}( deref((Expr&&)expr, txs) );
        auto const arity = arity_of<remove_cv_ref_t<Expr>::kind>();
        return transform_expression_tag<strict,Expr,Tup,i,arity>( (Expr&&)expr, txs );
    }
};

constexpr auto replacements <...T> (T&&...t) { return placeholder_transform_t<T...>((T&&)t...); }
constexpr decltype(auto) replace_placeholders <Expr,...T> (Expr&& expr, T&&...t) { return transform((Expr&&)expr, replacements((T&&)t...)); }
constexpr auto evaluation <...T> (T&&...t) { return evaluation_transform_t<T...>((T&&)t...); }
constexpr decltype(auto) evaluate<Expr,...T> (Expr&& expr, T&&...t) { return transform( (Expr&&)expr, evaluation((T&&)t...) ); }
constexpr auto detail::make_transform_tuple<...Tx>(Tx&...tx) { return hana::tuple<Tx*...>{&tx...}; }
struct detail::transform_<struct> {
    constexpr decltype(auto) operator() <Expr,Tx,...Txs> (Expr&& expr, Tx& tx, Txs&...txs) const
    {   auto txtup = make_transform_tuple(tx,txs...); auto kind = remove_cv_ref_t<Expr>::kind;
        return transform_impl<strict,0,kind==expr_ref>{}( (Expr&&)expr, txtup );
    }
};
constexpr decltype(auto) transform <Expr,Tx,...Txs> (Expr&& expr, Tx&& tx, Txs&&...txs)
{ return transform_<false>{}( (Expr&&)expr, tx, txs... ); }
constexpr decltype(auto) transform_strict <Expr,Tx,...Txs> (Expr&& expr, Tx&& tx, Txs&&...txs)
{ return transform_<true>{}( (Expr&&)expr, tx, txs... ); }
```

### Oprator Defining Macros

```c++
#define BOOST_YAP_USER_UNARY_OPERATOR(op_name, exprT, res_exprT) ...
#define BOOST_YAP_USER_BINARY_OPERATOR(op_name, exprT, res_exprT) ...
#define BOOST_YAP_USER_ASSIGN_OPERATOR(self, exprT) ...
#define BOOST_YAP_USER_SUBSCRIPT_OPERATOR(exprT) ...
#define BOOST_YAP_USER_CALL_OPERATOR(exprT) ...
#define BOOST_YAP_USER_CALL_OPERATOR_N(exprT, n) ...
#define BOOST_YAP_USER_EXPR_IF_ELSE(exprT) ...
#define BOOST_YAP_USER_UDT_ANY_IF_ELSE(exprT,trait) ...
#define BOOST_YAP_USER_UDT_UNARY_OPERATOR(op_name, exprT, trait) ...
#define BOOST_YAP_USER_UDT_UDT_BINARY_OPERATOR(op_name, exprT, trait1, trait2) ...
#define BOOST_YAP_USER_UDT_ANY_BINARY_OPERATOR(op_name, exprT, trait) ...
#define BOOST_YAP_USER_LITERAL_PLACEHOLDER_OPERATOR(exprT) ...
```

* `BOOST_YAP_USER_UNARY_OPERATOR` define 3 non-member overloads with `const&`, `&`, `&&` args.
* `BOOST_YAP_USER_BINARY_OPERATOR` define 6 non-member functions with forward and inversed arg order.
* `BOOST_YAP_USER_ASSIGN_OPERATOR` define 3 member overloads.
* `BOOST_YAP_USER_SUBSCRIPT_OPERATOR` define 3 member overloads.
* `BOOST_YAP_USER_CALL_OPERATOR` define 3 variadic member overloads.
* `BOOST_YAP_USER_CALL_OPERATOR_N` define 3 non-variadic member overloads with n args.
* `BOOST_YAP_USER_EXPR_IF_ELSE` define 1 non-member function.
* `BOOST_YAP_USER_UDT_ANY_IF_ELSE` define non-member function with a trait.
* `BOOST_YAP_USER_UDT_UNARY_OPERATOR` define non-member function with trait.
* `BOOST_YAP_USER_UDT_UDT_BINARY_OPERATOR` define non-member function with a trait for each arg.
* `BOOST_YAP_USER_UDT_ANY_BINARY_OPERATOR` define non-member function with trait.
* `BOOST_YAP_USER_LITERAL_PLACEHOLDER_OPERATOR` define a placeholder literal `_p` for parameter template.

------
### Expression Template Algorithms

* Header `<boost/yap/expression.hpp>`

```c++
struct expression<k,Tup> {
    using tuple_type = Tup; static const expr_kind kind = k;
    tuple_type elements;
    constexpr ctor(){}
    constexpr ctor(tuple_type&& t); // move-in init
    constexpr decltype(auto) value() {&|const&|&&} { return value(*<std::move>(*this)); }
    constexpr decltype(auto) left() {&|const&|&&} { return left(*<std::move>(*this)); }
    constexpr decltype(auto) right() {&|const&|&&} { return right(*<std::move>(*this)); }
    // op=, op[], op()
};
struct expression<terminal, hana::tuple<T>> {
    using tuple_type = Tup; static const expr_kind kind = terminal;
    tuple_type elements;
    constexpr ctor(){}
    constexpr ctor(T&& t); constexpr ctor(tuple_type{const&|&&} t); // move-in/copy init
    constexpr decltype(auto) value() {&|const&|&&} { return value(*<std::move>(*this)); }
    // op=, op[], op()
};
// all operators defined for `expression`

constexpr auto make_expression<k,...> (T&&...t) { return make_expression<expression,k>((T&&)t...); }
constexpr auto make_terminal <T> (T&& t) { return make_terminal<expression>((T&&)t); }
constexpr decltype(auto) as_expr <T> (T&& t) { return as_expr<expression>( (T&&)t ); }
```

------
### Printing Support

* Header `<boost/yap/print.hpp>`

```c++
constexpr char const* op_string(expr_kind kind) {
    switch (kind) {
    case expr_ref: return "ref";
    case terminal: return "term";
    case unary_plus: return "+"; // also every operators
    default: return "** ERROR: UNKNOWN OPERATOR! **";
    }
}
std::ostream& detail::print_kind(ostream& os, expr_kind k) { return os << op_string(k); }
std::ostream& detail::print_value <T> (ostream& os, T const& x) {
    if constexpr (requires{...}) return os << x;
    else return os << "<<unprintable-value>>";
}
std::ostream& detail::print_value <i> (ostream& os, hana::llong<i>) { return os << i << "_p"; }
std::ostream& detail::print_type <T> (ostream& os, hana::tuple<T> const&) {
    os << typeindex::type_id<T>().pretty_name();
    if (std::is_const_v<remove_reference_t<T>>) os << " const"; // also " volatile"
    if (std::is_lvalue_reference_v<T>) os << " &"; /// also " &&"
    return os;
}
bool detail::is_const_expr_ref <T> (T const&) { return false; }
bool detail::is_const_expr_ref <T,exprT<k,TT>> (exprT<expr_ref, hana::tuple<T const*>> const&) { return true; }

std::ostream& detail::print_impl <Expr> (ostream& os, Expr const& expr, int indent, char const* indent_str, bool is_ref=false, bool is_const_ref=false) {
    if constexpr (Expr::kind == expr_ref) print_impl(os, deref(expr), indent, indent_str, true, is_const_expr_ref(expr));
    else {
        for (int i = 0; i < indent; ++i) os << indent_str; // fill indent
        if constexpr (Expr::kind == terminal) {
            os << "term<"; print_type(os, expr.elements); os << ">[=" print_value(os, value(expr)); os << "]";
            if (is_const_ref) os << " const &" else os << " &"; os << "\n";
        } else {
            os << "expr<"; print_kind(os, Expr::kind); os << ">";
            if (is_const_ref) os << " const &" else os << " &"; os << "\n";
            hana::for_each(expr.elements, [=,&os](auto const& ele){ print_impl(os, ele, indent+1, indent_str); });
        }
    }
    return os;
}
std::ostream& print <Expr> (ostream& os, Expr const& expr) { return print_impl(os, expr, 0, "    "); }
```

------
#### Dependency

#### Boost.Hana

* `<boost/hana/*.hpp>`

#### Boost.Preprocessor

* `<boost/preprocessor/*.hpp>`

#### Boost.TypeIndex

* `<boost/type_index.hpp>` - required by printing.

------
### Standard Facilities
