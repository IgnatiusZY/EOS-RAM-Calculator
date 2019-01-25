# EOS-RAM-Calculator
EOS RAM Calculator

Chainbase === Boost Multi Index Table

`https://github.com/EOSIO/eos/blob/master/libraries/chain/include/eosio/chain/contract_table_objects.hpp`

`https://github.com/EOSIO/eos/blob/master/libraries/chain/include/eosio/chain/config.hpp`

Each EOSIO Multi-Index Table
`contract_table_objects.hpp`
```
class table_id_object : public chainbase::object<table_id_object_type, table_id_object> {
   OBJECT_CTOR(table_id_object)

   id_type        id;                                                ~= 16 bits
   account_name   code;                                              ~= 64 bits
   scope_name     scope;                                             ~= 64 bits
   table_name     table;                                             ~= 64 bits
   account_name   payer;                                             ~= 64 bits
   uint32_t       count = 0; /// The number of elements in the table ~= 32 bits
};
```

Each EOSIO Multi-Index Row
`contract_table_objects.hpp`
```
struct key_value_object : public chainbase::object<key_value_object_type, key_value_object> {
   OBJECT_CTOR(key_value_object, (value))

   typedef uint64_t key_type;
   static const int number_of_keys = 1;

   id_type        id;                                                ~= 16 bits
   table_id       t_id;                                              ~= 16 bits
   uint64_t       primary_key;                                       ~= 64 bits
   account_name   payer = 0;                                         ~= 64 bits
   shared_blob    value;                                             -> Inherited from shared_string ~ Pointer (i.e. 32 bits on a 32-bit machine OR a 64 bits on a 64-bit machine)
};
```

`contract_table_objects.hpp`
```
template<>
struct billable_size<table_id_object> {
   static const uint64_t overhead = overhead_per_row_per_index_ram_bytes * 2;  ///< overhead for 2x indices internal-key and code,scope,table
   static const uint64_t value = 44 + overhead; ///< 44 bytes for constant size fields + overhead
};

template<>
struct billable_size<key_value_object> {
   static const uint64_t overhead = overhead_per_row_per_index_ram_bytes * 2;  ///< overhead for potentially single-row table, 2x indices internal-key and primary key
   static const uint64_t value = 32 + 8 + 4 + overhead; ///< 32 bytes for our constant size fields, 8 for pointer to vector data, 4 bytes for a size of vector + overhead
};

template<>
struct billable_size<index64_object> {
   static const uint64_t overhead = overhead_per_row_per_index_ram_bytes * 3;  ///< overhead for potentially single-row table, 3x indices internal-key, primary key and primary+secondary key
   static const uint64_t value = 24 + 8 + overhead; ///< 24 bytes for fixed fields + 8 bytes key + overhead
};

template<>
struct billable_size<index128_object> {
   static const uint64_t overhead = overhead_per_row_per_index_ram_bytes * 3;  ///< overhead for potentially single-row table, 3x indices internal-key, primary key and primary+secondary key
   static const uint64_t value = 24 + 16 + overhead; ///< 24 bytes for fixed fields + 16 bytes key + overhead
};

template<>
struct billable_size<index256_object> {
   static const uint64_t overhead = overhead_per_row_per_index_ram_bytes * 3;  ///< overhead for potentially single-row table, 3x indices internal-key, primary key and primary+secondary key
   static const uint64_t value = 24 + 32 + overhead; ///< 24 bytes for fixed fields + 32 bytes key + overhead
};

template<>
struct billable_size<index_double_object> {
   static const uint64_t overhead = overhead_per_row_per_index_ram_bytes * 3;  ///< overhead for potentially single-row table, 3x indices internal-key, primary key and primary+secondary key
   static const uint64_t value = 24 + 8 + overhead; ///< 24 bytes for fixed fields + 8 bytes key + overhead
};

template<>
struct billable_size<index_long_double_object> {
   static const uint64_t overhead = overhead_per_row_per_index_ram_bytes * 3;  ///< overhead for potentially single-row table, 3x indices internal-key, primary key and primary+secondary key
   static const uint64_t value = 24 + 16 + overhead; ///< 24 bytes for fixed fields + 16 bytes key + overhead
```

`config.hpp`
```
const static uint64_t billable_alignment = 16;

const static uint32_t   overhead_per_row_per_index_ram_bytes = 32;    ///< overhead accounts for basic tracking structures in a row per index
```

`config.hpp`
```
template<typename T>
constexpr uint64_t billable_size_v = ((billable_size<T>::value + billable_alignment - 1) / billable_alignment) * billable_alignment;
```

`Net Billable RAM Cost = (<no_of_struct_unique_scopes> * config::billable_size_v<table_id_object>) + config::billable_size_v<key_value_object> + <contract_struct_row_size>`

`https://github.com/EOSIO/eos/issues/4532` ~= 240 BYTES!

Amount of Code
```
Amount of Code = code_size (sizeof(.wasm)) * setcode_ram_bytes_multiplier (10)
```

Net RAM for DApp
```
Net RAM for DApp = Amount of Code + Net Billable RAM Cost
```