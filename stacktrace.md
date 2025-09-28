# Boost.StackTrace

* lib: `boost/libs/stacktrace`
* repo: `boostorg/stacktrace`
* commit: `cf59922`, 2025-09-09

------
#### `basic_stacktrace` API

* Header `<boost/stacktrace/stacktrace_fwd.hpp>`
* Header `<boost/stacktrace.hpp>`

```c++
using detail::native_frame_ptr_t = const void*;
To detail::void_ptr_cast<To,From> (From* v) noexcept; // is_pointer_v<T> && sizeof(From*) == sizeof(To)

class frame { native_frame_ptr_t addr_{nullptr}; // wrapper of pointer
public:
    constexpr ctor() noexcept : addr_{nullptr} {};
    explicit ctor<T> (T* func_addr) noexcept : addr_(void_ptr_cast<native_frame_ptr_t>(func_addr)) {}; // init

    constexpr bool empty() const noexcept { return !addr_; }
    constexpr explicit operator bool() const noexcept { return !empty(); }
    constexpr native_frame_ptr_t address() const noexcept { return addr_; }
    std::string name() const; std::string source_file() const; size_t source_line() const;
};
std::string detail::to_string(const frame* frames, size_t size);
constexpr bool operator < (const frame& lhs, const frame& rhs) noexcept; // compare by address(); also >, <=, >=, ==, !=
size_t hash_value(const frame& f) noexcept { return (size_t)f.address(); }
std::string to_string(const frame& f);
std::basic_stream<Ch,Tr>& operator<< (std::basic_stream<Ch,Tr>& os, const frame& f) { return os<< to_string(f); }

class basic_stacktrace<Alloc=std::allocator<frame>> {
    std::vector<frame,Alloc> impl_; using vec_ = vector<frame,Alloc>; // frames, wrapped container
    void fill(native_frame_ptr_t* begin, size_t size); // fill with array buffer
    void init(size_t frames_to_skip, size_t max_depth); // capture and fill, call `collect`
public:
    using value_type = vec_::value_type; // other vector's member types: allocator_type, size_type, difference_type
    using pointer = vec_::const_pointer; // reference, iterator, reverse_iterator are const_ ones

    explicit ctor(const allocator_type& a={}) noexcept : impl_(a) { init(0, -1); } // capture on ctor
    ctor(size_t skip, size_t max_depth, const allocator_type& a={}) noexcept : impl_(a) { init(skip, max_depth); }
    // copy/move ctor, copy/move assign

    size_type size() const noexcept { return impl_.size(); }
    // and empty, operator[], [cr]begin/end, all forward to impl_
    constexpr explicit operator bool() const noexcept { return !empty(); }
    const std::vector<frame,Alloc>& as_vector() const noexcept { return impl_; }

    static basic_stacktrace from_dump (const void* begin, size_t buffer_size, const allocator_type& a={}); // call `from_dump` then fill
    static basic_stacktrace from_dump <Ch,Tr> (std::basic_istream<Ch,Tr>& is, const allocator_type& a={});
    static basic_stacktrace from_current_exception(const allocator_type& a={}) noexcept; // call `current_exception_stacktrace` then fill
};
bool operator< <A1,A2> (const basic_stacktrace<A1>& lhs, const basic_stacktrace<A2>& rhs) noexcept
{ return lhs.size() < rhs.size() || lhs.as_vector() < rhs.as_vector(); } // and >, <=, >=, ==, !=
size_t hash_value <A> (const basic_stacktrace<A>& st) noexcept; // hash_range on members
std::string to_string<A> (const basic_stacktrace<A>& st); // detail::to_string for array of frame
std::basic_stream<Ch,Tr,A>& operator<< (std::basic_stream<Ch,Tr>& os, const basic_stacktrace<A>& st) { return os<< to_string(st); }

using stacktrace = basic_stacktrace<>;
```

* Implementations:
  - `noop`
  - `addr2line` (need fork), `backtrace` (libbacktrace), `basic` (just try libdl's symbol names)
  - `windbg` and `windbg_cached`(use `thread_local` data for threading support)

------
#### Stacktrace Capturing

* Header `<boost/stacktrace/this_thread.hpp>`

```c++
// capture stacktrace frame data
constexpr size_t detail::max_frames_dump = 128;
size_t detail::this_thread_frames::collect(native_frame_ptr_t* out_frames, size_t max_count, size_t skip) noexcept;
size_t detail::dump (const char* file, const native_frame_ptr_t* frames, size_t count) noexcept;
size_t detail::dump (platform_specific_descriptor fd, const native_frame_ptr_t* frames, size_t count) noexcept;
size_t detail::from_dump (const char* file, native_frame_ptr_t* frames) noexcept; // load from file

// dump to null-terminated array of native_frame_ptr_t
size_t safe_dump_to (void* memory, size_t size) noexcept;
size_t safe_dump_to (size_t skip, void* memory, size_t size) noexcept;
size_t safe_dump_to (const char* file) noexcept; // dump to file
size_t safe_dump_to (size_t skip, size_t max_depth; const char* file) noexcept;
size_t safe_dump_to (platform_specific_descriptor fd) noexcept; // dump to opened file
size_t safe_dump_to (size_t skip, size_t max_depth; platform_specific_descriptor fd) noexcept;

// capture stacktrace on exception throwing
const char* impl::current_exception_stacktrace() noexcept; // buffer of captured dump of current exception
bool& impl::ref_capture_stacktraces_at_throw() noexcept; // switch option for dumping stacktrace on throw
void impl::assert_no_pending_traces() noexcept; // assistant

void this_thread::set_capture_stacktraces_at_throw(bool enable=true) noexcept { ref_capture_stacktraces_at_throw() = enable; }
bool this_thread::get_capture_stacktraces_at_throw() noexcept { return ref_capture_stacktraces_at_throw(); }
```

* `collect` implemented for `noop`, `unwind` (libc `backtrace` or libunwind `_Unwind_Backtrace`), `msvc` (`RtlCaptureStackBackTrace`).
* `dump` implemented for `noop`, `posix`, `win` (file read/write).
* Auto capture stacktrace on exception throw:
  - `win`: implemention based on `AddVectoredExceptionHandler`, dump stacktrace then rethrow
  - `posix`: implemented based on `__cxa_allocate_exception`, dump stacktrace into additional space.

------
#### Configuration

* Header-only / static/dynamic linking: `BOOST_STACKTRACE_LINK`, `BOOST_STACKTRACE_DYN_LINK`
* `BOOST_STACKTRACE_USE_WINDBG`, `BOOST_STACKTRACE_USE_WINDBG_CACHED`, `BOOST_STACKTRACE_USE_BACKTRACE`, `BOOST_STACKTRACE_USE_ADDR2LINE`, `BOOST_STACKTRACE_USE_NOOP`, or nothing for `basic`

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`, `<boost/config/workaround.hpp>`

#### Boost.ContainerHash

* `<boost/container_hash/hash_fwd.hpp>`

#### Boost.Core

* `<boost/core/demangle.hpp>`
* `<boost/core/noncopyable.hpp>`
* `<boost/core/no_exceptions_support.hpp>`

#### Boost.Predef

* `<boost/predef.hpp>`

#### Boost.WinAPI

* `<boost/winapi/*.hpp>` - for windows targets

------
### Standard Facilities

Library: `<stacktrace>` (C++23)
