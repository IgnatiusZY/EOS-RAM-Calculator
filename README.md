# EOS-RAM-Calculator
EOS RAM Calculator

For each Table (Overhead)
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

For each Row (Overhead)
```
struct key_value_object : public chainbase::object<key_value_object_type, key_value_object> {
   OBJECT_CTOR(key_value_object, (value))

   typedef uint64_t key_type;
   static const int number_of_keys = 1;

   id_type        id;                                                ~= 16 bits
   table_id       t_id;                                              ~= 16 bits
   uint64_t       primary_key;                                       ~= 64 bits
   account_name   payer = 0;                                         ~= 64 bits
   shared_blob    value;                                             -> Row|Record
};
```

The Chainbase is the Boost Multi Index Table.

** Each unique Code-Scope is a different table_id_object with a single key_value_object (-> table_id_object & key_value_object Overhead Cost occur for each Record).

** For a non-unique Code-Scope (e.g. Same Code == Scope (get_self(), get_self().value)) (-> 1 table_id_object Overhead Cost & key_value_object Overhead Cost for each new Record).
