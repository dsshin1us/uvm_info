# Singleton Design Pattern


Design Pattern Type
: Creational [^1]

Objective
: Create an object of a single instance for a global coordination

**Requirement**
1. Only a single instance
2. Easily access
3. Control its instantiation

**Implementation in Systemverilog**
```verilog
    class Singleton;
        protected static Singleton  m_self;                     // Local static instance

        protected function new();                               // Protected new method
        endfunction : new

        static function get();                                  // Static Get method
            if (m_self==null)       m_self = new;               // Create iff uncreated
            return                  m_self;
        endfunction : get
    endclass : Singleton

    // Call
    Singleton s = Singleton::get();
```

**Examples in UVM**
```verilog
    // Singletons in the UVM
    //  + uvm_coreservice_t
    //  + uvm_factory
    //  + uvm_root
    //  + uvm_enum_wrapper
    //  + uvm_resource_db
    //  + uvm phases
    
    class uvm_reset_phase extends uvm_task_phase; 
        virtual task exec_task(uvm_component comp, uvm_phase phase); 
            comp.reset_phase(phase); 
        endtask
        local static uvm_reset_phase m_inst;                    // Local Static Instance
        static const string type_name = "uvm_reset_phase";

        // Function: get
        // Returns the singleton phase handle 
        static function uvm_reset_phase get();                  // Static Get method
            if(m_inst == null)
                m_inst = new;                                   // Create iff uncreated
            return m_inst; 
        endfunction
        protected function new(string name="reset");            // Protected new method
            super.new(name); 
        endfunction
        virtual function string get_type_name(); 
            return type_name; 
        endfunction
    endclass

    // Call
    uvm_reset_phase     m_reset_ph = uvm_reset_phase::get();    // uvm_phase is not registered in factory
``` 




[^1]: [Design Patterns: Elements of Reusable Object-Oriented Software](https://springframework.guru/gang-of-four-design-patterns/)