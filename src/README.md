# Notes on the Project's Implementation

The base code in the repo is forked from Udacity's C++ Course ([repo](https://github.com/udacity/CppND-Memory-Management-Chatbot)).

## Task 0: Eliminate Segmentation fault when program exits

### Implementation

- `src/graphnode.cpp`: `GraphNode::~GraphNode()` - remove line `delete _chatBot();`

  The reason for the segmentation fault when the program exits was due to an additional `delete _chatBot;` instance in the `GraphNode()` destructor, when this command had already been performed in the `ChatLogic::~ChatLogic()` destructor in `src/chatlogic.cpp`. The latter destructor is performed before `~Graphnode()` as can be seen when executing the following commands in `gdb`:

  ```bash
  $> gdb ./membot
  (gdb) run     // Let the program run, then exit the program. This creates a segmentation fault, as well as a coredump.
  [...]
  0x000000000040e62e in ChatBot::~ChatBot() ()
  (gdb) where
  #0  0x000000000040e62e in ChatBot::~ChatBot() ()
  #1  0x00000000004220be in GraphNode::~GraphNode() ()
  #2  0x000000000041b259 in ChatLogic::~ChatLogic() ()
  [...]
  ```

### Result

No more segmentation fault when exiting the program
