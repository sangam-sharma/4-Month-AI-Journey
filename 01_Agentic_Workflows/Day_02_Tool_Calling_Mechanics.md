# Day 2: Demystifying Tool Calling & Function Calling

## 🧠 Core Concept
Tool calling (or function calling) gives LLMs the ability to interact with the outside world—like querying databases, looking up stock prices, or reading local files. 

While it might sound "spooky" or magical that an LLM is running code on your machine, the reality is much simpler: **an LLM is strictly a token generator.** It never executes tools directly. All agentic tool execution relies on surrounding application software, smart system prompting, JSON schemas, and basic conditional logic (`if` statements).

---

## 🎭 Tool Calling: Perception vs. Reality

### 1. The Common Misconception (The "Magic" View)
Beginners often assume the LLM directly connects to external systems and runs tools on its own:

```text
User <---> Software <---> LLM
                           |
                           v
                        [Tools]  <-- INCORRECT: LLM doesn't call tools directly!
                    
### The Architectural Reality
## In reality, the software sits in the middle and handles all execution. The LLM only talks to the software using text/JSON:
User <---> Software <---> LLM
              |
              v
           [Tools]               <-- CORRECT: Software executes tools locally!

Here is the exact cycle of how software, LLMs, and tools communicate behind the scenes:

[1. Prompt + Tools Schema]
User  <--->  Software  ------------------------>  LLM
                ^                                  |
                |                                  | [2. Returns JSON Tool Intent]
                |                                  v
                +---- [3. Software Executes Tool] -+
                |
                v
[4. Sends Tool Result back to LLM] -------------> LLM
                                                   |
[5. Final Answer to User] <-----------------------+

Breakdown of the Loop:
Request + Schema: The user asks a question (e.g., "What's inside data.txt?"). The software packages this request alongside a JSON schema defining available functions (e.g., read_file(filename)).

JSON Intent Generation: The LLM inspects the prompt and tools schema. Instead of answering directly, it outputs a JSON string: {"tool": "read_file", "args": {"filename": "data.txt"}}.

Local Software Execution: The software receives the JSON, checks a condition (if response.has_tool_call), intercepts the request, and executes the actual Python/system function locally to read the file.

Context Injection: The software takes the raw output from the file and sends it back to the LLM as a new message.

Final Answer: The LLM reads the tool output and generates a human-friendly response back to the software, which displays it to the user.