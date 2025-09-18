# Boost.Signals

* lib: `boost/libs/signals`
* repo: `boostorg/signals`
* commit: `f898edf`, 2019-04-27

------
### Overview

Header `<boost/signals.hpp>`

_Deprecated by **Boost.Signals2**_

* Having similarity with *Boost.Function* to treat old compiler on signatures,
  provides both `signal<Signature>` and `signalN<R, ... Arg>` versions (supports upto 10 args).
* `signal` can connect with functors, even other signals, as slots, returning `connection`.
* `connection` can be used to disconnect or block separately.
* Slots binding on a `signal` supports priority grouping.
* A `trackable` object bound with a signal will auto disconnect when destructed.
* Returning value of each slot will be handled by a combiner, by default it use `last_value`
* The actual slot invocation is driven by the combiner in a _pull_ model on an iterator manner.

------
### Connection

```c++
struct bound_object { // supports `==` and `<`
  void* obj;      void* data;
  void (*disconnect)(void*, void*);
};
struct basic_connection {   // shared connection data
  void* signal;   void* signal_data;
  void (*signal_disconnect)(void*, void*);  // nullptr means disconnected, set to nullptr before call
  bool blocked_;
  std::list<bound_object> bound_objects;    // call 'disconnect' for each one when disconnected
};
class connection {
  shared_ptr<basic_connection> con;
  bool controlling_connection;  // default false, cause disconnect on destroy
public:
  // default ctor, copy-ctor, dtor, copy-assign, swap
  // equality and comparasion "==", "<", etc., comparing the shared_ptr pointer value
  void block(bool=true);    void unblock();   bool blocked() const;
  void disconnect() const;                    bool connected() const;
};
class scoped_connection : public connection { // disconnect on destroy
  bool released;
public:
  // default ctor, copy-ctor, dtor, copy-assign, swap
  connection release(); // don't disconnect on destroy
};

using connection_slot_pair = pair<connection, any>;     // comparisons are meaningless
auto is_disconnected = [](connection_slot_pair const &);
auto is_callable = [](connection_slot_pair const &);    // connected and not blocked
```

* A `connection` is a wrapper for a shared connection data.
* Every object requiring disconnect notification is remembered in the connection data.
* On disconnection, the `signal` and every tracking objects are called-back.

------
### Slot Grouping

```c++
enum connect_position { at_back, at_front };
class stored_group {
  enum storage_kind { sk_empty, sk_front, sk_back, sk_group } kind;
  shared_ptr<void> group; // actual group data
public:
  bool is_front() const; bool is_back() const; bool empty() const; void* get() const;
};
auto group_bridge_compare<Compare> = [Compare comp = c](const stored_group&, const stored_group&)
  {/*...*/} // 'is_front' < 'comp' < 'is_back'

using compare_type = function<bool(stored_group, stored_group)>;
class named_slot_map {
  using slot_container_type = std::map<stored_group, std::list<connection_slot_pair>, compare_type>;
  slot_container_type groups; // store in map
  
public:
  typedef named_slot_map_iterator iterator;   // flatten the grouped slots
  named_slot_map(compare_type const& compare);
  void clear();
  iterator begin(); iterator end();

  iterator insert(const stored_group& name,   // empty to specify whole list
                  const connection& con, const any& slot, connect_position at);
  void disconnect(const stored_group&);   void erase(iterator); // disconnect and erase
  void remove_disconnected_slots();
};
class named_slot_map_iterator : iterator_facade<...> {
  slot_container_type::iterator group, last_group; // current and end
  optional<std::list<connection_slot_pair>::iterator> slot_; // no value when at end
public:
  // ctor, copy-ctor, copy-assign
  // 'dereference', 'increment', 'equal'
};
```

* Slots and `connection` are stored in a priority list, implemented by a `std::map`
* Removing from the list will cause disconnect.

------
### Slot And Trackable Bindings

```c++
class trackable {
  static void signal_disconnected(void*, void*);  // notification callback, remove the connection from list
  void signal_connected(connection c, bound_object& b) const {
    auto pos = connected_signals.insert(connected_signals.end(), c);
    b.obj = this; b.data = pos; b.disconnect = signal_disconnected;
  }
  std::list<connection> connected_signals;  // all connection bound for this object
  bool dying; // flag to prevent recursive call
protected:
  // ctor, copy-ctor, dtor, copy-assign
};

auto bound_objects_visitor = [std::vector<const trackable*>& v=v](const auto& t)
  { /*...*/ } // unwrap 'reference_wrapper', push every 'trackable' into 'v'

class slot_base {
  struct data_t {
    std::vector<const trackable*> bound_objects;  // only used during connection creating
    connection watch_bound_objects;   // controlling by default
  };
  shared_ptr<data_t> data;
  bool is_active() const; // connection is connected
  static void bound_object_destructed(void*, void*);
}
template <typename SlotFunction>  // SlotFunction is 'function<>'
class slot : public slot_base {   // non-copyable
  SlotFunction slot_function;
public:
  template <typename F> slot(F const& f);   // implicit converting constructor
  SlotFunction const& get_slot_function() const;
  void release() const { data->watch_bound_objects.set_controlling(false); } // dont auto disconnect
}
```

* `slot` can be implicit converted from any matching object, by default it is a `function`
* `signal` instances are wrapped by `reference_wrapper` in `slot` storage
* On `slot` construction, all `trackable` are collected by `bound_objects_visitor`.
* Using `visit_each` to support *Boost.Bind* and provide customization.
* The `watch_bound_objects` by default controlls disconnection, a `slot` _owns_ a connection in concept.
* The `connection`s remembered by `trackable` are all `controlling` ones, thus destruction of `trackable`
  will disconnect all of them.

------
### Connecting Signal And Slots

```c++
class signal_base_impl {
  int call_depth;             // detecting recursive calls
  struct { bool delayed_disconnect, clearing; } flags;
  named_slot_map slots_;      // all connected slots
  any combiner_;
  
  void disconnect(stored_group const&);

  static void slot_disconnected(void*, void*);    // notify callback
protected:
  // ctor, copy-ctor, dtor, copy-assign
};

class signal_base : public noncopyable {  // noncopyable by default
protected:
  shared_ptr<signal_base_impl> impl;      // lightweight wrapper over slots data
public:
  signal_base(compare_type const& comp, any const& combiner);
  ~signal_base();

  bool empty() const;
  std::size_t num_slots() const;
  void disconnect_all_slots();

protected:
  connection connect_slot(any const& slot_, stored_group const& name,
                          shared_ptr<slot_base::data_t> data, connect_position at) {
    auto pos = slots_.insert(name, data->watch_bound_objects, slot_, at);
    pos->first.get_connection()->signal = this; pos->first.get_connection()->signal_data = pos;
    pos->first.get_connection()->signal_disconnect = signal_base_impl::slot_disconnected;
    pos->first.set_controlling(); data->watch_bound_objects.set_controlling(false);
  }
}
```

* Recursive call is detected, disconnected slots are remove only at top recursive level.
* Both `disconnect` functions and `slot_disconnected` will either remove slot from `slots_` or set `delayed_disconnect`.
* When a `slot` is connected to `signal`, ownership is taken, notification is retargeted from the slot to the signal.

------
### Signal API And Invocation

```c++
template <typename T> struct last_value; // functor to return last value from a range

template <typename Function, typename Iterator>
class slot_call_iterator : public iterator_facade<...> {  // cached invocation on slots
  Iterator iter, end;
  Function f;
  optional<result_type>* cache; // refer to stack object
public:
  slot_call_iterator(Iterator iter, Iterator end, Function func, optional<result_type>&);
  reference derefrence() const; // store 'f(*iter)' if not already cached
  void increment();             // reset cache, filter 'iter' by 'is_callable'
  bool equal(slot_call_iterator const &);
};

template <typename Signature, // R(Args...)
  typename Combiner = last_value<R>,
  typename Group = int, typename GroupCompare = std::less<Group>,
  typename SlotFunction = function<Signature> >
class signal : public signal_base, public trackable {
public:
  typedef SlotFunction slot_function_type;
  typedef slot<slot_function_type> slot_type;
  typedef conditional_t<is_void_v<R>, unusable, R> slot_result_type;
  // typedef ArgN argN_type;
  typedef slot_call_iterator<_functor_,   // unsafe_any_cast from slot and call with bound arguments
    named_slot_map::iterator>   slot_call_iterator;

  explicit signal(Combiner const& = Combiner(), GroupCompare const& = GroupCompare());
  Combiner [const] & combiner() [const];  // unsafe_any_cast
  
  connection connect([group_type const&,] slot_type const&, connect_position = at_back);
  template <typename T> void disconnect(T const&); // 'T' may be group name or a functor
  
  result_type operator()(Args...args) [const] {
    // remember call depth
    auto f = [=](auto const& slot) {
      F* target = unsafe_any_cast<F>(&slot.second);
      return (*target)(args...);
    };
    optional<slot_result_type> cache;
    return Combiner()(slot_call_iterator(slots_.begin(), slots_.end(), f, cache),
                      slot_call_iterator(slots_.end(), slots_.end(), f, cache));
  }
};
```

* `GroupCompare` is wrapped by `group_bridge_compare`.
* If argument for `disconnect` is a functor, compare it with each value of `slot` target.
* On `connect`, callable objects are converted to `slot` implicitly.

------
### Dependency

#### Boost.Any

* `<boost/any.hpp>`

#### Boost.Config

* `<boost/config.hpp>`
* `<boost/config/auto_link.hpp>` - for library naming

#### Boost.Core

* `<boost/noncopyable.hpp>`
* `<boost/utility/addressof.hpp>`
* `<boost/ref.hpp>`
* `<boost/visit_each.hpp>`

#### Boost.Iterator

* `<boost/iterator/iterator_facade.hpp>`

#### Boost.Function

* `<boost/function.hpp>`

#### Boost.MPL

* `<boost/mpl/bool.hpp>`

#### Boost.Optional

* `<boost/optional.hpp>`

#### Boost.SmartPtr

* `<boost/shared_ptr.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/conversion_traits.hpp>`
* `<boost/type_traits/function_traits.hpp>`
* `<boost/type_traits/is_convertible.hpp>`
* `<boost/type_traits.hpp>`

#### Boost.Utility

* `<boost/operators.hpp>`

------
### Standard Facilities
