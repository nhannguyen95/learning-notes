---

mindmap-plugin: basic

---

# Java Concurrency

## #process-vs-thread
- #process
   - Isolated, independently executing program
- #thread
   - Sometimes called #lightweight-process
   - A process can spawn multiple threads
   - Same-process threads share process-wide resources (e.g. memory address space)
   - Each thread has its own program counter, stack, local vars
   - Basic unit of scheduling in modern OSs

## References
- #java-concurrency-in-practice-2006

## Threads' risks
- Safety hazards
   - Safety = nothing bad ever happens
   - Examples
      - #race-condition
- Liveness hazards
   - Liveness = something good eventually happens
   - Examples
      - #deadlock
      - #starvation
      - #livelock
- Performance hazards