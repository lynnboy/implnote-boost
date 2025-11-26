# Boost.MQTT5

* lib: `boost/libs/mqtt5`
* repo: `boostorg/mqtt5`
* commit: `150ba94`, 2025-09-19

------
### Common Parts

##### Errors

```c++
enum class disconnect_rc_e : uint8_t { normal_disconnect=0, disconnect_with_will_message=4 };
enum class detail::disconnect_rc_e : uint8_t { normal_disconnect=0, disconnect_with_will_message=4, unspecified_error=0x80, malformed_packet, protocol_error, implementation_specific_error,
    topic_name_invalid=0x90, receive_maximum_exceeded=0x93, topic_alias_invalid, packet_too_large, message_rate_too_high, quota_exceeded, administrative_action, payload_format_invalid };
enum class client::error : int { malformed_packet = 100, packet_too_large, session_expired, pid_overrun, invalid_topic, qos_not_supported,
    retain_not_available, topic_alias_maximum_reached, wildcard_subscription_not_available, subscription_identifier_not_available, shared_subscription_not_available };
std::string client::client_error_to_string(error err);
struct client::client_ec_category : system::error_category {
    const char* name() const noexcept override { return "mqtt_client_error"; }
    std::string message(int ev) const noexcept override { return client_error_to_string((error)ev); }
};
const client_ec_category& client::get_error_code_category() { static client_ec_category cat; return cat; }
system::error_code client::make_error_code(error r) { return {(int)r, get_error_code_category()}; }
std::ostream& client::operator<<(std::ostream& os, const error& err) { return os << get_error_code_category().name() << ":" << (int)err; }
struct system::is_error_code_enum<client::error> : std::true_type {};
```

##### Reason Codes

```c++
enum class reason_codes::category : uint8_t { none, connack, puback, pubrec, pubrel, pubcomp, suback, unsuback, auth, disconnect };
class reason_code {
    uint8_t _code={0xff}; reason_codes::category _category={none};
public: constexpr ctor(); contexpr ctor(uint8_t code, category cat); const explicit ctor(uint8_t code);
    explicit operator bool() const noexcept { return _code >= 0x80; }
    constexpr uint8_t value() const noexcept { return _code; }
    friend std::ostream& operator<<(std::ostream& os, const self& rc) { return os << rc.message(); }
    friend bool operator< (const self& lhs, const self& rhs) { return lhs._code < rhs._code; }
    friend bool operator== (const self& lhs, const self& rhs) { return lhs._code == rhs._code && lhs._category == rhs._category; }
    std::string message() const;
};
constexpr reason_code reason_codes::empty{},/*0xff, invalid*/
    success{0x00}, normal_disconnection{0x00, disconnect}, granted_qos_0{0x00,suback}, granted_qos_1{0x01}, granted_qos_2{0x02},
    disconnect_with_will_message{0x04}, no_matching_subscribers{0x10}, no_subscription_existed{0x11}, continue_authentication{0x18}, reauthenticate{0x19},
    unspecified_error{0x80}, malformed_packet{0x81}, protocol_error{0x82}, implementation_specific_error{0x83}, unsupported_protocol_version{0x84},
    client_identifier_not_valid{0x85}, bad_username_or_password{0x86}, not_authorized{0x87}, server_unavailable{0x88}, server_busy{0x89}, banned{0x8a},
    server_shutting_down{0x8b}, bad_authentication_method{0x8c}, keep_alive_timeout{0x8d}, session_taken_over{0x8e}, topic_filter_invalid{0x8f},
    topic_name_invalid{0x90}, packet_identifier_in_use{0x91}, packet_identifier_not_found{0x92}, receive_maximum_exceeded{0x93}, topic_alias_invalid{0x94},
    packet_too_large{0x95}, message_rate_too_high{0x96}, quota_exceeded{0x97}, administrative_action{0x98}, payload_format_invalid{0x99},
    retain_not_supported{0x9a}, qos_not_supported{0x9b}, use_another_server{0x9c}, server_moved{0x9d}, shared_subscription_not_supported{0x9e},
    connection_rate_exceeded{0x9f}, maximum_connect_time{0xa0}, subscription_ids_not_supported{0xa1}, wildcard_subscription_not_supported{0xa2};
std::pair<reason_code*, size_t> reason_codes::detail::valid_codes<category cat>() { constexpr switch(cat)
    case connack: static reason_code valid_codes[] = {
        success, unspecified_error, malformed_packet, protocol_error, implementation_specific_error, unsupported_protocol_version, client_identifier_not_valid,
        bad_username_or_password, not_authorized, server_unavailable, server_busy, banned, bad_authentication_method, topic_name_invalid, packet_too_large, quota_exceeded,
        payload_format_invalid, retain_not_supported, qos_not_supported, use_another_server, server_moved, connection_rate_exceeded };
    case auth: static reason_code valid_codes[] = { success, continue_authentication, reauthenticate };
    case puback: case pubrec: static reason_code valid_codes[] = {
        success, no_matching_subscribers, unspecified_error, implementation_specific_error, not_authorized,
        topic_name_invalid, packet_identifier_in_use, quota_exceeded, payload_format_invalid };
    case pubrel: case pubcomp: static reason_code valid_codes[] = { success, packet_identifier_not_found };
    case suback: static reason_code valid_codes[] = {
        granted_qos_0, granted_qos_1, granted_qos_2, unspecified_error, implementation_specific_error, not_authorized, topic_filter_invalid,
        packet_identifier_in_use, quota_exceeded, shared_subscription_not_supported, subscription_ids_not_supported, wildcard_subscription_not_supported };
    case unsuback: static reason_code valid_codes[] = {
        success, no_subscription_existed, unspecified_error, implementation_specific_error, not_authorized, topic_filter_invalid, packet_identifier_in_use };
    case disconnect: static reason_code valid_codes[] = {
        normal_disconnection, unspecified_error, malformed_packet, protocol_error, implementation_specific_error, not_authorized, server_busy, server_shutting_down,
        keep_alive_timeout, session_taken_over, topic_filter_invalid, topic_name_invalid, receive_maximum_exceeded, topic_alias_invalid, packet_too_large,
        message_rate_too_high, quota_exceeded, administrative_action, payload_format_invalid, retain_not_supported, qos_not_supported, use_another_server, server_moved,
        shared_subscription_not_supported, connection_rate_exceeded, maximum_connect_time, subscription_ids_not_supported, wildcard_subscription_not_supported };
    return std::make_pair(valid_codes, sizeof(valid_codes)/sizeof(reason_code));
}
std::optional<reason_code> to_reason_code<reason_codes::category cat>(uint8_t code); // find in valid_codes<cat>(); otherwise nullopt
```

##### Property Types

```c++
namespace prop;
enum property_type : uint8_t { payload_format_indicator_t=1 /*u8*/, message_expiry_interval_t /*u32*/, content_type_t /*str*/, response_topic_t=0x8 /*str*/, correlation_data_t /*str*/,
    subscription_identifier_t=0xb, session_expiry_interval_t=0x11 /*u32*/, assigned_client_identifier_t /*str*/, server_keep_alive_t /*u16*/,
    authentication_method_t /*str*/, authentication_data_t /*str*/, request_problem_information_t /*u8*/, will_delay_interval_t /*u32*/, request_response_information_t /*u8*/, response_information_t /*str*/,
    server_reference_t=0x1c /*str*/, reason_string_t=0x1f /*str*/, receive_maximum_t=0x21 /*u16*/, topic_alias_maximum_t /*u16*/, topic_alias_t /*u16*/, maximum_qos_t /*u8*/, retain_available_t /*u8*/,
    user_property_t, maximum_packet_size_t /*u32*/, wildcard_subscription_available_t /*u8*/, subscription_identifier_available_t /*u8*/, shared_subscription_available_t /*u8*/ };

class alignas(8) subscription_identifiers : public container::small_vector<int32_t, 1> {
public: using base::ctor; ctor(int32_t v);
    bool has_value() const noexcept { return !empty(); } explicit operator bool() const noexcept { return !empty(); }
    int32_t& operator*() noexcept { return front(); } int32_t operator*() const noexcept { return front(); }
    void emplace(int32_t val=0) { *this = val; }
    int32_t value() const { return front(); } int32_t value_or(int32_t def) const noexcept { return empty() ? def : front(); }
    void reset() noexcept { clear(); }
};
using user_property_value_t = std::vector<std::pair<std::string, std::string>>;

struct property_traits<property_type p> { // specialize for each property_type enumerators
    static constexpr std::string_view name = <NAME>; // without `_t`
    using type = <TYPE>; // subscription_identifier is subscription_identifiers, user_property is user_property_value_t,
        // others are std::optional<> of uint8/uint16/uint32/std::string
};
using value_type_t<property_type p> = property_traits<p>::type;
const std::string_view name_v<property_type p> = property_traits<p>::name;

class properties<property_type ...Ps> {
    struct property<property_type p> {
        using key = std::integral_constant<property_type, p>;
        constexpr static std::string_view name = name_v<p>;
        value_type_t<p> value;
    };
    std::tuple<property<Ps>...> _props;
public:
    constexpr <const> auto& operator[]<property_type v>(std::integral_constant<property_type, v>) <const> noexcept { return std::get<property<v>>(_props).value; }
    constexpr bool apply_on<Func>(uint8_t property_id, Func&& func) noexcept (...) requires(...);
    constexpr bool visit<Func>(Func&& func) <const> noexcept(...) requires(...);
};
```

##### Data Types

```c++
struct authority_path { std::string host, port, path; };
enum class qos_e : uint8_t { at_most_once, at_least_once, exactly_once };
enum class retain_e : uint8_t { no, yes };
enum class dup_e : uint8_t { no, yes };
enum class auth_step_e { client_initial, server_challenge, server_final };
enum class no_local_e : uint8_t { no, yes };
enum class retain_as_published_e : uint8_t { dont, retain };
enum class retain_handling_e : uint8_t { send, new_subscription_only, not_send };
struct subscribe_options { qus_e max_qos{exactly_once}; no_local_e no_local{yes};
    retain_as_published_e retain_as_published{retain}; retain_handling_e retain_handling{new_subscription_only}; };
struct subscribe_topic { std::string topic_filter; subscribe_options sub_opts; };

class connect_props : public prop::properties<session_expiry_interval_t, receive_maximum_t, maximum_packet_size_t, topic_alias_maximum_t,
    request_response_information_t, request_problem_information_t, user_property_t, authentication_method_t, authentication_data_t>{};
class connack_props : public prop::properties<session_expiry_interval_t, receive_maximum_t, maximum_qos_t, retain_available_t, maximum_packet_size_t,
    assigned_client_identifier_t, topic_alias_maximum_t, reason_string_t, user_property_t, wildcard_subscription_available_t subscription_identifier_available_t,
    shared_subscription_available_t, server_keep_alive_t, response_information_t, server_reference_t, authentication_method_t, authentication_data_t>{};
class publish_props : public prop::properties<payload_format_indicator_t, message_expiry_interval_t, content_type_t,
    response_topic_t, correlation_data_t, subscription_identifier_t, topic_alias_t, user_property_t>{};
class puback_props : public prop::properties<reason_string_t, user_property_t>{};
class pubcomp_props : public prop::properties<reason_string_t, user_property_t>{};
class pubrec_props : public prop::properties<reason_string_t, user_property_t>{};
class pubrel_props : public prop::properties<reason_string_t, user_property_t>{};
class subscribe_props : public prop::properties<subscription_identifier_t, user_property_t>{};
class suback_props : public prop::properties<reason_string_t, user_property_t>{};
class unsubscribe_props : public prop::properties<user_property_t>{};
class unsuback_props : public prop::properties<reason_string_t, user_property_t>{};
class disconnect_props : public prop::properties<session_expiry_interval_t, reason_string_t, user_property_t, server_reference_t>{};
class auth_props : public prop::properties<authentication_method_t, authentication_data_t, reason_string_t, user_property_t>{};
class will_props : public prop::properties<will_delay_interval_t, payload_format_indicator_t, message_expiry_interval_t,
    content_type_t, response_topic_t, correlation_data_t, user_property_t>{};

class will : public will_props {
    std::string _topic, _message; qos_e _qos; retain_e _retain;
public: ctor(); ctor(std::string topic, std::string message, qos_e qos=at_most_once, retain_e retain=no, <will_props props>);
    std::string_view topic() const; std::string_view message() const; constexpr qos_e qos() const; constexpr retain_e retain() const;
};

using detail::byte_citer = std::string::const_iterator;
using detail::time_stamp = std::chrono::time_point<std::chrono::steady_clock>;
using detail::duration = time_stamp::duration;
struct detail::credentials {
    std::string client_id; std::optional<std::string> username; std::optional<std::string> password;
    ctor(); ctor(std::string client_id, std::string username, std::string password);
};
class detail::session_state {
    uint8_t _flags = 0;
    static constexpr uint8_t session_present_flag = 1, subscriptions_present_flag = 2;
public: void session_present(bool present); bool session_present() const;
    void subscriptions_present(bool present); bool subscriptions_present() const;
};
struct detail::mqtt_ctx {
    credentials creds; std::optional<will> will_msg; uint16_t keep_alive=60;
    connect_props co_props; connack_props ca_props; session_state state; any_authenticator authenticator;
    ctor(); ctor(const self&); // ca_props and state are not copied
};
using detail::serial_num_t = uint32_t;
constexpr serial_num_t detail::no_serial = 0;
enum class detail::send_flag { none=0, throttled=1, prioritized=2, terminal=4 };
```

##### Control Packet

```c++
static constexpr int32_t detail::default_max_send_size = 268'435'460, default_max_recv_size = 65'536;
enum class detail::control_code_e : uint8_t { no_packet=0, connect=0x10, connack=0x20, publish=0x30, puback=0x40, pubrec=0x50,
    pubrel=0x60, pubcomp=0x70, subscribe=0x80, suback=0x90, unsubscribe=0xa0, unsuback=0xb0, pingreq=0xc0, pingresp=0xd0, disconnect=0xe0, auth=0xf0 };
constexpr struct detail::with_pid_{} with_pid {};
constexpr struct detail::no_pid_{} no_pid {};
class detail::control_packet<Alloc> {
    uint16_t _packet_id; std::unique_ptr<std::string, boost::alloc_deleter<std::string, Alloc>> _packet;
    ctor(const Alloc& a, uint16_t packet_id, std::string packet) : _packet_id{packet_id}, _packet(allocate_unique<std::string>(a, std::move(packet))){}
public: // allow move, disable copy
    static self of<EncodeFun,...Args>(with_pid_, const Alloc& alloc, EncodeFun&& encode, uint16_t packet_id, Args&&...args);
    static self of<EncodeFun,...Args>(no_pid_, const Alloc& alloc, EncodeFun&& encode, Args&&...args);
    size_t size() const { return _packet->size(); }
    control_code_e control_code() const { return {(*_packet->data()) & 0xF0}; }
    uint16_t packet_id() const { return _packet_id; }
    qos_e qos() const { return {(*_packet->data()) & 0x06 >> 1}; }
    self& set_dup() { (*_packet->data()) |= 0x08; return *this; }
    std::string_view wire_data() const { return *_packet; }
};
class detail::packet_id_allocator {
    struct interval { uint16_t start, end; };
    std::vector<interval> _free_ids{{MAX_PACKET_ID, (uint16_t)0};
    static constexpr uint16_t MAX_PACKET_ID = 65535;
public: ctor(); // allow move, disable copy
    uint16_t allocate(); void free(uint16_t pid);
};
```

##### Logging

```c++
class noop_logger{};
constexpr bool has_at_resolve<T>; // detect T::at_resolve(error_code, std::string_view, std::string_view, const asio::ip::tcp::resolver::results_type&)
constexpr bool has_at_tcp_connect<T>; // detect T::at_tcp_connect(error_code, asio::ip::tcp::endpoint)
constexpr bool has_at_tls_handshake<T>; // detect T::at_tls_handshake(error_code, asio::ip::tcp::endpoint)
constexpr bool has_at_ws_handshake<T>; // detect T::at_ws_handshake(error_code, asio::ip::tcp::endpoint)
constexpr bool has_at_connack<T>; // detect T::at_connack(reason_code, bool, const connack_props&)
constexpr bool has_at_disconnect<T>; // detect T::at_disconnect(reason_code, const disconnect_props&)

enum class log_level : uint8_t { error=1, warning, info, debug };
class logger { constexpr static auto prefix = "[Boost.MQTT5]"; log_level _level;
public: ctor(log_level level=info);
    void at_resolve(error_code ec, std::string_view host, std::string_view port, const asio::ip::tcp::resolver::results_type& eps);
    void at_tcp_connect(error_code ec, asio::ip::tcp::endpoint ep);
    void at_tls_handshake(error_code ec, asio::ip::tcp::endpoint ep);
    void at_ws_handshake(error_code ec, asio::ip::tcp::endpoint ep);
    void at_connack(reason_code rc, bool session_present, const connack_props& ca_props);
    void at_disconnect(reason_code rc, const disconnect_props& dc_props);
};

class detail::log_invoke<LoggerType> { LoggerType _logger;
public: explicit ctor(LoggerType logger={});
    void at_XXX(...) { if constexpr (has_at_XXX) _logger.at_XXX(...); }
};
```

------
### Concepts

```c++
concept StreamType<S> = beast::AsyncStream<S> && requires (S s) {
    typename S::lowest_layer_type;
    requires(std::derived_from<S::lowest_layer_type, boost::asio::ip::tcp::socket>);
    {s.lowest_layer()}->std::convertible_to<S::lowest_layer_type&>;
    async_shutdown(s);
};

concept TlsContext<Ctx>; // opaque to MQTT5

concept Authenticator<Auth> = requires(Auth a, auth_step_e step, std::string data,
  asio::any_completion_handler<void(boost::mqtt5::error_code ec, std::string client_data)> h) {
    a.async_auth(step, data, h);
    {a.method()} -> std::same_as<std::string_view>;
};

concept LoggerType<Logger> = requires(Logger l, error_code ec, std::string_view sv,
  const asio::ip::tcp::resolver::results_type& eps, asio::ip::tcp::endpoint ep,
  reason_code rc, bool b, const connack_props& ca_props, const disconnect_props& dc_props) {
    // all members are optional
    l.at_resolve(ec, sv, sv, eps);
    l.at_tcp_connect(ec, ep);
    l.at_tls_handshake(ec, ep);
    l.at_ws_handshake(ec, ep);
    l.at_connack(rc, b, ca_props);
    l.at_disconnect(rc, dc_props);
};
```

------
### MQTT5 Client

```c++
class mqtt_client<StreamType, TlsContext=std::monostate, LoggerType=noop_logger> {
    using stream_type = StreamType; using tls_context_type = TlsContext; using logger_type = LoggerType;
    using client_service_type = client_service<stream_type, tls_context_type, logger_type>; using impl_type = std::shared_ptr<client_service_type>;
    impl_type _impl;
public: using executor_type = stream_type::executor_type;
    struct rebind_executor<Executor> { using other = self<detail::rebind_executor<stream_type, Executor>::other, TlsContext, LoggerType>; };
    explicit ctor(const executor_type& ex, tls_context_type tls_context={}, logger_type logger={})
        : _impl{std::make_shared<client_service_type>(ex, std::move(tls_context), std::move(logger))}{}
    explicit ctor<ExecCtx>(ExecCtx& context, tls_context_type tls_context={}, logger_type logger={}) requires is_convertible_v<ExecCtx&, asio::execution_context&>
        : self{context.get_executor(), std::move(tls_context), std::move(logger)}{}
    ctor(self&&) noexcept = default; ~dtor() { if (_impl) _impl->cancel(); }
    self& operator=(self&& other) noecxept { _impl->cancel(); _impl=std::move(other._impl); return *this; }

    executor_type get_executor() const noexcept { return _impl->get_executor(); }
    decltype(auto) tls_context<Ctx>() requires !std::is_same_v<Ctx,std::monostate> { return _impl->tls_context(); }
    decltype(auto) async_run<Token=asio::default_completion_token<executor_type>::type>(Token&& token={})
    { return asio::async_initiate<Token,void(error_code)>(initiate_async_run(_impl), token); }
    void cancel() { auto impl=_impl; _impl = impl->dup(); impl->cancel(); }

    self& will(will will) { _impl->will(std::move(will)); return *this; }
    self& credentials(std::string client_id, std::string username="", std::string password="")
    { _impl->credentials(std::move(client_id), std::move(username), std::move(password)); return *this; }
    self& brokers(std::string hosts, uint16_t defualt_port=1883) { _impl->brokers(std::move(hosts), default_port); return *this; }
    self& authenticator<Auth>(Auth&& authenticator) { _impl->authenticator(std::forward<Auth>(authenticator)); return *this; }
    self& keep_alive(uint16_t seconds) { _impl->keep_alive(seconds); return *this; }
    self& connect_properties(connect_props pros) { _impl->connect_properties(std::move(props)); return *this; }
    self& connect_property<p>(std::integral_constant<prop::property_type, p> prop, value_type_t<p> value)
    { _impl->connect_property(prop, std::move(value)); return *this; }

    void re_authenticate() { re_auth_op{_impl}.perform(); }
    const auto& connack_property<p>(std::integral_constant<prop::property_type, p> prop) const { return _impl->connack_property(prop); }
    const connack_props connack_properties() const { return _impl->connack_properties(); }
    decltype(auto) async_publish<qos_type,Token=asio::default_completion_token<executor_type>::type>
        (std::string topic, std::string payload, retain_e retain, const publish_props& props, Token&& token={})
    { return asio::async_initiate<Token,on_publish_signature<qos_type>>(
        initiate_async_publish<qos_type>{_impl}, token, std::move(topic), std::move(payload), retain, props); }
    decltype(auto) async_subscribe<Token=asio::default_completion_token<executor_type>::type>
        (const std::vector<subscribe_topic>& topics, const subscribe_props& props, Token&& token={})
    { return asio::async_initiate<Token,void(error_code,std::vector<reason_code>,suback_props)>
        (initiate_async_subscribe{_impl}, token, topics, props); }
    decltype(auto) async_subscribe<Token=asio::default_completion_token<executor_type>::type>
        (const subscribe_topic& topic, const subscribe_props& props, Token&& token={})
    { return async_subscribe(std::vector{topic}, props, std::forward<Token>(token)); }
    decltype(auto) async_unsubscribe<Token=asio::default_completion_token<executor_type>::type>
        (const std::vector<std::string>& topics, const unsubscribe_props& props, Token&& token={})
    { return asio::async_initiate<Token,void(error_code,std::vector<reason_code>,unsuback_props)>
        (initiate_async_unsubscribe{_impl}, token, topics, props); }
    decltype(auto) async_unsubscribe<Token=asio::default_completion_token<executor_type>::type>
        (const std::string& topic, const unsubscribe_props& props, Token&& token={})
    { return async_unsubscribe(std::vector{topic}, props, std::forward<Token>(token)); }
    decltype(auto) async_receive<Token=asio::default_completion_token<executor_type>::type>(Token&& token={})
    { return _impl->async_channel_receive(std::forward<Token>(token)); }
    decltype(auto) async_disconnect<Token=asio::default_completion_token<executor_type>::type>
        (disconnect_rc_e reason_code, const disconnect_props& props, Token&& token={})
    { auto impl=_impl; _impl=impl.dup(); return _impl->async_terminal_disconnect({reason_code}, props, impl, std::forward<Token>(token)); }
    decltype(auto) async_disconnect<Token=asio::default_completion_token<executor_type>::type>(Token&& token={})
    { return async_disconnect(normal_disconnection, {}, std::forward<Token>(token)); }
};
```

------
### Implementation Details

##### Operations

```c++
struct detail::data_span :  std::pair<byte_citer, byte_citer> {
    auto first() const; auto last() const;
    void expand_suffix(size_t); void remove_prefix(size_t); size_t size() const;
};
class detail::assemble_op<ClientService,Handler> {
    struct on_read{}; using client_service = ClientService; using handler_type = Handler;
    client_service& _svc; handler_type _handler; std::string& _read_buff; data_span& _data_span;

    void prepare_buffer(ptrdiff_t extra_len);
    uint32_t max_recv_size() const;
    duration compute_read_timeout() const;
    static bool valid_header(uint8_t control_byte);
    void dispatch(uint8_t control_byte, byte_citer first, byte_citer last);
    void complete(error_code ec, uint8_t control_code, byte_citer first, byte_citer last);)
public: ctor(client_service& svc, handler_type&& handler, std::string& read_buff, data_span& active_span); // allow move, disable copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using executor_type = asio::associated_executor_t<handler_type>;
    executor_type get_executor() const noexcept { return asio::get_associated_executor(_handler); }
    void perform<CompletionCondition>(CompletionCondition cc);
    void operator()<CompletionCondition>(on_read, error_code ec, size_t bytes_read, CompletionCondition cc);
};

class detail::connect_op<Stream,LoggerType> {
    static constexpr size_t min_packet_sz = 5;
    struct on_connect{}; struct on_tls_handshake{}; struct on_ws_handshake{}; struct on_send_connect{}; struct on_fixed_header{};
    struct on_read_packet{}; struct on_init_auth_data{}; struct on_auth_data{}; struct on_send_auth{}; struct on_complete_auth{}; struct on_shutdown{};
    Stream& _stream; mqtt_ctx& _ctx; log_invoke<LoggerType>& _log;
    using handler_type = asio::any_completion_handler<void(error_code)>; using endpoint = asio::ip::tcp::endpoint;
    handler_type _handler; std::unique_ptr<std::string> _buffer_ptr; asio::cancellation_state _cancellation_state;
    bool is_cancelled() const;
    void complete(error_code ec);
public: ctor<Handler>(Stream& stream, mqtt_ctx& ctx, log_invoke<LoggerType>& log, Handler&& handler); // allow move, disable copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using cancellation_slot_type = asio::cancellation_slot;
    cancellation_slot_type get_cancellation_slot() const noexcept { return _cancellation_state.slot(); }
    using executor_type = asio::associated_executor_t<handler_type>;
    executor_type get_executor() const noexcept { return asio::get_associated_executor(_handler); }
    void perform(const endpoint& ep, authority_path ap);
    void operator()(on_connect, error_code ec, endpoint ep, authority_path ap);
    void do_tls_handshake(endpoint ep, authority_path ap);
    void operator()(on_tls_handshake, error_code ec, endpoint ep, authority_path ap);
    void do_ws_handshake(endpoint ep, authority_path ap);
    void operator()(on_ws_handshake, error_code ec, endpoint ap);
    void operator()(on_init_auth_data, error_code ec, std::string data);
    void send_connect();
    void operator()(on_send_connect, error_code ec, size_t);
    void operator()(on_fixed_header, error_code ec, size_t num_read);
    void operator()(on_read_packet, error_code ec, size_t, control_code_e code, byte_citer first, byte_citer last);
    void on_connack(byte_citer first, byte_citer last);
    void on_auth(byte_citer first, byte_citer last);
    void operator()(on_auth_data, error_code ec, std::string data);
    void operator()(on_send_auth, error_code ec, size_t);
    void operator()(on_complete_auth, error_code ec, std::string);
    void do_shutdown(error_code connect_ec);
    void operator()(on_shutdown, error_code connect_ec, error_code);
};

class detail::disconnect_op<ClientService,DisconnectContext> {
    using client_service = ClientService;
    struct on_disconnect{}; struct on_shutdown{};
    std::shared_ptr<client_service> _svc_ptr; DisconnectContext _context;
    using handler_type = asio::any_completion_handler<void(error_code), ClientService::executor_type>;
    handler_type _handler;
    static error_code validate_disconnect(const disconnect_props& props);
    void complete(error_code ec);
    void complete_immediate(error_code ec);
public: ctor<Handler>(std::shared_ptr<client_service> svc_ptr, DisconnectContext&& context, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using executor_type = client_service::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void perform();
    void send_disconnect(control_packet<allocator_type> disconnect);
    void operator()(on_disconnect, control_packet<allocator_type> disconnect, error_code ec);
    void operator()(on_shutdown, error_code ec);
};

class detail::terminal_disconnect_op<ClientService,Handler> {
    using client_service = ClientService; using handler_type = Handler;
    static constexpr uint8_t seconds = 5;
    std::shared_ptr<client_service> _svc_ptr; std::unique_ptr<asio::steady_timer> _timer; handler_type _handler;
public: ctor(std::shared_ptr<client_service> svc_ptr, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using cancellation_slot_type = asio::associated_cancellation_slot_t<handler_type>;
    cancellation_slot_type get_cancellation_slot() const noexcept { return asio::get_associated_cancellation_slot(_handler); }
    using executor_type = asio::associated_executor_t<handler_type>;
    executor_type get_executor() const noexcept { return asio::get_associated_executor(_handler); }
    void perform<DisconnectContext>(DisconnectContext&& context);
    void operator()(std::array<size_t, 2>, error_code disconnect_ec, error_code);
};

class detail::initiate_async_disconnect<ClientService,terminal> {
    std::shared_ptr<ClientService> _svc_ptr;
public: explicit ctor(std::shared_ptr<ClientService> svc_ptr);
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void operator()<Handler>(Handler&& handler, disconnect_rc_e rc, const disconnect_props& props);
};
decltype(auto) detail::async_disconnect<ClientService,Token>(disconnect_rc_e reason_code, const disconnect_props& props, std::shared_ptr<ClientService> svc_ptr, Token&& token);
decltype(auto) detail::async_terminal_disconnect<ClientService,Token>(disconnect_rc_e reason_code, const disconnect_props& props, std::shared_ptr<ClientService> svc_ptr, Token&& token);

class detail::ping_op<ClientService,Handler> {
    using client_service = ClientService; using handler_type = Handler;
    struct on_timer{}; struct on_pingreq{};
    std::shared_ptr<client_service> _svc_ptr; handler_type _handler;
    duration compute_wait_time() const;
    void complete();
public: ctor(std::shared_ptr<client_service> svc_ptr, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void perform();
    void operator()(on_timer, error_code ec);
    void operator()(on_pingreq, error_code ec);
};

class detail::publish_rec_op<ClientService> {
    using client_service = ClientService;
    struct on_puback{}; struct on_pubrec{}; struct on_pubrel{}; struct on_pubcomp{};
    std::shared_ptr<client_service> _svc_ptr; decoders::publish_message _message;
    void on_malformed_packet(const std::string& reason);
    void complete();
public: ctor(std::shared_ptr<client_service> svc_ptr); // allow move, no copy
    using allocator_type = asio::recycling_allocator<void>;
    allocator_type get_allocator() const noexcept { return {}; }
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void perform(publish_message message);
    void send_puback(control_packet<allocator_type> puback);
    void operator()(on_puback, error_code ec);
    void send_pubrec(control_packet<allocator_type> pubrec);
    void operator()(on_pubrec, control_packet<allocator_type> pubrec, error_code ec);
    void wait_pubrel(uint16_t packet_id);
    void operator()(on_pubrel, uint16_t packet_id, error_code ec, byte_citer first, byte_citer last);
    void send_pubcomp(control_packet<allocator_type> pubcomp);
    void operator()(on_pubcomp, control_packet<allocator_type> pubcomp, error_code ec);
};

using detail::on_publish_signature<qos_e qos_type> =
    qos_type==at_most_once ? void(error_code) : qos_type==at_least_once ? void(error_code, reason_code, puback_props) : void(error_code, reason_code, pubcomp_props);
using detail::on_publish_props_type<qos_e qos_type> = qos_type==at_most_once ? void : qos_type==at_least_once ? puback_props : pubcomp_props;
class detail::publish_send_op<ClientService,Handler,qos_type> {
    using client_service = ClientService; using handler_type = cancellable_handler<Handler,client_service::executor_type>;
    struct on_publish{}; struct on_puback{}; struct on_pubrec{}; struct on_pubrel{}; struct on_pubcomp{};
    std::shared_ptr<client_service> _svc_ptr; handler_type _handler; serial_num_t _serial_num;
    error_code validate_publish(const std::string& topic, const std::string& payload, retain_e retain, const publish_props& props) const;
    error_code validate_props(const publish_props& props) const;
    void on_malformed_packet(const std::string& reason);
    void complete<q=qos_type>(error_code ec, uint16_t=0) requires q==at_most_once;
    void complete_immediate<q=qos_type>(error_code ec, uint16_t) requires q==at_most_once;
    void complete<Props=on_publish_props_type<qos_type>>(error_code ec, uint16_t packet_id, reason_code rc=empty, Props&& props={}) requires is_same_v<Props,puback_props>||is_same_v<Props,pubcomp_props>;
    void complete_immediate<Props=on_publish_props_type<qos_type>>(error_code ec, uint16_t packet_id) requires is_same_v<Props,puback_props>||is_same_v<Props,pubcomp_props>;
public: ctor(std::shared_ptr<client_service> svc_ptr, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void perform(std::string topic, std::string payload, retain_e retain, const publish_props& props);
    void send_publish(control_packet<allocator_type> publish);
    void resend_publish(control_packet<allocator_type> publish);
    void operator()(on_publish, control_packet<allocator_type> publish, error_code ec);
    void operator()<q=qos_type>(on_puback, control_packet<allocator_type> publish, error_code ec, byte_citer first, byte_citer last) requires q==at_least_once;
    void operator()<q=qos_type>(on_pubrec, control_packet<allocator_type> publish, error_code ec, byte_citer first, byte_citer last) requires q==exactly_once;
    void send_pubrel(control_packet<allocator_type> pubrel, bool throttled);
    void operator()<q=qos_type>(on_pubrel, control_packet<allocator_type> pubrel, error_code ec) requires q==exactly_once;
    void operator()<q=qos_type>(on_pubcomp, control_packet<allocator_type> pubrel, error_code ec, byte_citer first, byte_citer last) requires q==exactly_once;
};
class detail::initiate_async_publish<ClientService,qos_type> {
    std::shared_ptr<ClientService> _svc_ptr;
public: explicit ctor(std::shared_ptr<ClientService> svc_ptr);
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void operator()<Handler>(Handler&& handler, std::string topic, std::string payload, retain_e retain, const publish_props& props);
};

class detail::re_auth_op<ClientService> {
    using client_service = ClientService;
    struct on_auth_data{};
    std::shared_ptr<client_service> _svc_ptr; any_authenticator _auth;
    void on_auth_fail(std::string message, disconnect_rc_e reason)
public: ctor<Handler>(std::shared_ptr<client_service> svc_ptr); // allow move, no copy
    using allocator_type = asio::recycling_allocator<void>;
    allocator_type get_allocator() const noexcept { return {}; }
    using executor_type = client_service::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void perform();
    void perform(decoders::auth_message auth_message);
    void operator()(on_auth_data, auth_step_e auth_step,, error_code ec, std::string data);
};

class detail::read_message_op<ClientService,Handler> {
    using client_service = ClientService; using handler_type = Handler;
    struct on_message{}; struct on_disconnect{};
    std::shared_ptr<client_service> _svc_ptr; handler_type _handler;
    void dispatch(uint8_t control_byte, byte_citer first, byte_citer last);
    void on_malformed_packet(disconnect_rc_e rc, const std::string& reason);
    void complete();
public: ctor(std::shared_ptr<client_service> svc_ptr, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void perform();
    void operator()(on_message, error_code ec, uint8_t control_code, byte_citer first, byte_citer last);
    void operator()(on_disconnect, error_code ec);
};

class detail::read_op<Owner,Handler> {
    using handler_type = Handler;
    struct on_read{}; struct on_reconnect{};
    Owner& _owner; handler_type _handler;
    void complete(error_code ec, size_t bytes_read);
    static bool should_reconnect(error_code ec);
public: ctor(Owner& owner, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using executor_type = asio::associated_executor_t<handler_type>;
    executor_type get_executor() const noexcept { return asio::get_associated_executor(_handler); }
    void perform<BufferType>(const BufferType& buffer, duration wait_for);
    void operator()(on_read, Owner::stream_ptr stream_ptr, std::array<size_t,2> ord, error_code read_ec, size_t bytes_read, error_code);
    void operator()(on_reconnect, error_code ec);
};

class detail::exponential_backoff {
    int _curr_exp{0};
    static constexpr int _base_multiplier=1000, _max_exp=4;
    boost::random::rand48 _generator{std::time(0)}; boost::random::uniform_smallint<> _distribution{-500,500};
public: ctor();
    duration generate();
};
class detail::reconnect_op<Owner> {
    using handler_type = asio::any_completion_handler<void(error_code)>;
    struct on_locked{}; struct on_next_endpoint{}; struct on_connect{}; struct on_backoff{};
    Owner& _owner; handler_type _handler; std::unique_ptr<std::string> _buffer_ptr; exponential_backoff _generator;
    using endpoint = asio::ip::tcp::endpoint; using epoints = asio::ip::tjcp::resolver::result_type;
    void complete(error_code ec);
public: ctor<Handler>(Owner& owner, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using cancellation_slot_type = asio::associated_cancellation_slot_t<Handler>;
    cancellation_slot_type get_cancellation_slot() const noexcept { return asio::get_associated_cancellation_slot(_handler); }
    using executor_type = asio::associated_executor_t<handler_type>;
    executor_type get_executor() const noexcept { return asio::get_associated_executor(_handler); }
    void perform(Owner::stream_ptr s);
    void operator()(on_locked, Owner::stream_ptr stream_ptr, error_code ec);
    void do_reconnect();
    void backoff_and_reconnect();
    void operator()(on_backoff, error_code ec);
    void operator()(on_next_endpoint, error_code ec, epoints eps, authority_path ap);
    void connect(epoints::const_iterator eps, authority_path ap);
    void operator()(on_connect, Owner::stream_ptr sptr, epoints::const_iterator eps, authority_path ap, std::array<size_t,2> ord, error_code connect_ec, error_code timer_ec);
};

class detail::run_op<ClientService,Handler> {
    using client_service = ClientService; using handler_type = Handler;
    std::shared_ptr<client_service> _svc_ptr; handler_type _handler;
public: ctor(std::shared_ptr<client_service> svc_ptr, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void perform();
    void operator()(std::array<size_t,3>);
};
class detail::initiate_async_run<ClientService> {
    std::shared_ptr<ClientService> _svc_ptr;
public: explicit ctor(std::shared_ptr<ClientService> svc_ptr);
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void operator()<Handler>(Handler&& handler);
};

class detail::sentry_op<ClientService,Handler> {
    using client_service = ClientService; using handler_type = Handler;
    struct on_timer{}; struct on_disconnect{};
    static constexpr auto check_interval = std::chrono::seconds{3};
    std::shared_ptr<client_service> _svc_ptr; handler_type _handler;
    void complete();
public: ctor(std::shared_ptr<client_service> svc_ptr, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void perform();
    void operator()(on_timer, error_code ec);
    void operator()(on_disconnect, error_code ec);
};

constexpr bool detail::is_basic_socket<T> = ...; // asio::basic_stream_socket<>;
class detail::shutdown_op<Owner> {
    using handler_type = asio::any_completion_handler<void(error_code)>;
    struct on_locked{}; struct on_shutdown{};
    Owner& _owner; handler_type _handler;
    void complete(Owner::stream_ptr sptr, error_code ec);
public: ctor<Handler>(Owner& owner, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using cancellation_slot_type = asio::associated_cancellation_slot_t<Handler>;
    cancellation_slot_type get_cancellation_slot() const noexcept { return asio::get_associated_cancellation_slot(_handler); }
    using executor_type = Owner::executor_type;
    executor_type get_executor() const noexcept { return _owner.get_executor()_; }
    void perform();
    void operator()(on_locked, Owner::stream_ptr stream_ptr, error_code ec);
    void operator()(on_shutdown, Owner::stream_ptr sptr, std::array<size_t,2>, error_code, error_code);
};

class detail::subscribe_op<ClientService,Handler> {
    using client_service = ClientService; using handler_type = cancellable_handler<Handler, ClientService::executor_type>;
    struct on_subscribe{}; struct on_suback{};
    std::shared_ptr<client_service> _svc_ptr; handler_type _handler; size_t _num_topics{0};
    error_code validate_subscribe(const std::vector<subscribe_topic>& topics, const subscribe_props& props) const;
    error_code validate_topic(const subscribe_topic& topic) const;
    error_code validate_props(const subscribe_props& props) const;
    static std::vector<reason_code> to_reason_codes(std::vector<uint8_t> codes);
    void on_malformed_packet(const std::string& reason);
    void complete_immediate(error_code ec, uint16_t packet_id);
    void complete(error_code ec, uint16_t packet_id, std::vector<reason_code> reason_codes={}, suback_props props={});
public: ctor(std::shared_ptr<client_service> svc_ptr, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void perform(const std::vector<subscribe_topic>& topics, const subscribe_props& props);
    void send_subscribe(control_packet<allocator_type> subscribe);
    void resend_subscribe(control_packet<allocator_type> subscribe);
    void operator()(on_subscribe, control_packet<allocator_type> packet, error_code ec);
    void operator()(on_suback, control_packet<allocator_type> packet, error_code ec, byte_citer first, byte_citer last);
};
class detail::initiate_async_subscribe<ClientService> {
    std::shared_ptr<ClientService> _svc_ptr;
public: explicit ctor(std::shared_ptr<ClientService> svc_ptr);
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void operator()<Handler>(Handler&& handler, const std::vector<subscribe_topic>& topics, const subscribe_props& props);
};

class detail::unsubscribe_op<ClientService,Handler> {
    using client_service = ClientService; using handler_type = cancellable_handler<Handler, ClientService::executor_type>;
    struct on_unsubscribe{}; struct on_unsuback{};
    std::shared_ptr<client_service> _svc_ptr; handler_type _handler; size_t _num_topics{0};
    error_code validate_unsubscribe(const std::vector<std::string>& topics, const unsubscribe_props& props) const;
    static std::vector<reason_code> to_reason_codes(std::vector<uint8_t> codes);
    void on_malformed_packet(const std::string& reason);
    void complete_immediate(error_code ec, uint16_t packet_id);
    void complete(error_code ec, uint16_t packet_id, std::vector<reason_code> reason_codes={}, unsuback_props props={});
public: ctor(std::shared_ptr<client_service> svc_ptr, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void perform(const std::vector<std::string>& topics, const unsubscribe_props& props);
    void send_unsubscribe(control_packet<allocator_type> unsubscribe);
    void resend_unsubscribe(control_packet<allocator_type> unsubscribe);
    void operator()(on_unsubscribe, control_packet<allocator_type> packet, error_code ec);
    void operator()(on_unsuback, control_packet<allocator_type> packet, error_code ec, byte_citer first, byte_citer last);
};
class detail::initiate_async_unsubscribe<ClientService> {
    std::shared_ptr<ClientService> _svc_ptr;
public: explicit ctor(std::shared_ptr<ClientService> svc_ptr);
    using executor_type = ClientService::executor_type;
    executor_type get_executor() const noexcept { return _svc_ptr->get_executor(); }
    void operator()<Handler>(Handler&& handler, const std::vector<std::string>& topics, const unsubscribe_props& props);
};

class detail::write_op<Owner,Handler> {
    using handler_type = Handler;
    struct on_write{}; struct on_reconnect{};
    Owner& _owner; handler_type _handler;
    void complete(error_code ec, size_t bytes_written);
    static bool should_reconnect(error_code ec);
public: ctor(Owner& owner, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using executor_type = asio::associated_executor_t<handler_type>;
    executor_type get_executor() const noexcept { return asio::get_associated_executor(_handler); }
    void perform<BufferType>(BufferType& buffer);
    void operator()(on_write, Owner::stream_ptr stream_ptr, error_code ec, size_t bytes_written);
    void operator()(on_reconnect, error_code ec);
};

```

##### Messages

```c++
auto decoders::type_parse<It,Parser>(It& first, const It last, const Parser& p);
struct decoders::basic::int_parser<Attr> { using attribute_type = Attr;
    bool parse<It>(It& first, const It last, Attr& attr) const; };
constexpr int_parser<uint8_t> decoders::basic::byte_;
constexpr int_parser<uint16_t> decoders::basic::word_;
constexpr int_parser<uint32_t> decoders::basic::dword_;

namespace boost::mqtt5::decoders::basic {
struct varint_parser { using attribute_type = int32_t; bool parse<It>(It& first, const It last, int32_t& attr) const; };
constexpr varint_parser varint_;

struct len_prefix_parser { using attribute_type = std::string; bool parse<It>(It& first, const It last, std::string& attr) const; };
constexpr len_prefix_parser utf8_;
constexpr len_prefix_parser binary_;

struct conditional_parser<Subject> {
    using subject_attr_type = Subject::attribute_type;
    using attribute_type = std::optional<subject_attr_type>;
    Subject p; bool condition;
    bool parse<It>(It& first, const It last, attribute_type& attr) const;
};
struct conditional_gen { bool _condition; conditional_parser<Subject> operator[]<Subject>(const Subject& p) const { return {p,_condition}; } };
constexpr conditional_gen if_(bool condition) { return {condition}; }

struct verbatim_parser { using attribute_type = std::string; bool parse<It>(It& first, const It last, std::string& attr) const; };
constexpr verbatim_parser verbatim_;

struct seq_parser<...Ps> {
    using attribute_type = std::tuple<Ps::attribute_type...>;
    std::tuple<Ps...> parsers;
    bool parse<It>(It& first, const It last, attribute_type& attr) const;
};
constexpr seq_parser<Ps...,P2> operator>> <...Ps,P2> (const seq_parser<Ps...>& p1, const P2& p2);
constexpr seq_parser<P1,P2> operator>> <P1,P2> (const P1& p1, const P2& p2);

struct attr_parser<Attr> { using attribute_type = Attr; Attr attr; bool parse<It>(It& first, const It last, Attr& attr) const; };
constexpr attr_parser<Attr> attr(const Attr& val);

struct plus_parser<Subject> {
    using subject_attr_type = Subject::attribute_type;
    using attribute_type = std::optional<subject_attr_type>;
    Subject p;
    bool parse<It,Attr>(It& first, const It last, Attr& attr) const;
};
constexpr plus_parser<P> operator+<P>(const P& p);
}

namespace boost::mqtt5::decoders::prop {
bool detail::parse_to_prop<It,Prop>(It& iter, const It last, Prop& prop);

struct prop_parser<Props> { using attribute_type = Props; bool parse<It>(It& first, const It last, Props& attr) const; };
constexpr auto props_<Props> = prop_parser<Props>{};
}

namespace boost::mqtt5::decoders {
using fixed_header = std::tuple<uint8_t, uint32_t>;
std::optional<fixed_header> decode_fixed_header(byte_citer& it, const byte_citer last);
using packet_id = uint16_t;
std::optional<packet_id> decode_packet_id(byte_citer& it);
using connect_message = std::tuple<std::string, std::optional<std::string>, std::optional<std::string>, uint16_t, bool, connect_props, std::optional<will>>;
std::optional<connect_message> decode_connect(uint32_t remain_length, byte_citer& it);
using connack_message = std::tuple<uint8_t, uint8_t, connack_props>;
std::optional<connack_message> decode_connack(uint32_t remain_length, byte_citer& it);
using publish_message = std::tuple<std::string, std::optional<uint16_t>, uint8_t, publish_props, std::string>;
std::optional<publish_message> decode_publish(uint8_t control_byte, uint32_t remain_length, byte_citer& it);
using puback_message = std::tuple<uint8_t, puback_props>;
std::optional<puback_message> decode_puback(uint32_t remain_length, byte_citer& it);
using pubrec_message = std::tuple<uint8_t, pubrec_props>;
std::optional<pubrec_message> decode_pubrec(uint32_t remain_length, byte_citer& it);
using pubrel_message = std::tuple<uint8_t, pubrel_props>;
std::optional<pubrel_message> decode_pubrel(uint32_t remain_length, byte_citer& it);
using pubcomp_message = std::tuple<uint8_t, pubcomp_props>;
std::optional<pubcomp_message> decode_pubcomp(uint32_t remain_length, byte_citer& it);
using subscribe_message = std::tuple<subscribe_props, std::vector<std::tuple<std::string, uint8_t>>>;
std::optional<subscribe_message> decode_subscribe(uint32_t remain_length, byte_citer& it);
using suback_message = std::tuple<suback_props, std::vector<uint8_t>>;
std::optional<suback_message> decode_suback(uint32_t remain_length, byte_citer& it);
using unsubscribe_message = std::tuple<unsubscribe_props, std::vector<std::string>>;
std::optional<unsubscribe_message> decode_unsubscribe(uint32_t remain_length, byte_citer& it);
using unsuback_message = std::tuple<unsuback_props, std::vector<uint8_t>>;
std::optional<unsuback_message> decode_unsuback(uint32_t remain_length, byte_citer& it);
using disconnect_message = std::tuple<uint8_t, disconnect_props>;
std::optional<disconnect_message> decode_disconnect(uint32_t remain_length, byte_citer& it);
using auth_message = std::tuple<uint8_t, auth_props>;
std::optional<auth_message> decode_auth(uint32_t remain_length, byte_citer& it);
}

namespace boost::mqtt5::encoders::basic {
using varint_t = int*;
void to_variable_bytes(std::string& s, int32_t val);
size_t variable_length(int32_t val);

struct encoder{};

class flag_def<bits,Repr=uint8_t> : public encoder {
    using least_type<num_bits> = ...; // choose by bit-width: uint8_t/uint16_t/uint32_t/uint64_t/void
    Repr _val{0};
public: ctor(Repr val); ctor()=default;
    auto operator()<T,projection=identity> (T&& value, projection proj={}) const; // T is or not optional<>
    size_t byte_size() const { return sizeof(Repr); }
    auto operator|<rhs_bits,rhs_repr>(const self<rhs_bits,rhs_repr>& rhs) const;
    std::string& encode(std::string& s) const;
};
constexpr auto flag<bits,Repr=uint8_t> = flag_def<bits,Repr>{};

class int_val<T,Repr> : public encoder { T _val; // T maybe optional
public: ctor(T val);
    size_t byte_size() const;
    std::string& encode(std::string& s) const;
};
struct int_def<Repr=uint8_t> {
    auto operator()<T> (T&& value) const { return int_val<T,Repr>{std::forward<T>(val)}; }
    auto operator()<T,projection> (T&& value, projection proj) const; // T is or not optional<>
};
constexpr auto byte_ = int_def<uint8_t>{};
constexpr auto int16_ = int_def<uint16_t>{};
constexpr auto int32_ = int_def<uint32_t>{};
constexpr auto varlen_ = int_def<varint_t>{};

class array_val<T> : public encoder { T _val; bool _with_length; // T maybe optional
public: ctor(T val, bool with_length);
    size_t byte_size() const;
    std::string& encode(std::string& s) const;
};
struct array_def<with_length=true> {
    auto operator()<T> (T&& value) const { return array_val<T,Repr>{std::forward<T>(val), with_length}; }
    auto operator()<T,projection> (T&& value, projection proj) const; // T is or not optional<>
};
using utf8_def = array_def<true>;
constexpr auto utf8_ = utf8_def{};
constexpr auto binary_ = array_def<true>{};
constexpr auto verbatim_ = array_def<false>{};

class composed_val<T,U> : public encoder { T _lhs; U _rhs;
public: ctor(T lhs, U rhs);
    size_t byte_size() const;
    std::string& encode(std::string& s) const;
};
auto operator&<T,U>(T&& t, U&& u) requires (...); // T and U are both encoder
std::string& operator<< <T> (std::string& s, T&& t) requires(...); // T is encoder
}

namespace boost::mqtt5::encoders::prop {
auto encoder_for_prop_value<T>(const T& val);
class prop_val<T,property_type p> : public encoder { // T is std::optional<>, std::vector<> or small_vector<>
    T _val; static inline std::remove_cv_rev_t<T> nulltype;
public: ctor(T val):_val{val}{} ctor():_val{nulltype}{}
    size_t byte_size() const;
    std::string& encode(std::string& s) const;
};
class props_val<Props> : public encoder {
    static inline std::decay_t<Props> nulltype;
    static auto to_prop_val<p,T>(const T& val) { return prop_val<const T&,p>{val}; }
    static auto to_prop_vals<...ps,T>(const properties<ps...>& props) { return make_tuple(to_prop_val<ps>(props[ps{}])...); }
    decltype(to_prop_vals(std::declval<Props>())) _prop_vals;
    bool _may_omit;
public: ctor(Props val, bool may_omit); ctor(bool may_omit);
    size_t byte_size() const;
    std::string& encode(std::string& s) const;
};
struct props_def<may_omit> { auto operator()<T>(T&& prop_container) const; };
constexpr auto props_ = props_def<false>{};
constexpr auto props_may_omit_ = props_def<true>{};
}

namespace boost::mqtt5::encoders {
std::string encode<Encoder>(const Encoder& e);
std::string encode_connect(std::string_view client_id, std::optional<std::string_view> user_name, std::optional<std::string_view> password,
    uint16_t keep_alive, bool clean_start, const connect_props& props, const std::optional<will>& w);
std::string encode_connack(bool session_present, uint8_t reason_code, const connack_props props);
std::string encode_publish(uint16_t packet_id, std::string_view topic_name, std::string_view payload,
    qos_e qos, retain_e retain, dup_e dup, const publish_props& props);
std::string encode_puback(uint16_t packet_id, uint8_t reason_code, const puback_props& props);
std::string encode_pubrec(uint16_t packet_id, uint8_t reason_code, const pubrec_props& props);
std::string encode_pubrel(uint16_t packet_id, uint8_t reason_code, const pubrel_props& props);
std::string encode_pubcomp(uint16_t packet_id, uint8_t reason_code, const pubcomp_props& props);
std::string encode_subscribe(uint16_t packet_id, const std::vector<subscribe_topic>& topics, const subscribe_props& props);
std::string encode_suback(uint16_t packet_id, const std::vector<uint8_t>& reason_codes, const suback_props& props);
std::string encode_unsubscribe(uint16_t packet_id, const std::vector<std::string>& topics, const unsubscribe_props& props);
std::string encode_unsuback(uint16_t packet_id, const std::vector<uint8_t>& reason_codes, const unsuback_props& props);
std::string encode_pingreq();
std::string encode_pingresp();
std::string encode_disconnect(uint8_t reason_code, const disconnect_props& props);
std::string encode_auth(uint8_t reason_code, const auth_props& props);
}
```

##### Client Service Implementation

```c++
class detail::stream_context<StreamType,TlsContext,std::enable_if_t<has_tls_layer<StreamType>>> {
    using tls_context_type = TlsContext;
    mqtt_ctx _mqtt_context; std::shared_ptr<tls_context_type> _tls_context_ptr;
public: explicit ctor(TlsContext tls_context); ctor(cosnt self& other);
    <const> auto& mqtt_context() <const> { return _mqtt_context; }
    auto& tls_context() { return *_tls_context_ptr; }
    <const> auto& session_state() <const> { return _mqtt_context.state; }
    void will(will will) { _mqtt_context.will_msg = std::move(will); }
    const auto& connack_property<property_type p>(<p> prop) const { return _mqtt_context.ca_props[prop]; }
    const auto& connack_properties() const { return _mqtt_context.ca_props; }
    <const> auto& connect_property<property_type p>(<p> prop) <const> { return _mqtt_context.co_props[prop]; }
    void connect_properties(connect_props props) { _mqtt_context.co_props = std::move(props); }
    void credentials(std::string client_id, std::string username="", std::string password="")
    { _mqtt_context.creds = {std::move(client_id), std::move(username), std::move(password)}; }
    void authenticator<Auth>(Auth&& authenticator) { _mqtt_context.authenticator = any_authenticator{std::forward<Auth>(authenticator)}; }
};
class detail::stream_context<StreamType,TlsContext,std::enable_if_t<!has_tls_layer<StreamType>>> {
    mqtt_ctx _mqtt_context;
public: // same as above: ctor, mqtt_context, session_state, will, connack_property, connack_properties, connect_property, connect_properties, credentials, authenticator
};

class detail::client_service<StreamType,TlsContext=std::monostate,LoggerType=noop_logger> {
    using stream_context_type = stream_context<StreamType,TlsContext>;
    using stream_type = autoconnect_stream<StreamType, stream_context_type, LoggerType>;
    using tls_context_type = TlsContext; using logger_type = LoggerType;
    using receive_channel = asio::experimental::basic_channel<executor_type, channel_traits<>, void(error_code, std::string, std::string, publish_props)>;
    executor_type _executor; log_invoke<logger_type> _log; stream_context_type _stream_context; stream_type _stream;
    packet_id_allocator _pid_allocator; replies _replies; async_sender<self> _async_sender;
    std::string _read_buff; data_span _active_span; receive_channel _rec_channel; asio::steady_timer _ping_timer, _sentry_timer;
    ctor(const self& other);
public: using executor_type = stream_type::executor_type;
    explicit ctor(const executor_type& ex, tls_context_type tls_context={}, logger_type logger={});
    executor_type get_executor() const noexcept { return _executor; }
    auto dup() const { return std::shared_ptr<self>(new self{*this}); }
    decltype(auto) tls_context<Ctx=TlsContext>() requires !std::is_same_v<Ctx,std::monostate> { return _stream_context.tls_context(); }
    void will(will will) { if (!is_open()) _stream_context.will(std::move(will)); }
    void credential(std::string client_id, std::string username="", std::string password=""); // forward to _stream_context
    void brokers(std::string hosts, uint16_t default_port); // forward to _stream
    void authenticator<Auth>(Auth&& authenticator) requires is_authenticator<Auth>::value; // forward to _stream_context
    uint16_t negotiated_keep_alive() const { return connack_property(server_keep_alive).value_or(_stream_context.mqtt_context().keep_alive); }
    void keep_alive(uint16_t seconds) { if (!is_open()) _stream_context.mqtt_context().keep_alive = seconds; }
    const auto& connect_property<p>(<p> prop) const; // forward to _stream_context
    void connect_property<p>(<p> prop, value_type_t<p> value) const; // forward to _stream_context
    void connect_properties(connect_props props); // forward to _stream_context
    const auto& connack_property<p>(<p> prop) const; // forward to _stream_context
    const auto& connack_properties(); // forward to _stream_context
    void open_stream() { _stream.open(); }
    bool is_open() const { return _stream.is_open(); }
    decltype(auto) async_shutdown<Token>(Token&& token) { return _stream.async_shutdown(std::forward<Token>(token)); }
    void cancel() { if (!_stream.is_open()) return;
        _ping_timer.cancel(); _sentry_timer.cancel();
        _rec_channel.close(); _replies.cancel_unanswered(); _async_sender.cancel(); _stream.cancel(); stream.close(); }
    log_invoke<LoggerType>& log() { return _log; }
    uint16_t allocate_pid() { return _pid_allocator.allocate(); }
    void free_pid(uint16_t pid, bool was_throttled=false) { _pid_allocator.free(pid); if (was_throttled) _async_sender.throttled_op_done(); }
    serial_num_t next_serial_num() { return _async_sender.next_serial_num(); }
    bool subscriptions_present() const; // forward to _stream_context.session_state()
    void subscriptions_present(bool present); // forward to _stream_context.session_state()
    void update_session_state() { auto& st = _stream_context.session_state();
        if (!st.session_present()) { _replies.clear_pending_pubrels(); st.session_present(true);
            if (st.subscriptions_present()) { channel_store_error(session_expired); st.subscriptions_present(false); } }
        _ping_timer.cancel();
    }
    bool channel_store(decoders::publish_message message) { auto& [topic, packet_id, flags, props, payload] = message;
        return _rec_channel.try_send(error_code{}, std::move(topic), std::move(payload), std::move(props)); }
    bool channel_store_error(error_code ec) { return _rec_channel.try_send(ec, {}, {}, {}); }
    decltype(auto) async_send<BufferType,Token>(const BufferType& buffer, serial_num_t serial_num, unsigned flags, Token&& token)
    { return _async_sender.async_send(buffer, serial_num, flags, std::forward<Token>(token)); }
    decltype(auto) async_assemble<Token>(Token&& token)
    { return asio::async_initiate<Token,void(error_code, uint8_t, byte_citer, byte_citer)>(
        [](auto h, self& self, std::string& read_buff, data_span& active_span)
        { assemble_op{self, std::move(h), read_buff, active_span}.perform(asio::transfer_at_least(0)); },
        token, std::ref(*this), std::ref(_read_buff), std::ref(_active_span)); }
    decltype(auto) async_wait_reply<Token>(control_code_e code, uint16_t packet_id, Token&& token)
    { return _replies.async_wait_reply(code, packet_id, std::forward<Token>(token)); }
    decltype(auto) async_channel_receive<Token>(Token&& token) { return _rec_channel.async_receive(std::forward<Token>(token)); }
};
```

##### Async Sender & Replies

```c++
class detail::write_req {
    using handler_type = asio::any_completion_handler<void(error_code)>;
    static constexpr unsigned SERIAL_BITS = sizeof(serial_num_t)*8;
    asio::const_buffer _buffer; serial_num_t _serial_num; unsigned _flags; handler_type _handler;
    bool prioritized() const { return _falgs & prioritized; }
public: ctor(asio::const_buffer buffer, serial_num_t serial_num, unsigned flags, handler_type handler); // allow move, no copy
    static serial_num_t next_serial_num(serial_num_t last) { return last+1; }
    asio::const_buffer buffer() const { return _buffer; }
    void complete(error_code ec) { std::move(_handler)(ec); }
    void complete_post(const asio::any_io_executor& ex, error_code ec) { asio::post(ex, asio::prepend(std::move(_handler), ec)); }
    bool empty() const { return !_handler; }
    bool throttled() const { return _falgs & throttled; }
    bool terminal() const { return _falgs & terminal; }
    bool operator< (const self& other) const;
};

class detail::async_sender<ClientService> {
    using client_service = ClientService;
    using queue_allocator_type = asio::recycling_allocator<write_req>;
    using write_queue_t = std::vector<write_req, queue_allocator_type>;
    ClientService& _svc; write_queue_t _write_queue; bool _write_in_progress {false};
    static constexpr uint16_t MAX_LIMIT = 65535;
    uint16_t _limit{MAX_LIMIT}, _quota{MAX_LIMIT}; serial_num_t _last_serial_num{0};
    void do_write();
public: explicit ctor(ClientService& svc) : _svc{svc}{} // allow move, no copy
    using allocator_type = queue_allocator_type;
    allocator_type get_allocator() const noexcept { return {}; }
    using executor_type = client_service::executor_type;
    executor_type get_executor() const noexcept { return _svc.get_executor(); }
    serial_num_t next_serial_num() { return _last_serial_num = write_req::next_serial_num(_last_serial_num); }
    decltype(auto) async_send<Token,BufferType>(const BufferType& buffer, serial_num_t serial_num, unsigned flags, Token&& token);
    void cancel();
    void resend();
    void operator(write_queue_t write_queue, error_code ec, size_t);
    void throttled_op_done();
};

class detail::replies {
    using Sig = void(error_code, byte_citer, byte_citer);
    static constexpr auto max_reply_time = std::chrono::seconds(20);
    class reply_handler {
        asio::any_completion_handler<Sig> _handler; control_code_e _code; uint16_t _packet_id; 
        std::chrono::time_point<std::chrono::system_clock> _ts;
    public: ctor<H>(control_code_e code, uint16_t pid, H&& handler); // allow move, no copy
        void complete(error_code ec, byte_citer first={}, byte_citer last={}) { std::move(_handler)(ec, first, last); }
        void complete_post(const executor_type& ex, error_code ec) { asio::post(ex, asio::prepend(std::move(_handler), ec, {}, {})); }
        uint16_t packet_id() const noexcept { return _packet_id; }
        control_code_e code() const noexcept { return _code; }
        auto time() const noexcept { return _ts; }
    };
    struct fast_reply { control_code_e code; uint16_t packet_id; std::unique_ptr<std::string> packet; };
    using handlers = std::vector<reply_handler>; using fast_replies = std::vector<fast_reply>;
    executor_type _ex; handlers _handlers; fast_replies _fast_replies;
    handlers::iterator find_handler(control_code_e code, uint16_t packet_id);
    fast_replies::iterator find_fast_reply(control_code_e code, uint16_t packet_id);
public: using executor_type = asio::any_io_executor;
    explicit ctor<Executor>(Executor ex); // allow move, no copy
    decltype(auto) async_wait_reply<Token>(control_code_e code, uint16_t packet_id, Token&& token);
    void dispatch(error_code ec, control_code_e code, uint16_t packet_id, byte_citer first, byte_citer last);
    void resend_unanswered();
    void cancel_unanswered();
    bool any_expired();
    void clear_fast_repies();
    void clear_pending_pubrels();
};
```

##### Auto Connect Stream

```c++
using epoints = asio::ip::tcp::resolver::results_type;
class detail::resolve_op<Owner,Handler> {
    using handler_type = Handler;
    struct on_resolve{};
    Owner& _owner; handler_type _handler;
    void complete(error_code ec, epoints eps, authority_path ap);
    void complete_post(error_code ec, epoints eps, authority_path ap);
public: ctor(Owner& owner, Handler&& handler); // allow move, no copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using cancellation_slot_type = asio::associated_cancellation_slot_t<Handler>;
    cancellation_slot_type get_cancellation_slot() const noexcept { return asio::get_associated_cancellation_slot(_handler); }
    using executor_type = asio::associated_executor_t<handler_type>;
    executor_type get_executor() const noexcept { return asio::get_associated_executor(_handler); }
    void perform();
    void operator()(on_resolve, std::array<size_t,2> ord, error_code resolve_ec, epoints epts, error_code timer_ec, authority_path ap);
};

class detail::endpoints<LoggerType> {
    using logger_type = LoggerType;
    asio::ip::tcp::resolver _resolver; asio::steady_timer& _connect_timer;
    std::vector<authority_path> _servers; int _current_host{-1}; log_invoke<logger_type>& _log;
public: ctor<Executor>(Executor ex, asio::steady_timer& timer, log_invoke<logger_type>& log); // no copy
    void clone_servers(const self& other) { _servers = other.servers; }
    using executor_type = asio::ip::tcp::resolver::executor_type;
    executor_type get_executor() noexcept { return _resolver.get_executor(); }
    decltype(auto) async_next_endpoint<Token>(Token&& token)
    { return asio::async_initiate<Token,void(error_code, epoints, authority_path)>(
        [](auto h, self& self){ resolve_op{self, std::move(h)}.perform(); }, token, std::ref(*this)); }
    void brokers(std::string hosts, uint16_t default_port);
};

class detail::autoconnect_stream<StreamType,StreamContext=std::monostate,LoggerType=noop_logger> {
    using stream_ptr = std::shared_ptr<stream_type>;
    executor_type _stream_executor; async_mutex _conn_mtx;
    asio::steady_timer _read_timer, _connect_timer; endpoints<logger_type> _endpoints;
    stream_ptr _stream_ptr; stream_context_type& _stream_context; log_invoke<logger_type>& _log;
    log_invoke<logger_type>& log() { return _log; }
    static void open_lowest_layer(const stream_ptr& sptr, asio::ip::tcp protocol);
    stream_ptr construct_next_layer() const;
    stream_ptr construct_and_open_next_layer(asio::ip::tcp protocol) const;
    void replace_next_layer(stream_ptr sptr);
    decltype(auto) async_reconnect<Token>(stream_ptr s, Token&& token)
    { return asio::async_initiate<Token,void(error_code)>([](auto h, self& self, stream_ptr s){ reconnect_op{self, std::move(h) }.perform(std::move(s)); },
        token, std::ref(*this), std::move(s)); }
public: using stream_type = StreamType; using stream_context_type = StreamContext;
    using logger_type = LoggerType; using executor_type = stream_type::executor_type;
    ctor(const executor_type& ex, stream_context_type& context, log_invoke<logger_type>& log); // disable copy
    using next_layer_type = stream_type;
    <const> next_layer_type& next_layer() <const> { return *_stream_ptr; }
    void brokers(std::string hosts, uint16_t default_port) { _endpoints.brokers(std::move(hosts), default_port); }
    void clone_endpoints(const self& other) { _endpoints.clone_servers(other._endpoints); }
    bool is_open() const noexcept { return lowest_layer(*_stream_ptr).is_open(); }
    void open() { open_lowest_layer(_stream_ptr, asio::io::tcp::v4()); }
    void cancel() { _conn_mtx.cancel(); _connect_timer.cancel(); }
    void close() { error_code ec; lowest_layer(*_stream_ptr).close(ec); }
    void async_shutdown<Token>(Token&& token)
    { return asio::async_initiate<Token,void(error_code)>([](auto h, self& self){ shutdown_op{self, std::move(h) }.perform(); }, token, std::ref(*this)); }
    bool was_connected() const { error_code ec, lowest_layer(*_stream_ptr).remote_endpoint(ec); return ec == success; }
    decltype(auto) async_read_some<BufferType,Token>(const BufferType& buffer, duration wait_for, Token&& token)
    { return asio::async_initiate<Token,void(error_code,size_t)>([](auto h, self& self, auto buf, auto wait_for){ read_op{self, std::move(h)}.perform(buf, wait_for); },
        token, std::ref(*this), buffer, wait_for); }
    decltype(auto) async_write<BufferType,Token>(const BufferType& buffer, Token&& token)
    { return asio::async_initiate<Token,void(error_code,size_t)>([](auto h, self& self, auto buf){ write_op{self, std::move(h)}.perform(buf); },
        token, std::ref(*this), buffer); }
};
```

##### Authenticator Wrapper

```c++
constexpr bool detail::is_authenticator<T> = requires Authenticator<T>;
class detail::auth_fun_base { // function pointer wrapper
  using auth_func = void(*)(auth_step_e, std::string, auth_handler_type, self*);
  auth_func _auth_func;
public: ctor(auth_func f);
  void async_auth(auth_step_e, std::string, auth_handler_type, self*); // forward to _auth_func;
};
class detail::auth_fun<Auth> requires is_authenticator<Auth> : public auth_fun_base {
  Auth _authenticator;
public: ctor(Auth authenticator) : base{&async_auth}, _authenticator{authenticator}{}
  static void async_auth(auth_step_e, std::string, auth_handler_type, base* ptr); // forward to ptr->_authenticator.async_auth(...);
};
class detail::any_authenticator {
  std::string _method; std::shared_ptr<auth_fun_base> _auth_fun;
public: ctor(); ctor<Auth>(Auth&& a) : _methd{a.method()}, _auth_fun{new auth_fun<Auth>{std::forward<Auth>(a)}} {}
  std::string_view method() const { return _method; }
  decltype(auto) async_auth<Token>(auth_step_e step, std::string data, Token&& token) {
    auto initiation = [](auto h, auto& self, auth step, auto data){ self._auth_fun->async_auth(step, std::move(data), std::move(h)); };
    return asio::async_initiate<Token,void(error_code,std::string)>(initiation, token, std::ref(*this), std::move(data));
  }
}
```

##### Async

```c++
struct tls_handshake<StreamType>{};
void assign_tls_sni<TlsContext, TlsStream>(const authority_path& ap, TlsContext& ctx, TlsStream& s);

struct ws_handshake_traits<Stream>{};

using detail::tracking_type<Handler,DfltExecutor> = decltype(...);
decltype(auto) tracking_executor<Handler, DfltExecutor>(const Handler& handler, const DfltExecutor& ex)
{ return asio::prefer(asio::get_associated_executor(handler, ex), asio::execution::outstanding_work.tracked); }

constexpr auto detail::handshake_handler_t = [](error_code){};
using detail::tls_handshake_t<T> = T::handshake_type;
using detail::tls_handshake_type_of<T> = boost::detected_or_t<void, tls_handshake_t, T>;
constexpr bool detail::has_tls_handshake<T> = ...; // detect T::async_handshake(handshake_type,handshake_handler_t)
constexpr bool detail::has_ws_handshake<T> = ...; // detect T::async_handshake(string_view,string_view,handshake_handler_t)
constexpr bool detail::has_next_layer<T> = ...; // detect T::next_layer()

using detail::next_layer_type<T> = ...; // T::next_layer_type;
next_layer_type<T>& detail::next_layer<T>(T&& a)
{ if constexpr (has_next_layer<T>) return a.next_layer(); else return std::forward<T>(a); }
using detail::lowest_layer_type<T> = ...; // T::next_layer_type::...::next_layer_type;
lowest_layer_type<T>& detail::lowest_layer<T>(T&& a)
{ if constexpr (has_next_layer<T>) return lowest_layer(a.next_layer()); else return std::forward<T>(a); }

constexpr bool detail::has_tls_layer<T> = ...; // has_tls_handshake on any layer of T
constexpr bool detail::has_tls_context<T> = ...; // T::tls_context()
void detail::setup_tls_sni<TlsContext, Stream>(const authority_path& ap, TlsContext& ctx, Stream& s); // assign_tls_sni on first TLS layer

constexpr bool detail::has_async_write<T,B> = ...; // T::async_write(B, write_handler_t);

decltype(auto) detail::async_write<Stream,ConstBufSeq,Token>(Stream& stream, const ConstBufSeq& buff, Token&& token); // T::async_write or asio::async_write

class detail::async_mutex {
    using queued_op_t = asio::any_completion_handler<void(error_code)>; using queue_t = std::deque<queued_op_t>;
    class tracked_op<Handler,Executor> { executor_type _executor; Handler _handler;
    public: ctor(Handler&& h, const Executor& ex); // allow move, disable copy
        using allocator_type = asio::associated_allocator_t<Handler>;
        allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
        using cancellation_slot_type = asio::associated_cancellation_slot_t<Handler>;
        cancellation_slot_type get_cancellation_slot() const noexcept { return asio::get_associated_cancellation_slot(_handler); }
        using executor_type = tracking_type<Handler,Executor>;
        executor_type get_executor() const noexcept { return _executor; }
        void operator()(error_code ec) { std::move(_handler)(ec); }
    };
    class cancel_waiting_op { queue_t::iterator _ihandler;
    public: explicit ctor(queue_t::iterator ih);
        void operator()(asio::cancellation_type_t type); };
    
    bool _locked{false}; queue_t waiting; executor_type _ex;

    void execute_op(queued_op_t op);
    void execute_or_queue<Handler>(Handler&& handler) noexcept;
public: explicit ctor<Executor>(Executor&& ex) : _ex{std::forward<Executor>(ex)}{}  ~dtor(){cancel();} // disable copy
    const executor_type& get_executor() const noexcept { return _ex; }
    bool is_locked() const noexcept { return _locked; }
    decltype(auto) lock<Token>(Token&& token) noexcept;
    void unlock();
    void cancel();
};

class detail::cancellable_handler<Handler,Executor> {
    Executor _executor; Handler _handler; tracking_type<Handler,Executor> _handler_ex; asio::cancellation_state _cancellation_state;
public: ctor(Handler&& handler, const Executor& ex); // allow move, disable copy
    using allocator_type = asio::associated_allocator_t<Handler>;
    allocator_type get_allocator() const noexcept { return asio::get_associated_allocator(_handler); }
    using cancellation_slot_type = asio::associated_cancellation_slot_t<Handler>;
    cancellation_slot_type get_cancellation_slot() const noexcept { return _cancellation_state.slot(); }
    using executor_type = tracking_type<Handler,Executor>;
    executor_type get_executor() const noexcept { return _handler_ex; }
    using immediate_executor_type = asio::associated_immediate_executor_t<Handler,Executor>;
    immediate_executor_type get_immediate_executor() const noexcept { return asio::get_associated_immediate_executor(_handler, _executor); }
    asio::cancellatin_type_t cancelled() const { return _cancellation_state.cancelled(); }
    void complete<...Args>(Args&&...args);
    void complete_immediate<...Args>(Args&&...args);
};

class detail::bounded_deque<Element> {
    std::deque<Element> _buffer; static cosntexpr size_t MAX_SIZE = 65535;
public: ctor(); ctor(size_t n) : _buffer(n){}
    size_t size() const; void push_back<E>(E&& e); void pop_front(); void clear(); <const> auto& front() <const> noexcept;
};
struct detail::channel_traits <...Sigs> { struct rebind<...NewSigs> { using other = channel_traits<NewSigs...>; }; };
struct detail::channel_traits <R(error_code,Args...)> {
    struct rebind<...NewSigs> { using other = channel_traits<NewSigs...>; };
    struct container<Element> { using type = bounded_deque<Element>; };
    using receive_cancelled_signature = R(error_code, Args...);
    static void invoke_receive_cancelled<F>(F f);
    using receive_closed_signature = R(error_code, Args...);
    static void invoke_receive_closed<F>(F f);
};

struct detail::rebind_executor<Stream,Executor> { using other = Stream::rebind_executor<Executor>::other; };
struct detail::rebind_executor<asio::ssl::stream<Stream>,Executor> { using other = asio::ssl::stream<rebind_executor<Stream,Executor>::other>; };
struct detail::rebind_executor<beast::websocket::stream<asio::ssl::stream<Stream>, deflate_supported>, Executor>
{ using other = beast::websocket::stream<asio::ssl::stream<typename rebind_executor<Stream, Executor>::other>,deflate_supported>; };

void detail::async_shutdown<Stream,ShutdownHandler>(Stream& ShutdownHandler&&){ static_assert(false); } // unknown Stream Type
void detail::async_shutdown<P,E,H>(asio::basic_stream_socket<P,E>& socket, H&& handler)
{ error_code ec; socket.shutdown(shutdown_both, ec); return std::move(handler)(ec); }
void detail::async_shutdown<Stream,H>(asio::ssl::stream<Stream>& socket, H&& handler) { stream.async_shutdown(std::move(handler)); }

struct detail::ws_handshake_traits<beast::websocket::stream<Stream>>
{ static decltype(auto) async_handshake<Token>(beast::websocket::stream<Stream>& stream, authority_path ap, Token&& token); };
void detail::async_shutdown<Stream,H>(beast::websocket::stream<Stream>& stream, H&& handler) { stream.async_close(normal, std::move(handler)); }
```

##### Utils

```c++
constexpr bool detail::is_optional<T>; // std::optional<U>
constexpr bool detail::is_specialization<T,Templ<...>>; // T is Templ<...>
constexpr bool detail::is_vector<T> = is_specialization<remove_cv_ref_t<T>, std::vector>;
constexpr bool detail::is_small_vector<T> = ...; // derived from small_vector_base<...>
constexpr bool detail::is_pair<T> = is_specialization<remove_cv_ref_t<T>, std::pair>;
constexpr bool detail::is_boost_iterator<T> = is_specialization<remove_cv_ref_t<T>, boost::iterator_range>;

enum class detail::validation_result : uint8_t { valid, has_wildcard_character, invalid };
int detail::pop_front_unichar(std::string_view& s);
validation_result detail::validate_mqtt_utf8_char(int c);
bool detail::is_valid_string_size(size_t sz);
bool detail::is_utf8(validation_result result);
validation_result detail::validate_mqtt_utf8(std::string_view str);
bool detail::is_valid_string_pair(const std::pair<std::string, std::string>& str_pair);

static constexpr int32_t detail::min_subscription_identifier=1, max_subscription_identifier=268'435'455;
static constexpr std::string_view detail::shared_sub_prefix = "$share/";
bool detail::is_utf8_no_wildcard(validation_result result);
bool detail::is_not_empty(size_t sz);
bool detail::is_valid_topic_size(size_t sz);
validation_result detail::validate_topic_name(std::string_view str);
validation_result detail::validate_topic_alias_name(std::string_view str);
validation_result detail::validate_shared_topic_name(std::string_view str);
validation_result detail::validate_topic_filter(std::string_view str);
validation_result detail::validate_shared_topic_filter(std::string_view str);
```

------
### Dependency

#### Boost.ASIO

* `<boost/asio/**.hpp>`

#### Boost.Assert

* `<boost/assert.hpp>`

#### Boost.Beast

* `<boost/beast/**.hpp>`

#### Boost.Container

* `<boost/container/small_vector.hpp>`

#### Boost.Core

* `<boost/core/identity.hpp>`

#### Boost.Endian

* `<boost/endian/conversion.hpp>`

#### Boost.Random

* `<boost/random/linear_congruential.hpp>`
* `<boost/random/uniform_smallint.hpp>`

#### Boost.Range

* `<boost/range/iterator_range_core.hpp>`

#### Boost.SmartPtr

* `<boost/smart_ptr/allocate_unique.hpp>`

#### Boost.System

* `<boost/system/error_code.hpp>`

#### Boost.TypeTraits

* `<boost/type_traits/*.hpp>`

------
### Standard Facilities
