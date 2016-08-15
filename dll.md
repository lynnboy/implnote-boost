# Boost.DLL

* lib: `boost/libs/dll`
* repo: `boostorg/dll`
* commit: `d2be74eb`, 2016-07-07

------
### Dynamic Library Supporting API

Main Header `<boost/dll.hpp>`

------
### Shared Library Loading

Header `<boost/dll/shared_library.hpp>`

The class `shared_library` handles library loading/unloading and low-level symbol finding.

```c++
enum load_mode::type {  // dll load options, defined to be platform value, or 0 otherwise
  default_mode,   // default, 'rtld_lazy' and 'rtld_local' is default
  dont_resolve_dll_references, load_ignore_code_authz_level, load_with_altered_search_path, // windows
  rtld_lazy, rtld_now, rtld_global, rtld_local, rtld_deepbind, // posix dlfcn.h
  append_decorations,  // both platform, try 'a.dll', 'liba.so', or 'liba.dylib' first
  search_system_folders // both platform, if off, treat path as relative to current directory
};
class shared_library {        // copyable type
  native_handle_t handle_;
public:
  using native_handle_t = void* [or HMODULE]; // library handle

  // default ctor, copy-ctor, copy-assign, move-ctor, move-assign, dtor, swap()
  // ctors for each 'load()' signature

  shared_library& assign(shared_library const&[, system::error_code& ec]);
  void load(filesystem::path const& lib_path, load_mode::type = default_mode);
  void load(filesystem::path const&, system::error_code& ec, load_mode::type = default_mode);
  void load(filesystem::path const&, load_mode::type, system::error_code& ec);
  void unload() noexcept;
  // is_loaded, operator!, explicit operator bool
  native_handle_t native() const noexcept;
  filesystem::path location() const;          // throws if not loaded
  filesystem::path location(system::error_code& ec) const;

  static filesystem::path suffix();           // fixed on each platform, '.dll', '.so', or '.dylib'

  bool has(const char* symbol_name) const noexcept; bool has(string const& symbol_name) const noexcept;
  T& get<T>(const char* symbol_name) const;         T& get(string const& symbol_name) const;
  T& get_alias<T>(const char* alias_name) const;    T& get_alias(string const& alias_name) const;
};
// operators '==', '!=', '<', and swap()
```

* API without `error_code` parameter will throw `system_error` on failure.
* On Windows, use `LoadLibraryEx` family API, on POSIXs, use `dlopen` family API.
* On Windows, use `GetModuleFileName` to get path from handle, on MacOS/iOS, iterate all loaded images for matching one.
* On POSIXes, also specially treated for current executable's file path.
* `get_alias` just calls `*get<T*>(alias_name)`, alias symbols are pointers to actual instance.
* `get` and `get_alias` supports member pointer and reference types for `T`.
* `shared_library` will unload when destroyed.

------
### Alias Symbol Declaration

Header `<boost/dll/alias.hpp>`

* `BOOST_DLL_AUTO_ALIAS(FuncOrVar)` - `BOOST_DLL_ALIAS(FuncOrVar, #FuncOrVar)`
* `BOOST_DLL_ALIAS(FuncOrVar, AliasName)` - `BOOST_DLL_ALIAS_SECTIONED(FuncOrVar, AliasName, boostdll)`
* `BOOST_DLL_ALIAS_SECTIONED(FuncOrVar, AliasName, SectionName)`

* On MacOS/iOS, sections are named "__DATA,SectionName"
* Alias names are implemented as `extern "C" const void *` pointing to real entity, with specified section and weak symbol.
* Alias are accessed via `shared_library::get_alias` to perform the indirection

* Because MinGW don't handle `dllexport` and weak symbol, thus at least on MinGW/Cygwin requires a translation unit to
  define `BOOST_DLL_FORCE_ALIAS_INSTANTIATION` to distinguish 'definition' from 'declarations'

------
### Reference Holding API

Header `<boost/dll/import.hpp>`.

```c++
class library_function<T> {
    shared_ptr<T> f_;
public:
    library_function(shared_ptr<shared_library> const& l, T* fp) : f_(l, fp) {}
    operator T*() const noexcept { return f_.get(); } // for pre-C++11
    auto operator()<Args...>(Args&&... args) const { return (*f_)(args...); } // for C++11
};
auto import[_alias]<T>(filesystem::path const& lib, [const char*|string const&] name,
                       load_mode::type mode = load_mode::default_mode);
auto import[_alias]<T>(shared_library [const&|&&] lib, [const const*|string const&] name);
```

* Result type will be callable wrapper `library_function` (or `boost::function<T>` on pre-C++11)
  for functions, while be `shared_ptr` for objects.

------
### Get Module Path

Header `<boost/dll/runtime_symbol_info.hpp>`

```c++
filesystem::path symbol_location<T>(T const& symbol[, system::error_code& ec]);
inline filesystem::path this_line_location([system::error_code& ec]) { return symbol_location(this_line_location); }
filesystem::path program_location([system::error_code& ec]);
```

------
### Module Symbol Query

Header `<boost/dll/library_info.hpp>`

```c++
class library_info : noncopyable {
  filesystem::ifstream f_;
  aligned_storage<...> impl_; // aligned for max size for elf_info, pe_info, macho_info, 32 and 64
  x_info_interface& impl() noexcept; // cast on 'impl_'
public:
  explicit library_info(filesystem::path const&, bool throw_if_not_native_format = true); // parse
  ~library_info() noexcept;

  std::vector<std::string> sections();
  std::vector<std::string> symbols();
  std::vector<std::string> symbols([const char*|string const&] section_name);
};

struct x_info_interface {
  virtual std::vector<std::string> sections() = 0;
  virtual std::vector<std::string> symbols() = 0;
  virtual std::vector<std::string> symbols(const char* section_name) = 0;
  virtual ~x_info_interface() noexcept {}
};

// PE
struct IMAGE_DOS_HEADER;
struct IMAGE_FILE_HEADER;
struct IMAGE_DATA_DIRECTORY;
struct IMAGE_EXPORT_DIRECTORY;
struct IMAGE_SECTION_HEADER;
struct IMAGE_OPTIONAL_HEADER_template<AddressOffsetT>; // IMAGE_OPTIONAL_HEADER32, IMAGE_OPTIONAL_HEADER64;
struct IMAGE_NT_HEADERS_template<AddressOffsetT>; // IMAGE_NT_HEADER32, IMAGE_NT_HEADER64;
template <class AddressOffsetT>
class pe_info: public x_info_interface {
public:
  explicit pe_info(filesystem::ifstream&) noexcept;
  // sections(), symbols(), symbols(section)
  static bool parsing_supported(filesystem::ifstream& f);
}
using pe_info32 = pe_info<DWORD>;
using pe_info64 = pe_info<ULONGLONG>;

// ELF
struct Elf_Ehdr_template<AddressOffsetT>; // Elf32_Ehdr; Elf64_Ehdr;
struct Elf_Shdr_template<AddressOffsetT>; // Elf32_Shdr; Elf64_Shdr;
struct Elf_Sym_template<AddressOffsetT>;  // Elf32_Sym; Elf64_Sym;
template <class AddressOffsetT>
class elf_info: public x_info_interface {
public:
  explicit elf_info(filesystem::ifstream&) noexcept;
  // sections(), symbols(), symbols(section)
  static bool parsing_supported(filesystem::ifstream& f);
}
using elf_info32 = pe_info<uint32_t>;
using elf_info64 = pe_info<uint64_t>;

// MACH-O
struct mach_header_template<AddressOffsetT>; // mach_header_32; mach_header_64;
struct load_comand;
struct segment_command_template<AddressOffsetT>; // segment_command_32; segment_command_64;
struct section_template<AddressOffsetT>;  // section_32; section_64;
struct symtab_command;
struct nlist_template<AddressOffsetT>;  // nlist_32; nlist_64;
template <class AddressOffsetT>
class macho_info: public x_info_interface {
public:
  explicit macho_info(filesystem::ifstream&) noexcept;
  // sections(), symbols(), symbols(section)
  static bool parsing_supported(filesystem::ifstream& f);
}
using macho_info32 = macho_info<uint32_t>;
using macho_info64 = macho_info<uint64_t>;
```

* Supports ELF, MACH-O, PE

------
### Experimental C++ Library Support

Header `<boost/dll/smart_library.hpp>`

```c++
class smart_library {
  shared_library _lib;
  mangled_storage_impl _storage;
public:
  shared_library const& shared_lib() const;     mangled_storage const& symbol_storage() const;

  // default-ctor, copy-ctor, dtor, assign(), swap()
  smart_library(path const&, [error_code& ec,] load_mode = default_mode);
  void load(path const&; [error_code& ec,] load_mode = default_mode);
  void unload() noexcept;
  bool is_loaded() const noexcept;  bool operator!() const noexcept; explicit operator bool() const noexcept;

  T& get_variable<T>(string const&) { return _lib.get<T>(_storage.get_variable<T>(name)); }
  Func& get_function<Func>(string const&) { return _lib.get<Func>(_storage.get_function<Func>(name)); }
  auto get_mem_fn<T,Func>(string const&) { return _lib.get<>(_storage.get_mem_fn<T,Func>(name)); }
  constructor<T> get_constructor<T>() { return load_ctor<T>(_lib, _storage.get_constructor<T>()); }
  destructor<T> get_destructor<T>() { return load_dtor<T>(_lib, _storage.get_destructor<T>()); }
  std::type_info& get_type_info<C>() const { return load_type_info<C>(_lib, _storage); }
  void add_type_alias<Alias>(string const&) { _storage.add_alias<Alias>(name); }

  bool has(const char*) const noexcept;    bool has(string const&) const noexcept; // same as `shared_library`
};
// ==, !=, <, swap
auto get<T_or_Sig>(smart_library const&, std::string const&); // sm.get_variable()/sm.get_function()
auto get<C,Sig>(smart_library const&, std::string const&); // sm.get_mem_fn()

string demangle_symbol(const char* mangled_name);

struct mangled_storage_impl {
    struct entry { string mangled, demangled; };
    vector<entry> storage_;
    map<typeindex::ctti_type_index, string> aliases_;

    // assign, swap, clear, get_storage
    // default-ctor, copy-ctor, move-ctor

    std::string get_name<T>() const; // find alias for T, otherwise ctti's pretty_name()
    mangled_storage_impl(vector<string> const& symbols) { add_symbols(symbols); }
    explicit mangled_storage_impl(library_info &);              void load(library_info &);
    explicit mangled_storage_impl(path const&, bool = true);    void load(path const&, bool = true);
    
    void add_alias<Alias>(string const& n)
    { aliases_.emplace(ctti_type_index::type_id<Alias>(), n); }
    void add_symbols(vector<string> const& symbols)
    { for (auto& sym : symbols) storage_.emplace_back(sym, demangled_symbol(sym)); }

    string get_name<T>() const {
      auto tx = stl_type_index::type_id<T>();
      return aliases_.count(tx) ? aliases_[tx] : tx.pretty_name();
    }
    
    string get_variable<T>(string const& n) {
        return find_if(storage_, [&](auto e) { return e.demangled == n; }).mangled;
    }
    string get_function<Func>(string const& n) {
        string matcher = n + '(' + arg_list<Func>() + ')'; // compose signature
        return find_if(storage_, [&](auto e) { return e.demangled == matcher; }).mangled;
    }
    string get_mem_fn<T,Func>(string const& n) {
        string matcher = get_name<T>() + "::" + n + '(' + arg_list<Func>() + ')'
                + const_rule<T>() + volatile_rule<T>(); // compose signature
        return find_if(storage_, [&](auto e) { return e.demangled == matcher; }).mangled;
    }
    using ctor_sym = ...; using dtor_sym = ...;
    ctor_sym get_constructor<Sig>() {
        string matcher = get_return_type<Sig>() + "::" + ... // compose signature
        return ctor_sym { // use mangled name, handle different ABI cases
            find_if(storage_, [&](auto e) { return e.demangled == matcher; }).mangled
        };
    }
    dtor_sym get_destructor<Sig>(); // similar as ctor
};

struct constructor<Signature> { // Class(Args...)
    using standard_t = ...;     standard_t standard;        void call_standard(Class* const, Args...);
    using allocating_t = ...;   allocating_t allocating;    Class* call_allocating(Args...);
    bool has_allocating() const;    bool is_empty() const;
};
struct destructor<Class> {
    using standard_t = void (*)(Class* const);   standard_t standard;    void call_standard(Class* const);
    using standard_t = void (*)(Class* const);   deleting_t deleting;    void call_deleting(Class* const);
    bool has_deleting() const;      bool is_empty() const;
};
constructor<Signature> load_ctor<Signature,Lib>(Lib& lib, ctor_sym const& ct) {
    return constructor<Signature>(lib.get<standard_t>(ct)[,lib.get<allocating_t>(ct.C3));
}
destructor<Class> load_ctor<Class,Lib>(Lib& lib, dtor_sym const& dt) {
    return destructor<Class>(lib.get<standard_t>(dt)[,lib.get<deleting_t>(dt.D0));
}
```

* Only MSVC ABI and Itanium ABI are supported.
* Differences between Itanium ABI and MSVC ABI mangling rules are handled accordingly.
* Undeclared type names can be aliased by dummy type.
* `constructor` provides either a standard or an allocating callable entry.
* `destructor` provides either a standard or a deleting callable entry.

------
### Reference Holding API for C++

* Header `<boost/dll/import_class.hpp>`
* Header `<boost/dll/import_mangled.hpp>`

```c++
struct function_tuple<Return, ...Args> { fn _f; Return operator()(Args...); };
struct mem_fn_tuple<Class, Return, ...Args> { mem_fn _f; Return operator()(Class*, Args...); };
class mangled_library_function<...Ts> {
    shared_ptr<function_tuple<Ts...>> f_;
    auto operator()<...Args>(Args&&... args) const { return f_(forward<Args>(args)); }
};
class mangled_library_mem_fn<Class, ...Ts> {
    shared_ptr<mem_fn_tuple<Ts...>> f_;
    auto operator()<ClassIn,...Args>(ClassIn *cl, Args&&... args) const { return f_(cl, forward<Args>(args)); }
};

struct deleter<T> {
    destructor<T> dtor; bool use_deleting;
    void operator() (T* t) {
	if (use_deleting) dtor.call_deleting(t);
	else { dtor.call_standard(t); delete[] reinterpret_cast<char*>(t);
    }
};
struct mem_fn_call_proxy<Class, U> {
    Class* t; mangled_library_mem_fn<Class,U> & mem_fn;
    auto operator()<...Args>(Args&&...args) const { mem_fn(t, forward<Args>(args)...); }
};
struct mem_fn_call_proxy<T, Return(Args...)> {
    T* t; const std::string & name; smart_library & _lib;
    Return operator()(Args...args) const; // load mem_fn, then call for t;
};
class imported_class<T> {
    smart_library _lib;  unique_ptr<T, deleter<T>> _data;
    bool _is_allocating;  size_t _size;  std::type_info& _ti;
public:
    static imported_class<T> make<...Args>(smart_library [const &|&&], [size_t size,] Args...);
    // no ctors except move-ctor, move-assign
    explicit operator bool() const;
    bool is_[move|copy]_[constructible|assignable](); // lookup for T's ctor/assign in _lib
    std::type_info const& get_type_info();
    T* get();
    imported_class<T> copy() const;  imported_class<T> move();
    void copy_assign(imported_class<T> const&) const;  void move_assign(imported_class<T>&);
    const mem_fn_call_proxy<T,Sig> call<[Tin,]Sig>(string const&);
    const mem_fn_call_proxy<...> operator->*<Tin,T2>(mangled_library_mem_fn<Tin,T2>&);
    auto import<...Args>(string const& name); // convenience
};

auto import_mangled<...Args>(filesystem::path const&, [const char*|string cosnt&] name,
                             load_mode::type mode = load_mode::default_mode);
auto import_mangled<...Args>([smart_library|shared_library] [const&|&&], [const char*|string cosnt&] name);
imported_class<T> import_class<T, ...Args>(smart_library [const&|&|&&],[size_t size,][string const& alias_name,]Args...);
```

* Similar to `import`, but `import_mangled` can import set of overloaded functions.
* `import_class` will construct an instance of an importable class.
* When no `size` is provided for `import_class`, allocating ctor and deleting dtor should available, otherwise standard ctor and dtor should available.
* The object created by `import_class` will be deleted automatically, and move/copy operations are supported by named members.

------
### Dependency

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/cstdint.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.WinAPI

* `<boost/detail/winapi/*.hpp>` - on windows (but `library_info` API doesn't require this)

#### Boost.Core

* `<boost/utility/addressof.hpp>` - by `import`
* `<boost/swap.hpp>`
* `<boost/utility/enable_if.hpp>`
* `<boost/utility/explicit_operator_bool.hpp>`
* `<boost/noncopyable.hpp>` - by `library_info`
* `<boost/core/demangle.hpp>` - by `smart_library` on non-MSVC

#### Boost.FileSystem

* `<boost/filesystem/path.hpp>`
* `<boost/filesystem/operations.hpp>`
* `<boost/filesystem/fstream.hpp>` - by `library_info`

#### Boost.Move

* `<boost/move/utility.hpp>`
* `<boost/move/move.hpp>` - by `import_mangled`

#### Boost.PreDef

* `<boost/predef/os.h>`
* `<boost/predef/library/c.h>`
* `<boost/predef/compiler.h>` - by `BOOST_DLL_ALIAS`
* `<boost/predef/architecture.h>` - by `library_info`
* `<boost/predef/compiler/visualc.h>` - by `symbol_location`

#### Boost.System

* `<boost/system/error_code.hpp>`, `<boost/system/system_error.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>` - pointer, reference, void, ...
* `<boost/aligned_sotrage.hpp>` - by `library_info`

#### Boost.MPL

* `<boost/mpl/max_element.hpp>`, `<boost/mpl/vector_c.hpp>` - by `library_info`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Function

* `<boost/function.hpp>` - by import on pre-C++11

#### Boost.SmartPtr

* `<boost/make_shared.hpp>` - by import

#### Boost.TypeIndex

* `<boost/type_index/stl_type_index.hpp>` - by `smart_library`

#### Boost.Spirit

* `<boost/spirit/home/x3.hpp>` - by `smart_library` (on MSVC), used to compose C++ signature string.

------
### Standard Facilities
