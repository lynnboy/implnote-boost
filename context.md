# Boost.Context

* lib: `boost/libs/context`
* repo: `boostorg/context`
* commit: `230d5fe`, 2025-09-24

------
#### Stack Allocation

```c++
static constexpr size_t cache_alignment{64}, cacheline_length{64}, prefetch_stride{4*cacheline_length};

struct stack_context {
  size_t size{0}; void* sp{nullptr}; 
  using segments_context = void*[SEGMENTS]; segments_context segments_ctx{}; // USE_SEGMENTED_STACKS
  unsigned valgrind_stack_id{0}; // USE_VALGRIND
};
struct stack_traits {
  static bool is_unbounded() noexcept;
  static size_t page_size() noexcept;
  static size_t default_size() noexcept;
  static size_t minimum_size() noexcept;
  static size_t maximum_size() noexcept;
};

struct preallocated { void* sp; size_t size, stack_context sctx; };

class basic_fixedsize_stack<Tr> {
  size_t size_;
public: using traits_type = Tr;
  ctor(size_t size=Tr::default_size()) noexcept : size_{size}{}
  stack_context allocate();
  void deallocate(stack_context& sctx) noexcept;
};
using fixedsize_stack = basic_fixedsize_stack<stack_traits>;
using default_stack = fixedsize_stack;

class basic_pooled_fixedsize_stack<Tr> {
  class storage {
    std::atomic<size_t> use_count_{0}; size_t stack_size_;
    pool<default_user_allocator_malloc_free> storage_;
  public: ctor(size_t stack_size, size_t next_size, size_t max_size)
    : use_count_{0}, stack_size_{stack_size}, storage_{stack_size, next_size, max_size}{}
    stack_context allocate();
    void deallocate(stack_context& sctx) noexcept;
    friend void intrusive_ptr_add_ref(storage* s) noexcept;
    friend void intrusive_ptr_release(storage* s) noexcept;
  };
  intrusive_ptr<storage> storage_;
public: using traits_type = Tr;
  ctor(size_t stack_size=Tr::default_size(), size_t next_size=32, size_t max_size=0) noexcept;
  stack_context allocate();
  void deallocate(stack_context& sctx) noexcept;
};
using pooled_fixedsize_stack = basic_pooled_fixedsize_stack<stack_traits>;

class basic_protected_fixedsize_stack<Tr> {
  size_t size_;
public: using traits_type = Tr;
  ctor(size_t size=Tr::default_size()) noexcept : size_{size}{}
  stack_context allocate(); // VirtualProtect on Windows, mprotect on Posix
  void deallocate(stack_context& sctx) noexcept;
};
using protected_fixedsize_stack = basic_protected_fixedsize_stack<stack_traits>;

class basic_segmented_stack<Tr> { // Posix only
  size_t size_;
public: using traits_type = Tr;
  ctor(size_t size=Tr::default_size()) noexcept : size_{size}{}
  stack_context allocate(); // __splitstack_XXX API
  void deallocate(stack_context& sctx) noexcept;
};
using segmented_stack = basic_segmented_stack<stack_traits>;
```

------
### Fiber

#### fcontext version

```c++
using detail::fcontext_t = void*;
struct detail::transfer_t { fcontext_t fctx; void* data; };
transfer_t detail::jump_fcontext(fcontext_t const to, void* vp);
fcontext_t detail::make_fcontext( void * sp, std::size_t size, void (* fn)( transfer_t) );
transfer_t detail::ontop_fcontext( fcontext_t const to, void * vp, transfer_t (* fn)( transfer_t) );

struct detail::manage_exception_state{};
transfer_t detail::fiber_unwind(transfer_t t);
transfer_t detail::fiber_exit<Rec>(transfer_t t) noexcept;
void detail::fiber_entry<Rec>(transfer_t t) noexcept;
transfer_t detail::fiber_ontop<Ctx,Fn>(transfer_t t);

class detail::fiber_record<Ctx,StackAlloc,Fn> {
  stack_context sctx_; std::decay_t<StackAlloc> salloc_; std::decay_t<Fn> fn_;
  static void destroy(self* p) noexcept;
public: ctor(stack_context sctx, StackAlloc&& salloc, Fn&& fn) noexcept; // no copy
  void deallocate() noexcept { destroy(this); }
  fcontext_t run(fcontext_t fctx);
};
fcontext_t detail::create_fiber1<Record,StackAlloc,Fn>(StackAlloc&& salloc, Fn&& fn);
fcontext_t detail::create_fiber2<Record,StackAlloc,Fn>(preallocated palloc, StackAlloc&& salloc, Fn&& fn);

class fiber {
  fcontext_t fctx_{nullptr};
  ctor(fcontext_t fctx) noexcept;
public: ctor() noexcept=default;
  ctor<Fn>(Fn&& fn);
  ctor<StackAlloc, Fn>(std::allocator_arg_t, StackAlloc&& salloc, Fn&& fn);
  ctor<StackAlloc, Fn>(std::allocator_arg_t, preallocated palloc, StackAlloc&& salloc, Fn&& fn);
  ctor<Fn>(std::allocator_arg_t, segmented_stack, Fn&&); // for segmented stacks
  ctor<StackAlloc,Fn>(std::allocator_arg_t, preallocated, segmented_stack, Fn&&); // for segmented stacks
  ~dtor(); // and allow move, disable copy
  void swap(self& other) noexcept;

  fiber resume() &&; fiber resume_with<Fn>(Fn&& fn) &&;
  explicit operator bool() const noexcept; bool operator!() const noexcept;
  bool operator<(self const& other) const noexcept;

  friend std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os, fiber const& other);
};
void swap(fiber& l, fiber& r) noexcept;
using fiber_context = fiber;
```

#### WinFiber version

```c++
static VOID detail::fiber_entry_func<Record>(LPVOID data) noexcept;
struct detail::fiber_activation_record { // no copy
  LPVOID fiber{nullptr}; stack_context sctx{}; bool main_ctx{true};
  self* from{nullptr}; bool terminated{false}, force_unwind{false};
  static self*& current() noexcept;
  ctor() noexcept; // IsThreadFiber? GetCurrentFiber : ConvertThreadToFiber
  ctor(stack_context sctx_) noexcept;
  virtual ~dtor(); // ConvertFiberToThread / DeleteFiber

  bool is_main_context() const noexcept;
  self* resume(); // SwitchToFiber
  self* resume_with<Ctx,Fn>(Fn&& fn); // SwitchToFiber
};
struct detail::fiber_activation_record_initializer;
struct detail::forced_unwind{ fiber_activation_record* from{nullptr}; };
class detail::fiber_capture_record<Ctx,StackAlloc,Fn> : public fiber_activation_record {
  std::decay_t<StackAlloc> salloc_; std::decay_t<Fn> fn_;
  static void destroy(self* p) noexcept;
public: ctor(stack_context sctx, StackAlloc&& salloc, Fn&& fn) noexcept;
  void deallocate() noexcept override final;
  void run(); // invoke fn_, resume
};

static fiber_activation_record* detail::create_fiber1<Ctx,StackAlloc,Fn>(StackAlloc&& salloc, Fn&& fn);
static fiber_activation_record* detail::create_fiber2<Ctx,StackAlloc,Fn>(preallocated palloc, StackAlloc&& salloc, Fn&& fn);

class fiber {
  fiber_activation_record* ptr_{nullptr};
  ctor(fiber_activation_record* ptr) noexcept;
public: ctor() noexcept=default;
  ctor<Fn>(Fn&& fn);
  ctor<StackAlloc, Fn>(std::allocator_arg_t, StackAlloc&& salloc, Fn&& fn);
  ctor<StackAlloc, Fn>(std::allocator_arg_t, preallocated palloc, StackAlloc&& salloc, Fn&& fn);
  ~dtor(); // and allow move, disable copy
  void swap(self& other) noexcept;

  fiber resume() &&; fiber resume_with<Fn>(Fn&& fn) &&;
  explicit operator bool() const noexcept; bool operator!() const noexcept;
  bool operator<(self const& other) const noexcept;

  friend std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os, fiber const& other);
};
void swap(fiber& l, fiber& r) noexcept;
using fiber_context = fiber;
```

#### ucontext version

```c++
static void detail::fiber_entry_func<Record>(uint32_t data_high, uint32_t data_low) noexcept; // macos
static void detail::fiber_entry_func<Record>(void* data) noexcept; // otherwise
struct detail::fiber_activation_record { // no copy
  ucontext_t uctx{}; stack_context sctx{}; bool main_ctx{true};
  self* from{nullptr}; std::function<self*(self*&)> ontop{};
  bool terminated{false}, force_unwind{false};
  void *fake_stack{nullptr}, *stack_bottom{nullptr}; size_t stack_size{0}; // USE_ASAN
  void *tsan_fiber{nullptr}; bool destroy_tsan_fiber{true}; // USE_TSAN
  static self*& current() noexcept;
  ctor(); // getcontext
  ctor(stack_context sctx_) noexcept;
  virtual ~dtor();

  bool is_main_context() const noexcept;
  self* resume(); // swapcontext
  self* resume_with<Ctx,Fn>(Fn&& fn); // swapcontext
};
struct detail::fiber_activation_record_initializer;
struct detail::forced_unwind{ fiber_activation_record* from{nullptr}; };
class detail::fiber_capture_record<Ctx,StackAlloc,Fn> : public fiber_activation_record {
  std::decay_t<StackAlloc> salloc_; std::decay_t<Fn> fn_;
  static void destroy(self* p) noexcept;
public: ctor(stack_context sctx, StackAlloc&& salloc, Fn&& fn) noexcept;
  void deallocate() noexcept override final;
  void run(); // invoke fn_, resume
};

static fiber_activation_record* detail::create_fiber1<Ctx,StackAlloc,Fn>(StackAlloc&& salloc, Fn&& fn);
static fiber_activation_record* detail::create_fiber2<Ctx,StackAlloc,Fn>(preallocated palloc, StackAlloc&& salloc, Fn&& fn);

class fiber {
  fiber_activation_record* ptr_{nullptr};
  ctor(fiber_activation_record* ptr) noexcept;
public: ctor() noexcept=default;
  ctor<Fn>(Fn&& fn);
  ctor<StackAlloc, Fn>(std::allocator_arg_t, StackAlloc&& salloc, Fn&& fn);
  ctor<StackAlloc, Fn>(std::allocator_arg_t, preallocated palloc, StackAlloc&& salloc, Fn&& fn);
  ~dtor(); // and allow move, disable copy
  void swap(self& other) noexcept;

  fiber resume() &&; fiber resume_with<Fn>(Fn&& fn) &&;
  explicit operator bool() const noexcept; bool operator!() const noexcept;
  bool operator<(self const& other) const noexcept;

  friend std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os, fiber const& other);
};
void swap(fiber& l, fiber& r) noexcept;
using fiber_context = fiber;
```

------
### Continuation

#### fcontext version

```c++
transfer_t detail::context_unwind(transfer_t t);
transfer_t detail::context_exit<Rec>(transfer_t t) noexcept;
void detail::context_entry<Rec>(transfer_t t) noexcept;
transfer_t detail::context_ontop<Ctx,Fn>(transfer_t t);

class detail::record<Ctx,StackAlloc,Fn> { // no copy
  stack_context sctx_; std::decay_t<StackAlloc> salloc_; std::decay_t<Fn> fn_;
  static void destroy(record* p) noexcept;
public: ctor(stack_context sctx, StackAlloc&& salloc, Fn&& fn) noexcept;
  void deallocate() noexcept { destroy(this); }
  fcontext_t run(fcontext_t fctx); // invoke fn_, resume
};
fcontext_t detail::create_context1<Record,StackAlloc,Fn>(StackAlloc&& salloc, Fn&& fn);
fcontext_t detail::create_context2<Record,StackAlloc,Fn>(preallocated palloc, StackAlloc&& salloc, Fn&& fn);

class continuation {
  fcontext_t fctx_{nullptr};
  ctor(fcontext_t fctx) noexcept;
public: ctor() noexcept=default; ~dtor(); // allow move, no copy
  self resume() {&|&&}; // jump_fcontext
  self resume_with<Fn>(Fn&& fn) {&|&&};
  explicit operator bool() const noexcept; bool operator!() const noexcept;
  bool operator<(self const& other) const noexcept;
  std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os, self& const other);
  void swap(self& other) noexcept;
};
continuation callcc<Fn>(Fn&& fn);
continuation callcc<StackAlloc,Fn>(std::allocator_arg_t, StackAlloc&& salloc, Fn&& fn);
continuation callcc<StackAlloc,Fn>(std::allocator_arg_t, preallocated palloc, StackAlloc&& salloc, Fn&& fn);
continuation callcc<Fn>(std::allocator_arg_t, segmented_stack, Fn&&); // SEGMENTED_STACKS
continuation callcc<StackAlloc,Fn>(std::allocator_arg_t, preallocated, segmented_stack, Fn&&); // SEGMENTED_STACKS
void swap(continuation& l, continuation& r) noexcept;
```

#### WinFiber version

```c++
static VOID detail::entry_func<Record>(LPVOID data) noexcept;
struct detail::activation_record { // no copy
  LPVOID fiber{nullptr}; stack_context sctx{}; bool main_ctx{true};
  self* from{nullptr}; std::function<self*(self*&)> ontop{};
  bool terminated{false}, force_unwind{false};
  static self*& current() noexcept;
  ctor() noexcept; // IsThreadAFiber ? GetCurrentFiber : ConvertThreadToFiber
  ctor(stack_context sctx_) noexcept;
  virtual ~dtor();
  bool is_main_context() const noexcept;
  self* resume(); // SwitchToFiber
  virtual void deallocate() noexcept {}
};
struct detail::forced_unwind{ activation_record* from{nullptr}; };
class detail::capture_record<Ctx,StackAlloc,Fn> : public activation_record {
  std::decay_t<StackAlloc> salloc_; std::decay_t<Fn> fn_;
  static void destroy(self* p) noexcept;
public: ctor(stack_context sctx, StackAlloc&& salloc, Fn&& fn) noexcept;
  void deallocate() noexcept override final { destroy(this); }
  void run(); // invoke, resume()
};
activation_record* detail::create_context1<Ctx,StackAlloc,Fn>(StackAlloc&& salloc, Fn&& fn);
activation_record* detail::create_context2<Ctx,StackAlloc,Fn>(preallocated palloc, StackAlloc&& salloc, Fn&& fn);

class continuation {
  fcontext_t fctx_{nullptr};
  ctor(fcontext_t fctx) noexcept;
public: ctor() noexcept=default; ~dtor(); // allow move, no copy
  self resume() {&|&&}; // jump_fcontext
  self resume_with<Fn>(Fn&& fn) {&|&&};
  explicit operator bool() const noexcept; bool operator!() const noexcept;
  bool operator<(self const& other) const noexcept;
  std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os, self& const other);
  void swap(self& other) noexcept;
};
continuation callcc<Fn>(Fn&& fn);
continuation callcc<StackAlloc,Fn>(std::allocator_arg_t, StackAlloc&& salloc, Fn&& fn);
continuation callcc<StackAlloc,Fn>(std::allocator_arg_t, preallocated palloc, StackAlloc&& salloc, Fn&& fn);
void swap(continuation& l, continuation& r) noexcept;
```

#### ucontext version

```c++
static void detail::entry_func<Record>(uint32_t data_high, uint32_t data_low) noexcept; // macos
static void detail::entry_func<Record>(void* data) noexcept; // otherwise
struct detail::activation_record { // no copy
  ucontext_t uctx{}; stack_context sctx{}; bool main_ctx{true};
  self* from{nullptr}; std::function<self*(self*&)> ontop{};
  bool terminated{false}, force_unwind{false};
  void *fake_stack{nullptr}, *stack_bottom{nullptr}; size_t stack_size{0} // USE_ASAN
  static self*& current() noexcept;
  ctor(); // getcontext
  ctor(stack_context sctx_) noexcept;
  virtual ~dtor(){}
  bool is_main_context() const noexcept;
  self* resume(); // swapcontext
  virtual void deallocate() noexcept {}
};
struct detail::activation_record_initializer{};
struct detail::forced_unwind{ activation_record* from{nullptr}; };
class detail::capture_record<Ctx,StackAlloc,Fn> : public activation_record {
  std::decay_t<StackAlloc> salloc_; std::decay_t<Fn> fn_;
  static void destroy(self* p) noexcept;
public: ctor(stack_context sctx, StackAlloc&& salloc, Fn&& fn) noexcept;
  void deallocate() noexcept override final { destroy(this); }
  void run(); // invoke, resume()
};
activation_record* detail::create_context1<Ctx,StackAlloc,Fn>(StackAlloc&& salloc, Fn&& fn);
activation_record* detail::create_context2<Ctx,StackAlloc,Fn>(preallocated palloc, StackAlloc&& salloc, Fn&& fn);

class continuation {
  fcontext_t fctx_{nullptr};
  ctor(fcontext_t fctx) noexcept;
public: ctor() noexcept=default; ~dtor(); // allow move, no copy
  self resume() {&|&&}; // jump_fcontext
  self resume_with<Fn>(Fn&& fn) {&|&&};
  explicit operator bool() const noexcept; bool operator!() const noexcept;
  bool operator<(self const& other) const noexcept;
  std::basic_ostream<Ch,Tr>& operator<< <Ch,Tr> (std::basic_ostream<Ch,Tr>& os, self& const other);
  void swap(self& other) noexcept;
};
continuation callcc<Fn>(Fn&& fn);
continuation callcc<StackAlloc,Fn>(std::allocator_arg_t, StackAlloc&& salloc, Fn&& fn);
continuation callcc<StackAlloc,Fn>(std::allocator_arg_t, preallocated palloc, StackAlloc&& salloc, Fn&& fn);
void swap(continuation& l, continuation& r) noexcept;
```

------
### Configuration

* `BOOST_USE_SEGMENTED_STACKS`
* `SEGMENTS`: 10
* `USE_UCONTEXT` use `ucontext`, `USE_WINFIB`, use `WinFiber`, or default use `fcontext` version

------
### Dependency

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/detail/workaround.hpp>`
* `<boost/config/auto_link.hpp>`
* `<boost/cstdint.hpp>`

#### Boost.Core

* `<boost/core/ignore_unused.hpp>`

#### Boost.MP11

* `<boost/mp11/integer_sequence.hpp>`

#### Boost.Pool

* `<boost/pool/pool.hpp>`

#### Boost.PreDef

* `<boost/predef/os.h>`
* `<boost/predef.h>`

#### Boost.SmartPtr

* `<boost/intrusive_ptr.hpp>`

------
### Standard Facilities
