# Boost.Algorithm

* lib: `boost/libs/algorithm`
* repo: `boostorg/algorithm`
* commit: `0edbfe8`, 2025-08-25

------
### Searchers

```c++
class detail::skip_table<K,V,useArray>;
class detail::skip_table<K,V,false> { using skip_map = std::unordered_map<K,V>;
    const V k_default_value; skip_map skip_;
public: ctor(size_t patSize, V default_value) : k_default_value{default_value}, skip_{patSize}{}
    void insert(K key, V val) { skip_[key]=val; }
    V operator[](K key) const { auto it=skip_.find(key); return it==skip_.end()?k_default_value:it->second; }
};
class detail::skip_table<K,V,true> { using UK = make_unsigned<K>::type; using skip_map = array<V,(1<<CHRA_BIT*sizeof(K))>;
    const V k_default_value; skip_map skip_;
public: ctor(size_t, V default_value) : k_default_value{default_value} {}
    void insert(K key, V val) { skip_[(UK)key]=val; }
    V operator[](K key) const { return skip_[(UK)key]; }
};
struct detail::BM_traits<It> {
    using value_type = std::iterator_traits<It>::difference_type;
    using key_type = std::iterator_traits<It>::value_type;
    using skip_table_t = skip_table<key_type, value_type, (is_integral_v<key_type>&&sizeof(key_type)==1)>;
};

class boyer_moore<It,Tr=BM_traits<It>> { using difference_type = std::iterator_traits<It>::difference_type;
    It pat_f, pat_l; const difference_type k_pat_len; Tr::skip_table_t skip_; std::vector<difference_type> suffix_;
    std::pair<It2,It2> do_search<It2>(It2 corpus_f, It2 corpus_l) const {
        It curPos=corpus_f; const It lastPos=corpus_l-k_pat_len; difference_type j,k,m;
        while (curPos<=lastPos) {
            j=k_pat_len; while (pat_f[j-1]==curPos[j-1]) {j--; if (j==0) return {curPos,curPos+k_pat_len}; }
            k=skip_[curPos[j-1]]; m=j-k-1;
            if (k<j && m>suffix_[j]) curPos+=m; else curPos += suffix_[j];
        } return {corpus_l,corpus_l};
    }
    void build_skip_table(It f, It l) { for(size_t i=0;f!=l;++f,++i) skip_.insert(*f,i); }
    void compute_bm_prefix<It,Cont>(It f, It l, Cont& prefix) {
        const size_t count=std::distance(f,l); prefix[0]=0; size_t k=0;
        for (size_t i=1;i<count;++i) {
            while (k>0 && f[k]!=f[i]) k=prefix[k-1];
            if (f[k]==f[i]) k++;
            prefix[i]=k;
        }
    }
    void build_suffix_table(It f, It l) {
        const size_t count=std::distance(f,l);
        if (count>0) { std::vector<std::iterator_traits<It>::value_type> reversed(count); std::reverse_copy(f,l,reversed.begin());
            std::vector<difference_type> prefix{count}, prefix_r;
            compute_bm_prefix(f,l,prefix); compute_bm_prefix(reversed.begin(),reversed.end(),prefix_r);
            for (size_t i=0; i<=count; i++) suffix_[i]=count-prefix[count-1];
            for (size_t i=0; i<count; i++) { size_t j=count-prefix_r[i]; difference_type k=i-prefix_r[i]+1; if (suffix_[j]>k) suffix_[j]=k; }
        }
    }
public: ctor(It f, It l) : pat_f{f}, pat_l{l}, k_pat_len{std::distance(f,l)}, skip_{k_pat_len,-1}, suffix{k_pat_len+1}
    { build_skip_table(f,l), build_suffix_table(f,l); }
    ~dtor(){}
    std::pair<It2,It2> operator()<It2>(It2 corpus_f, It2 corpus_l) const {
        if (corpus_f==corpus_l) return {corpus_l,corpus_l}; if(pat_f==pat_l) return {corpus_f,corpus_f};
        if (std::distance(corpus_f,corpus_l) < k_pat_len) return {corpus_l,corpus_l};
        return do_search(corpus_f,corpus_l);
    }
    std::pair<range_iterator<R>::type,range_iterator<R>::type> operator()<R>(R& r) const { return (*this)(begin(r),end(r)); }
};
std::pair<It2,It2> boyer_moore_search<It,It2>(It2 corpus_f, It2 corpus_l, It pat_f, It pat_l)
{ return boyer_moore<It>{pat_f,pat_l}(corpus_f,corpus_l); }
std::pair<It2,It2> boyer_moore_search<R,It2>(It2 corpus_f, It2 corpus_l, const R& pat)
{ return boyer_moore<It>{begin(pat),end(pat)}(corpus_f,corpus_l); }
std::pair<range_iterator<CorR>::type,range_iterator<CorR>::type> boyer_moore_search<It,CorR>(CorR& corpus, It pat_f, It pat_l)
{ return boyer_moore<It>{pat_f,pat_l}(begin(corpus),end(corpus)); }
std::pair<range_iterator<CorR>::type,range_iterator<CorR>::type> boyer_moore_search<PatR,CorR>(CorR& corpus, const PatR& pat)
{ return boyer_moore{begin(pat),end(pat)}(begin(corpus),end(corpus)); }
boyer_moore<range_iterator<const R>::type> make_boyer_moore(const R& r) { return {begin(r),end(r)}; }
boyer_moore<range_iterator<R>::type> make_boyer_moore(R& r) { return {begin(r),end(r)}; }

class boyer_moore_horspool<It,Tr=BM_traits<It>> { using difference_type = std::iterator_traits<It>::difference_type;
    It pat_f, pat_l; const difference_type k_pat_len; Tr::skip_table_t skip_;
    std::pair<It2,It2> do_search<It2>(It2 corpus_f, It2 corpus_l) const {
        It curPos=corpus_f; const It lastPos=corpus_l-k_pat_len;
        while (curPos<=lastPos) {
            size_t j=k_pat_len-1; while (pat_f[j]==curPos[j]) {if (j==0) return {curPos,curPos+k_pat_len}; j--; }
            curPos += skip_[curPos[k_pat_len-1]];
        } return {corpus_l,corpus_l};
    }
public: ctor(It f, It l) : pat_f{f}, pat_l{l}, k_pat_len{std::distance(f,l)}, skip_{k_pat_len,k_pat_len}
    { size_t i=0; if (f!=l) for (It it=f; it!=l-1; ++it,++i) skip_.insert(*it,k_pat_len-1-i); }
    ~dtor(){}
    std::pair<It2,It2> operator()<It2>(It2 corpus_f, It2 corpus_l) const {
        if (corpus_f==corpus_l) return {corpus_l,corpus_l}; if(pat_f==pat_l) return {corpus_f,corpus_f};
        if (std::distance(corpus_f,corpus_l) < k_pat_len) return {corpus_l,corpus_l};
        return do_search(corpus_f,corpus_l);
    }
    std::pair<range_iterator<R>::type,range_iterator<R>::type> operator()<R>(R& r) const { return (*this)(begin(r),end(r)); }
};
std::pair<It2,It2> boyer_moore_horspool_search<It,It2>(It2 corpus_f, It2 corpus_l, It pat_f, It pat_l)
{ return boyer_moore_horspool<It>{pat_f,pat_l}(corpus_f,corpus_l); }
std::pair<It2,It2> boyer_moore_horspool_search<R,It2>(It2 corpus_f, It2 corpus_l, const R& pat)
{ return boyer_moore_horspool<It>{begin(pat),end(pat)}(corpus_f,corpus_l); }
std::pair<range_iterator<CorR>::type,range_iterator<CorR>::type> boyer_moore_horspool_search<It,CorR>(CorR& corpus, It pat_f, It pat_l)
{ return boyer_moore_horspool<It>{pat_f,pat_l}(begin(corpus),end(corpus)); }
std::pair<range_iterator<CorR>::type,range_iterator<CorR>::type> boyer_moore_horspool_search<PatR,CorR>(CorR& corpus, const PatR& pat)
{ return boyer_moore_horspool{begin(pat),end(pat)}(begin(corpus),end(corpus)); }
boyer_moore_horspool<range_iterator<const R>::type> make_boyer_moore_horspool(const R& r) { return {begin(r),end(r)}; }
boyer_moore_horspool<range_iterator<R>::type> make_boyer_moore_horspool(R& r) { return {begin(r),end(r)}; }

class knuth_morris_pratt<It> { using difference_type = std::iterator_traits<It>::difference_type;
    It pat_f, pat_l; const difference_type k_pat_len; Tr::skip_table_t skip_;
    std::pair<It2,It2> do_search<It2>(It2 corpus_f, It2 corpus_l, difference_type k_corpus_len) const {
        difference_type match_start=0, idx=0, const last_match=k_corpos_len-k_pat_len;
        while (match_start<=last_match) {
            while (pat_f[idx]==corpus_f[match_start+idx]) {if (+idx==k_pat_len) return {corpus_f+match_start,corpus_f+match_start+k_pat_len}; }
            match_start+=idx-skip_[idx]; idx=skip_[idx]>=0?skip_[idx]:0;
        } return {corpus_l,corpus_l};
    }
    void init_skip_table(It f, It l) { difference_type j, const count=std::distance(f,l);
        skip_[0]=-1;
        for (int i=1; i<=count; ++i) { j=skip_[i-1];
            while (j>=0) { if (f[j]==f[i-1]) break; j=skip_[j]; }
            skip_[i]=j+1;
        }
    }
public: ctor(It f, It l) : pat_f{f}, pat_l{l}, k_pat_len{std::distance(f,l)}, skip_{k_pat_len+1}
    { init_skip_table(pat_f,pat_l); }
    ~dtor(){}
    std::pair<It2,It2> operator()<It2>(It2 corpus_f, It2 corpus_l) const {
        if (corpus_f==corpus_l) return {corpus_l,corpus_l}; if(pat_f==pat_l) return {corpus_f,corpus_f};
        auto k_corpos_len=std::distance(corpus_f,corpus_l); if (k_corpus_len < k_pat_len) return {corpus_l,corpus_l};
        return do_search(corpus_f,corpus_l,k_corpus_len);
    }
    std::pair<range_iterator<R>::type,range_iterator<R>::type> operator()<R>(R& r) const { return (*this)(begin(r),end(r)); }
};
std::pair<It2,It2> knuth_morris_pratt_search<It,It2>(It2 corpus_f, It2 corpus_l, It pat_f, It pat_l)
{ return knuth_morris_pratt<It>{pat_f,pat_l}(corpus_f,corpus_l); }
std::pair<It2,It2> knuth_morris_pratt_search<R,It2>(It2 corpus_f, It2 corpus_l, const R& pat)
{ return knuth_morris_pratt<It>{begin(pat),end(pat)}(corpus_f,corpus_l); }
std::pair<range_iterator<CorR>::type,range_iterator<CorR>::type> knuth_morris_pratt_search<It,CorR>(CorR& corpus, It pat_f, It pat_l)
{ return knuth_morris_pratt<It>{pat_f,pat_l}(begin(corpus),end(corpus)); }
std::pair<range_iterator<CorR>::type,range_iterator<CorR>::type> knuth_morris_pratt_search<PatR,CorR>(CorR& corpus, const PatR& pat)
{ return knuth_morris_pratt{begin(pat),end(pat)}(begin(corpus),end(corpus)); }
knuth_morris_pratt<range_iterator<const R>::type> make_knuth_morris_pratt(const R& r) { return {begin(r),end(r)}; }
knuth_morris_pratt<range_iterator<R>::type> make_knuth_morris_pratt(R& r) { return {begin(r),end(r)}; }
```

------
### Standard Algorithms

#### C++11

```c++
constexpr bool all_of<InIt,Pred>(InIt f, InIt l, Pred p);
constexpr bool all_of<R,Pred>(const R& r, Pred p);
constexpr bool all_of_equal<InIt,T>(InIt f, InIt l, const T& val);
constexpr bool all_of_equal<R,T>(const R& r, const T& val);

constexpr bool any_of<InIt,Pred>(InIt f, InIt l, Pred p);
constexpr bool any_of<R,Pred>(const R& r, Pred p);
constexpr bool any_of_equal<InIt,T>(InIt f, InIt l, const T& val);
constexpr bool any_of_equal<R,T>(const R& r, const T& val);

constexpr bool none_of<InIt,Pred>(InIt f, InIt l, Pred p);
constexpr bool none_of<R,Pred>(const R& r, Pred p);
constexpr bool none_of_equal<InIt,T>(InIt f, InIt l, const T& val);
constexpr bool none_of_equal<R,T>(const R& r, const T& val);

constexpr bool one_of<InIt,Pred>(InIt f, InIt l, Pred p); // not in STD
constexpr bool one_of<R,Pred>(const R& r, Pred p);
constexpr bool one_of_equal<InIt,T>(InIt f, InIt l, const T& val);
constexpr bool one_of_equal<R,T>(const R& r, const T& val);

constexpr FwdIt is_sorted_until<FwdIt,Pred>(FwdIt f, FwdIt l, Pred p);
constexpr FwdIt is_sorted_until<FwdIt>(FwdIt f, FwdIt l); // std::less<value_type>
constexpr range_iterator<const R> is_sorted_until<R,Pred>(const R& range, Pred p);
constexpr range_iterator<const R> is_sorted_until<R>(const R& range);
constexpr bool is_sorted<FwdIt,Pred>(FwdIt f, FwdIt l, Pred p);
constexpr bool is_sorted<FwdIt>(FwdIt f, FwdIt l); // std::less<value_type>
constexpr range_iterator<const R> is_sorted<R,Pred>(const R& range, Pred p);
constexpr range_iterator<const R> is_sorted<R>(const R& range);
constexpr bool is_increasing<FwdIt>(FwdIt f, FwdIt l); // std::less<value_type>
constexpr bool is_increasing<R>(const R& range);
constexpr bool is_decreasing<FwdIt>(FwdIt f, FwdIt l); // std::greater<value_type>
constexpr bool is_decreasing<R>(const R& range);
constexpr bool is_strictly_increasing<FwdIt>(FwdIt f, FwdIt l); // std::less_equal<value_type>
constexpr bool is_strictly_increasing<R>(const R& range);
constexpr bool is_strictly_decreasing<FwdIt>(FwdIt f, FwdIt l); // std::greater_equal<value_type>
constexpr bool is_strictly_decreasing<R>(const R& range);

constexpr bool is_partitioned<InIt,UPred>(InIt f, InIt l, UPred p);
constexpr bool is_partitioned<R,UPred>(const R& r, UPred p);

bool is_permutation<FwdIt1,FwdIt2,BPred>(FwdIt1 f1, FwdIt1 l1, FwdIt2 f2, BPred p);
bool is_permutation<FwdIt1,FwdIt2>(FwdIt1 f1, FwdIt1 l1, FwdIt2 f2); // std::equal_to<value_type>
bool is_permutation<R,FwdIt,BPred>(const R& r, FwdIt f2, BPred pred);
bool is_permutation<R,FwdIt>(const R& r, FwdIt f2);

FwdIt partition_point<FwdIt,Pred>(FwdIt f, FwdIt l, Pred p);
range_iterator<R>::type partition_point<R,Pred>(Range& r, Pred p);

constexpr FwdIt partition_copy<FwdIt,Pred>(FwdIt f, FwdIt l, Pred p);
constexpr range_iterator<R>::type partition_copy<R,Pred>(Range& r, Pred p);

constexpr OutIt copy_if<InIt,OutIt,Pred>(InIt f, InIt l, OutIt result, Pred p);
constexpr OutIt copy_if<R,OutIt,Pred>(const R& r, OutIt result, Pred p);
constexpr std::pair<InIt,OutIt> copy_while<InIt,OutIt,Pred>(InIt f, InIt l, OutIt result, Pred p);
constexpr std::pair<range_iterator<const R>::type,OutIt> copy_while<R,OutIt,Pred>(const R& r, OutIt result, Pred p);
constexpr std::pair<InIt,OutIt> copy_until<InIt,OutIt,Pred>(InIt f, InIt l, OutIt result, Pred p);
constexpr std::pair<range_iterator<const R>::type,OutIt> copy_until<R,OutIt,Pred>(const R& r, OutIt result, Pred p);
constexpr std::pair<InIt,OutIt> copy_if_while<InIt,OutIt,CopyPred,TermPred>(InIt f, InIt l, OutIt result, CopyPred copy_pred, TermPred term_pred);
constexpr std::pair<range_iterator<const R>::type,OutIt> copy_if_while<R,OutIt,CopyPred,TermPred>(const R& r, OutIt result, CopyPred copy_pred, TermPred term_pred);
constexpr std::pair<InIt,OutIt> copy_if_until<InIt,OutIt,CopyPred,TermPred>(InIt f, InIt l, OutIt result, CopyPred copy_pred, TermPred term_pred);
constexpr std::pair<range_iterator<const R>::type,OutIt> copy_if_until<R,OutIt,CopyPred,TermPred>(const R& r, OutIt result, CopyPred copy_pred, TermPred term_pred);

constexpr OutIt copy_n<InIt,Size,OutIt>(InIt f, Size n, OutIt result);

constexpr void iota<FwdIt,T>(FwdIt f, FwdIt l, T value);
constexpr void iota<R,T>(Range& r, T value);
constexpr OutIt iota_n<OutIt,T>(OutIt out, T value, size_t n);

constexpr InIt find_if_not<InIt,Pred>(InIt f, InIt l, Pred p);
constexpr range_iterator<const R>::type find_if_not<R,Pred>(const R& r, Pred p);
```

#### C++14

```c++
constexpr bool equal<InIt1,InIt2,BPred>(InIt1 f1, InIt1 l1, InIt2 f2, InIt2 l2, BPred pred);
constexpr bool equal<InIt1,InIt2>(InIt1 f1, InIt1 l1, InIt2 f2, InIt2 l2); // v1==v2
constexpr std::pair<InIt1,InIt2> mismatch<InIt1,InIt2,BPred>(InIt1 f1, InIt1 l1, InIt2 f2, InIt2 l2, BPred pred);
constexpr std::pair<InIt1,InIt2> mismatch<InIt1,InIt2>(InIt1 f1, InIt1 l1, InIt2 f2, InIt2 l2); // v1==v2
constexpr bool is_permutation<FwdIt1,FwdIt2,BPred>(InIt1 f1, InIt1 l1, InIt2 f2, InIt2 l2, BPred pred);
constexpr bool is_permutation<FwdIt1,FwdIt2>(InIt1 f1, InIt1 l1, InIt2 f2, InIt2 l2); // v1==v2
```

#### C++17

```c++
InIt for_each_n<InIt,Size,Func>(InIt f, Size n, Func f);

OutIt exclusive_scan<InIt,OutIt,T,BinOp>(InIt f, InIt l, OutIt result, T init, BinOp op);
OutIt exclusive_scan<InIt,OutIt,T>(InIt f, InIt l, OutIt result, T init); // std::plus<value_type>
OutIt inclusive_scan<InIt,OutIt,T,BinOp>(InIt f, InIt l, OutIt result, BinOp op, T init);
OutIt inclusive_scan<InIt,OutIt,BinOp>(InIt f, InIt l, OutIt result, BinOp op);
OutIt inclusive_scan<InIt,OutIt>(InIt f, InIt l, OutIt result); // std::plus<value_type>
T reduce<InIt,T,BinOp>(InIt f, InIt l, T init, BinOp op);
T reduce<InIt,T>(InIt f, InIt l, T init); // std::plus<value_type>
std::iterator_traits<InIt>::value_type reduce<InIt>(InIt f, InIt l);
range_value<R>::type reduce(const R& r);
T reduce<R,T,BinOp>(const R& r, T init, BinOp op);
T reduce<R,T>(const R& r, T init);

OutIt transform_exclusive_scan<InIt,OutIt,T,BinOp,UnOp>(InIt f, InIt l, OutIt result, T init, BinOp bop, UnOp uop);
OutIt transform_inclusive_scan<InIt,OutIt,BinOp,UnOp,T>(InIt f, InIt l, OutIt result, BinOp bop, UnOp uop, T init);
OutIt transform_inclusive_scan<InIt,OutIt,BinOp,UnOp>(InIt f, InIt l, OutIt result, BinOp bop, UnOp uop);
T transform_reduce<InIt1,InIt2,T,BinOp1,BinOp2>(InIt1 f1, InIt1 l1, InIt2 f2, T init, BinOp1 op1, BinOp2 op2);
T transform_reduce<InIt,T,BinOp,UnOp>(InIt f, InIt l, T init, BinOp bop, UnOp uop);
T transform_reduce<InIt1,InIt2,T>(InIt1 f1, InIt1 l1, InIt2 f2, T init);
```

------
### Other Algorithms

```c++
constexpr T identity_operator<T>(std::multiplies<T>) { return {1}; }
constexpr T identity_operator<T>(std::plus<T>) { return {0}; }
constexpr T power<T,Integer,Op>(T x, Integer n, Op op);
constexpr T power<T,Integer>(T x, Integer n);

void apply_permutation<RAIt1,RAIt2>(RAIt1 item_b, RAIt1 item_e, RAIt2 ind_b, RAIt2 ind_e);
void apply_reverse_permutation<RAIt1,RAIt2>(RAIt1 item_b, RAIt1 item_e, RAIt2 ind_b, RAIt2 ind_e);
void apply_permutation<R1,R2>(R1& item_range, R2& ind_range);
void apply_reverse_permutation<R1,R2>(R1& item_range, R2& ind_range);

constexpr T const& clamp<T,Pred>(T const& val, type_identity<T>::type const& lo, type_identity<T>::type const& hi, Pred p);
constexpr T const& clamp<T>(T const& val, type_identity<T>::type const& lo, hi);
constexpr OutIt clamp_range<InIt,OutIt,Pred>(InIt f, InIt l, OutIt out, std::iterator_traits<InIt>::value_type const& lo, hi, Pred p);
constexpr OutIt clamp_range<InIt,OutIt>(InIt f, InIt l, OutIt out, std::iterator_traits<InIt>::value_type const& lo, hi);
constexpr OutIt clamp_range<R,OutIt,Pred>(const R& r, OutIt out, std::iterator_traits<range_iterator<const R>::type>::value_type const& lo, hi, Pred p);
constexpr OutIt clamp_range<R,OutIt>(const R& r, OutIt out, std::iterator_traits<range_iterator<const R>::type>::value_type const& lo, hi);
constexpr bool is_clamped<T,Pred>(const T& val, type_identity<T>::type const& lo, hi, Pred p);
constexpr bool is_clamped<T>(const T& val, type_identity<T>::type const& lo, hi);

constexpr BidiIt find_backward<BidiIt,T>(BidiIt f, BidiIt l, const T& x);
constexpr range_iterator<R>::type find_backward<R,T>(R& range, const T& x);
constexpr BidiIt find_not_backward(BidiIt,T)(BidiIt f, BidiIt l, const T& x);
constexpr range_iterator<R>::type find_not_backward<R,T>(R& range, const T& x);
constexpr BidiIt find_if_backward<BidiIt,Pred>(BidiIt f, BidiIt l, Pred p);
constexpr range_iterator<R>::type find_if_backward<R,Pred>(R& range, Pred p);
constexpr BidiIt find_if_not_backward<BidiIt,Pred>(BidiIt f, BidiIt l, Pred p);
constexpr range_iterator<R>::type find_if_not_backward<R,Pred>(R& range, Pred p);

constexpr InIt find_not<InIt,S,T>(InIt f, S l, const T& x);
constexpr range_iterator<R>::type find_not<R,T>(R& r, const T& x);

std::pair<BidiIt,BidiIt> gather<BidiIt,Pred>(BidiIt f, BidiIt l, BidiIt pivot, Pred p);
std::pair<range_iterator<BidiR>::type,range_iterator<BidiR>::> gather<BidiR,Pred>(BidiR& range, range_iterator<BidiR>::type pivot, Pred p);

struct hex_decode_error : virtual boost::exception, virtual std::exception{};
struct not_enough_input : virtual hex_decode_error{};
struct non_hex_input : virtual hex_decode_error{};
using bad_char = boost::error_info<struct bad_char_,char>;
OutIt hex<InIt,OutIt>(InIt f, InIt l, OutIt out);
OutIt hex_lower<InIt,OutIt>(InIt f, InIt l, OutIt out);
OutIt hex<T,OutIt>(const T* ptr, OutIt out);
OutIt hex_lower<T,OutIt>(const T* ptr, OutIt out);
OutIt hex<R,OutIt>(const R& r, OutIt out);
OutIt hex_lower<R,OutIt>(const R& r, OutIt out);
OutIt unhex<InIt,OutIt>(InIt f, InIt l, OutIt out);
OutIt unhex<T,OutIt>(const T* ptr, OutIt out);
OutIt unhex<R,OutIt>(const R& r, OutIt out);
Str hex<Str>(const Str& in);
Str hex_lower<Str>(const Str& in);
Str unhex<Str>(const Str& in);

bool is_palindrome<BidiIt,Pred>(BidiIt b, BidiIt e, Pred p);
bool is_palindrome<BidiIt>(BidiIt b, BidiIt e); // std::equal_to<value_type>
bool is_palindrome<R,Pred>(const R& range, Pred p);
bool is_palindrome<R>(const R& range);
bool is_palindrome<Pred>(const char* str, Pred p);
bool is_palindrome(const char* str);

InIt is_partitioned_until<InIt,UnPred>(InIt f, InIt l, UnPred p);
range_iterator<const R>::type is_partitioned_until<R,UnPred>(const R& r, UnPred p);

void sort_subrange<It,Pred>(It f, It l, It sub_f, It sub_l, Pred p);
void sort_subrange<It,Pred>(It f, It l, It sub_f, It sub_l);
void partition_subrange<It,Pred>(It f, It l, It sub_f, It sub_l, Pred p);
void partition_subrange<It,Pred>(It f, It l, It sub_f, It sub_l);
```

------
### Dependency

#### Boost.Array

* `<boost/array.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Bind

* `<boost/bind/bind.hpp>`

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.Core

* `<boost/core/enable_if.hpp>`

#### Boost.Exception

* `<boost/exception/exception.hpp>`
* `<boost/exception/info.hpp>`

#### Boost.Range

* `<boost/range/begin.hpp>`, `<boost/range/end.hpp>`
* `<boost/range/value_type.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_integral.hpp>`
* `<boost/type_traits/is_same.hpp>`
* `<boost/type_traits/make_unsigned.hpp>`
* `<boost/type_traits/remove_const.hpp>`
* `<boost/type_traits/remove_pointer.hpp>`
* `<boost/type_traits/type_identity.hpp>`

#### Boost.Unordered

* `<boost/unordered_map.hpp>` - by bm serchers

------
### Standard Facilities

Library:
* `boyer_moore_searcher`, `boyer_moore_horspool_searcher` (C++17)
* `all_of`, `any_of`, `none_of` (C++11), `ranges` version (C++20)
* `copy_if`, `copy_n` (C++11), `ranges` version (C++20)
* `find_if_not` (C++11), `ranges` version (C++20)
* `iota` (C++11), `ranges` version (C++20)
* `is_sorted`, `is_sorted_until` (C++11), `ranges` version (C++20)
* `is_partitioned`, `partition_copy`, `partition_point` (C++11), `ranges` version (C++20)
* `is_permutation` (C++11), `ranges` version (C++20)
* `equal(f1,l1,f2,l2,<pred>)`, `mismatch(f1,l1,f2,l2,<pred>)`, `is_permutation(f1,l1,f2,l2,<pred>)`  (C++14)
* `for_each_n` (C++17)
* `exclusive_scan`, `inclusive_scan`, `reduce`, `transform_exclusive_scan`, `transform_inclusive_scan`, `transform_reduce` (C++17)
* `clamp` (C++17)
