# TLM1 Port - One-to-One relationship

### PULL vs PUSH
TLM has a relationship between...
- PRODUCER (Produce transaction)
- CONSUMER (Consume transaction)


### Producer -> Consumer : push
```mermaid
flowchart LR
UVM_MONITOR-->|PUSH by put method| UVM_SCOREBOARD
```

This is when we try to send a transaction to the consumer.
If you need a single 


### Consumer <- Producer : pull
```mermaid
flowchart RL
UVM_SEQUENCER-->|PULL by get method|UVM_DRIVER
```

