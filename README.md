# Proposal: Inline Pre- and Post-Processing Decorators for Node-RED Nodes

## 📉 Summary

This proposal introduces optional **inline decorators** to all Node-RED nodes with inputs and/or outputs, enabling users to define lightweight logic blocks directly within the node editor. These decorators allow for pre-processing of incoming messages and post-processing of outgoing messages—reducing the need for separate `function` nodes used solely for trivial transformations or glue logic.

The goal is to **declutter flows**, **improve readability**, and **enhance maintainability**, while keeping the feature completely **opt-in** and compatible with existing flows.

---

## 🧠 Motivation

Node-RED users frequently insert `function` nodes into their flows to perform small operations like:

- Adjusting a numeric payload (`msg.payload *= 100`)
- Filtering messages (`if (!msg.payload) return null`)
- Adding a topic or timestamp (`msg.topic = 'temp'`)
- Serializing data (`msg.payload = JSON.stringify(msg.payload)`)

While function nodes are powerful, their overuse—especially for simple logic—can create visually noisy flows that are harder to read and maintain. This proposal introduces a cleaner alternative.

---

## 🌟 Proposed Enhancement

### For Any Node with Inputs

- **Pre-Processing Decorator**
  - Executed **before** the message enters the node's core logic.
  - Allows optional logic such as validation, normalization, or enrichment.
  - Syntax: JavaScript (same as function nodes)

### For Any Node with Outputs

- **Post-Processing Decorator**
  - Executed **after** the node has generated its output message(s).
  - Allows for formatting, tagging, or filtering before output is sent.

---

## 🖼 Visual and UX Design

- Nodes with active decorators show a **small visual indicator**:
  - Example: a colored dot, ellipsis (`…`) button, or corner highlight.
- Clicking the indicator opens a **popup or inline code editor**.
- Minimal by default, expandable as needed (multi-line).
- Hints and autocomplete for `msg`, `context`, etc., provided just like in function nodes.

---

## ✨ Example

**Without enhancement:**

```
[ Inject ] → [ Function: msg.payload *= 100 ] → [ Debug ]
```

**With enhancement:**

```
[ Inject (post: msg.payload *= 100) ] → [ Debug ]
```

---

## 🧹 Handling Multiple Inputs and Outputs

### Multi-Input Nodes

- **Default**: A single pre-processing block applies to all incoming messages.
- **Advanced Option**: Enable per-input logic using a helper field like `msg._inputIndex`.

```js
switch (msg._inputIndex) {
  case 0: msg.source = "sensorA"; break;
  case 1: msg.source = "sensorB"; break;
}
return msg;
```

- **UI**: Tabbed editor or dropdown for each input, with visual indicators.

### Multi-Output Nodes

- **Default**: A single post-processor block modifies all outgoing messages.
- **Advanced Option**: Enable per-output post-processing.

```js
return [
  (msgs[0] ? decorateOutput0(msgs[0]) : null),
  (msgs[1] ? decorateOutput1(msgs[1]) : null),
  null
];
```

- **UI**: Per-output code editors, labeled clearly, with tooltip support.

---

## ✅ Benefits

| Feature                     | Benefit                                         |
|-----------------------------|--------------------------------------------------|
| Inline logic blocks         | Reduces need for function-node "glue"            |
| Cleaner flows               | Fewer nodes overall; more readable                |
| Faster editing              | One-click access to local transform logic         |
| Compatible with existing flows | Doesn't break or replace existing behaviors     |
| Optional complexity         | Simple by default, powerful when needed           |

---

## 🚀 Future Enhancements

### 1. Refactor Tool for Existing Flows

An automated tool could analyze existing flows and **migrate simple `function` nodes** into nearby nodes as pre- or post-processing decorators.

- Detect trivial transformations (1–3 lines of code)
- Identify upstream or downstream target nodes
- Suggest or apply decorator inlining
- Leave advanced logic untouched

This would help users **modernize legacy flows** and unlock the benefits of cleaner designs with minimal effort.

### 2. Pre/Post Hooks for Subflows

Enable reusable decorators in subflows with global or parameterized pre- and post-hooks.

### 3. Library of Common Snippets

Provide a library of reusable inline decorators for:

- Clamping values
- Topic tagging
- JSON conversions
- Message filtering

### 4. Inline Debugging

Add decorator-level debug logging or tracing to monitor pre- and post-processing impact.

---

## 📬 Closing Thoughts

This proposal enhances Node-RED’s low-code philosophy by reducing visual noise without reducing expressive power. It’s a small but impactful improvement that:

- Empowers both novice and advanced users
- Speeds up development
- Makes flows easier to share, teach, and maintain

The decorators remain completely optional, allowing users to adopt them gradually, and making this a **non-disruptive, user-friendly upgrade**.

---

## ✏️ Feedback & Next Steps

We welcome comments from Node-RED maintainers, power users, and node authors. Suggested areas for discussion:

- Technical feasibility and performance impact
- UI/UX design preferences for the decorator interface
- Metadata approach for identifying inputs/outputs
- Phased implementation strategy (experimental → core)

