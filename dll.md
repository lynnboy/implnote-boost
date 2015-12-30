# Boost.DLL

* lib: `boost/libs/dll`
* repo: `boostorg/dll`
* commit: `9714445f`, 2015-12-29

------
### Dynamic Library Supporting API

Main Header `<boost/dll.hpp>`

------
### Shared Library Loading

Header `<boost/dll/shared_library.hpp>`

```c++
enum load_mode::type {  // dll load options, defined to be platform value, or 0 otherwise
  default_mode,   // default, 'rtld_lazy' and 'rtld_local' is default
  dont_resolve_dll_references, load_ignore_code_authz_level, load_with_altered_search_path, // windows
  rtld_lazy, rtld_now, rtld_global, rtld_local, rtld_deepbind, // posix dlfcn.h
  append_decorations  // both platform, try 'a.dll', 'liba.so', or 'liba.dylib' first
};
class shared_library {        // copyable type
  native_handle_t handle_;
public:
  using native_handle_t = void* [or HMODULE]; // library handle

  // default ctor, copy-ctor, copy-assign, move-ctor, move-assign, dtor, swap()
  // ctors for each 'load()' signature

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
  T& get_alias<T>(const char* symbol_name) const;   T& get_alias(string const& symbol_name) const;
};
// operators '==', '!=', '<', and swap()
```

* API without `error_code` parameter will throw `system_error` on failure.
* On Windows, use `LoadLibraryEx` family API, on POSIXs, use `dlopen` family API.
* `get_alias` just calls `*get<T*>(alias_name)`, alias symbols are pointers to actual instance.
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
auto import[_alias]<T>(filesystem::path const& lib, [const char*|string const&] name,
                       load_mode::type mode = load_mode::default_mode);
auto import[_alias]<T>(shared_library [const&|&&] lib, [const const*|string const&] name);
```

* Result type will be callable wrapper (or `boost::function<T>` on pre-C++11)
  for functions and `shared_ptr` for variables.

------
#### Get Module Path

Header `<boost/dll/runtime_symbol_info.hpp>`

```c++
filesystem::path symbol_location<T>(T const& symbol);
inline filesystem::path this_line_location() { return symbol_location(this_line_location); }
filesystem::path program_location(T const& symbol);
```

------
#### Module Symbol Query

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
  std::vector<std::string> symbols(const char* section_name);
  std::vector<std::string> symbols(string const& section_name);
};

struct x_info_interface {
  virtual std::vector<std::string> sections() = 0;
  virtual std::vector<std::string> symbols() = 0;
  virtual std::vector<std::string> symbols(const char* section_name) = 0;
  virtual ~x_info_interface() noexcept {}
};

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
### Dependency

#### Boost.Config

* `<boost/config.hpp>`

#### Boost.StaticAssert

* `<boost/static_assert.hpp>`

#### Boost.WinAPI

* `<boost/detail/winapi/*.hpp>` - on windows (but `library_info` API doesn't require this)

#### Boost.Core

* `<boost/utility/addressof.hpp>`
* `<boost/swap.hpp>`
* `<boost/utility/enable_if.hpp>` - by import
* `<boost/noncopyable.hpp>` - by `library_info`
* `<boost/utility/explicit_operator_bool.hpp>`

#### Boost.FileSystem

* `<boost/filesystem/path.hpp>`
* `<boost/filesystem/operations.hpp>`
* `<boost/filesystem/fstream.hpp>`

#### Boost.Move

* `<boost/move/utility.hpp>`

#### Boost.PreDef

* `<boost/predef/os.h>`
* `<boost/predef/library/c.h>`
* `<boost/predef/compiler.h>` - by `BOOST_DLL_ALIAS`
* `<boost/predef/architecture.h>` - by `library_info`

#### Boost.System

* `<boost/system/error_code.hpp>`, `<boost/system/system_error.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/is_pointer.hpp>`, `<boost/type_traits/is_const.hpp>`
* `<boost/type_traits/is_object.hpp>` - by import
* `<boost/type_traits/integral_constant.hpp>`
* `<boost/aligned_storage.hpp>` - by `library_info`

#### Boost.MPL

* `<boost/mpl/max_element.hpp>`, `<boost/mpl/vector_c.hpp>` - by `library_info`

#### Boost.ThrowException

* `<boost/throw_exception.hpp>`

#### Boost.Function

* `<boost/function.hpp>` - by import

#### Boost.SmartPtr

* `<boost/make_shared.hpp>` - by import

------
### Standard Facilities
