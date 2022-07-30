# Factory Method Pattern

Design Pattern Type
: Creational [^1]

Objective
: Create a class without exposing the mechanism to the user through a common interface (base class)

Requirement
1. Factory should be a singleton
2. Register the object with factory         (factory's dictionary)
3. Design a way to use creation via factory (object_wrappers)


Please see [Proxy pattern](2_proxy.md) for its interaction with Proxy

----

## Implementation in SystemVerilog
```systemverilog
    // From Proxy patterns page
    Wrapper #(Parent)           p = new;
    Wrapper #(AdoptedParent)    ap = new;

    Proxy       factory[string];                        // create a factory

    Parent p_h;                                         // Common interface

    begin
        factory["Parent"] = p;                          // Register onto factory
        $cast( p_h, factory["Parent"].get() );          // get method

        factory["Parent"] = ap;                         // Register onto factory (override)
        $cast( p_h, factory["AdoptedParent"].get() );   // get method
    end
```

----

## Implementation in UVM
```systemverilog
    // Here is the abstract factory class in base/uvm_factory.svh
    virtual class uvm_factory;
        static function uvm_factory         get();                                  // get method for singleton factory

        pure virtual function void          register( uvm_object_wrapper obj );     // register into factory

        pure virtual function void          set_inst_override_by_type( ... );       // override method
        pure virtual function void          set_inst_override_by_name( ... );       // override method
        pure virtual function void          set_type_override_by_type( ... );       // override method
        pure virtual function void          set_type_override_by_name( ... );       // override method

        pure virtual function uvm_object    create_object_by_type( ... );           // Creation method
        pure virtual function uvm_object    create_object_by_name( ... );           // Creation method
        pure virtual function uvm_object    create_component_by_type( ... );        // Creation method
        pure virtual function uvm_object    create_component_by_name( ... );        // Creation method

        // Other methods:  find_override_*, find_wrapper_by_name, print
    endclass :factory

    // Default Factory class
    class uvm_default_factory extends uvm_factory;
        ...
        // private members
        protected bit                       m_types[uvm_object_wrapper];                    // dictionary[type]      exist?
        protected bit                       m_lookup_strs[string];                          // dictionary[type name] exist?
        protected uvm_object_wrapper        m_type_names[string];                           // Store proxy by typename

        // overrides
        protected uvm_factory_override      m_type_overrides[$];                            // queue of "type override"
        protected uvm_factory_queue_class   m_inst_override_queues[uvm_object_wrapper];     // queue of "inst override"
        protected uvm_factory_queue_class   m_inst_override_name_queues[string];            // in case if override was called before register
        protected uvm_factory_override      m_wildcard_inst_overrides[$];                   // wildcard usage.  regex done by DPI
        local uvm_factory_override          m_override_info[$];                             // temp array for find_override_*
        ...
    endclass :uvm_factory

    function void uvm_default_factory::register( uvm_object_wrapper obj );
        // 1. check if null
        // 2. check if string(typename) is valid [warning]
        //    if so, push into "m_type_names" dictionary
        // 3. check if it was already registered..  [warning]
        //    if new, push into m_inst_override_queues
    endfunction : register

    function void uvm_default_factory::set_type_override_by_type(   uvm_object_wrapper original_type, 
                                                                    uvm_object_wrapper override_type,
                                                                    bit replace=1);
        // 1. check that original and override is not same [warning]
        // 2. register original and override type if new
        // 3. check for duplicates in m_type_overrides queue
        // 4. make a new override and push it into m_type_overrides
    endfunction : set_type_override_by_type

    function uvm_object uvm_default_factory::create_object_by_type( uvm_object_wrapper requested_type,
                                                                    string parent_inst_path="",
                                                                    string name="" );
        // 1. Flush m_override_info
        // 2. Find override by type
        //    if found, use the found object and call its create_object method
        //    else,     use input object     and call its create_object method
    endfunction : create_object_by_type    
```
---

## Object Registration
UVM defines the infrastructure of an object through uvm utility macro
> `uvm_object_utils( T )

Which has 3 main parts:

   1. `m_uvm_object_registry_internal(T,T)
      1. uvm_object_registry #(T,"T") is aliased into ***type_id***
      2. append ***get_type()*** static method
      3. append ***get_object_type()*** method
   
   2. `m_uvm_object_create_func(T)
      1. append ***create()*** method

   3. `m_uvm_get_type_name_func(T)
      1. assign ***type_name*** as static string
      2. append ***get_type_name()*** method
   
----

#### QUICK QUESTION: What is the difference between uvm_object::create() vs uvm_object_wrapper::create_object()
- [**uvm_object_wrapper**]  - Proxy (*uvm_object_registry*) implements ***create_object*** to create object of a specfic type
- [**uvm_object**] - m_uvm_object_create_func(T) appends create common method, but we do not use it directly.
  - We call uvm_object_registry#(T,T)::create() ....... or rather ..... type_id::create()
    - which gets factory context and call uvm_factory::create_component_by_type() method.

---
#### VERY ODD CODING STYLE
```systemverilog
    static function T create( string name, uvm_component parent, string contxt="" );
        ... 
        obj = factory.create_object_by_type(get(), contxt, name);
        if (!cast( create, obj )) begin                     // uvm_object casted into "create"
        //                                                     instead of using return call.. 
        //                                                     it casted onto the function create - NEAT!
        end
    endfunction
```


[^1]: [Design Patterns: Elements of Reusable Object-Oriented Software](https://springframework.guru/gang-of-four-design-patterns/)
