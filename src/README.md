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

## Task 1: Exclusive Ownership of `_chatLogic`

### Implementation

- `src/chatgui.cpp`: `ChatBotPanelDialog::ChatBotPanelDialog(*parent, id)`

  Changed the type of `_chatLogic` to be a `unique_ptr` class of type `ChatLogic` in order for it to be an exclusive resource to class `ChatBotPanelDialog`. Given that the type of `_chatLogic` was changed from `new` to `unique_ptr`, it is also necessary to remove the call of `delete` inside the `~ChatBotPanelDialog` destructor.

- `src/chatgui.h`: `ChatBotDialog`

  Changed the declaration type of `_chatLogic`. Also, given that the type of the pointer is changed, it is used `_chatLogic.get()` in order to correctly return the stored pointer.

### Result

No changes are observed in the functionality of the program, but the ownership of `_chatLogic` is unique.

## Task 2: The Rule of Five

### Implementation

  In order to comply with the Rule Of Five, the `copy constructor`,  `move constructor`, `copy assignment operator` and `move assignment operator` were introduced. The implementation was based on the `Rule Of Five` section from the `Move Semantics` lesson. Some adaptations were done based on comments in the Knowledge Section.

- `src/chatbot.cpp`: `ChatBot::ChatBot(const ChatBot &source)`

  **Copy Constructor**: TBD

- `src/chatbot.cpp`: `ChatBot::ChatBot(const ChatBot &&source)`

  **Move Constructor**: TBD

- `src/chatbot.cpp`: `ChatBot& ChatBot::operator=(const ChatBot &source)`

  **Copy Assignment Operator**: TBD

- `src/chatbot.cpp`: `ChatBot& ChatBot::operator=(const ChatBot &&source)`

  **Move Assignment Operator**: TBD

## Task 3: Exclusive Ownership of `_nodes`

### Implementation

- `src/chatlogic.h`: `class ChatLogic`

  Declaration for vector of raw pointers `_nodes` was transformed into a vector of `unique_ptr` of type `GraphNode`.

- `src/chatlogic.cpp`: `ChatLogic()::~ChatLogic()`

  Given that we are now working with a smart pointer of type `unique_ptr` it is no longer necessary to explicitly delete the pointer when destroying class `ChatLogic`, this is handled by the `unique_ptr` class.

- `src/chatlogic.cpp`: `ChatLogic::LoadAnswerGraphFromFile(filename)`

  In the declaration and initialization of `newNode`, `parentNode` and `childNode`, a raw pointer `GraphNode *node` was returned by the `std::find_if` function.. This was replaced with the correct `unique_ptr` type instead. Given then that the variables mentioned previously are now `unque_ptr`s, the calls of `SetChildNode` and `SetParentNode` to the new edge must use the correct function of `.get()` used for `unique_ptr`s.

  The assignment of the current node to `root` is also changed to use the correct functionality to access the `unique_ptr`. (solution for this one referred [here](https://knowledge.udacity.com/questions/120851))

## Task 4: Moving Smart Pointers

### Implementation

  - `src/graphnode.h`: `class GraphNode`

  Declaration for vector of raw pointers `_childEdges` was transformed into a vector of `unique_ptr` of type `GraphEdge`, as these are data handles owned by GraphNode.

  - `src/graphnode.cpp`: `GraphNode::AddEdgeToChildNode(edge)`

  This function is called in `ChatLogic::LoadAnswerGraphFromFile(filename)` where the passed variable `edge` is of type `unique_ptr` and as such its declaration is changed as well as it's moved to `_childedges` with the `std::move` function.

  - `src/chatlogic.h`: `class chatLogic`

  Declaration for vector of raw pointers `_edges` was transformed into a vector of `unique_ptr` of type `GraphEdge`.

  - `src/chatlogic.cpp`: `ChatLogic::LoadAnswerGraphFromFile(std::string filename)`

  the variable `edge` becomes a unique pointer, as required by this task. Given that, the storing of child and parent node need to be adapted to correctly call the unique pointer, i.e., use `.get()` and `std::move(edge)` respectively.


