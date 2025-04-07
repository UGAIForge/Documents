---

## 🗂️ Agent Configuration Data Structure

### 📄 Example JSON Structure of an Agent

```json
{
  "agent_name": "Agent name",
  "description": "Agent description",
  "context": {
    "context_xxx": {
      "type": "persona",
      "content": {
        "persona_xxx": {
          "key": "key1",
          "value": "value1"
        },
        "persona_xxx": {
          "key": "key2",
          "value": "value2"
        }
      }
    },
    "context_xxx": {
      "type": "audience",
      "content": {
        "audience_xxx": {
          "key": "key1",
          "value": "value1"
        },
      }
    }
  },
  "workers": {
    "worker_xxx": {
      "label": "worker name",
      "description": "worker description",
      "entry": false,
      "settings": {
        "input": "this is input",
        "output": "output to this value {{ref:data:data_xxx}}",
        "name": "gpt-4o",
        "temperature": 0.7,
        "max_output_tokens": 1000,
        "context_window": 8000
      },
      "spl": {
        "flows": [
          {
            "type": "main_flow",
            "label": "Main Flow",
            "description": "main flow description",
            "flow_content": [
              {
                "type": "command",
                "command": "run command with {{ref:data:data_xxx}}"
              },
              {
                "type": "if_else",
                "condition": {
                  "description": "Check value",
                  "expression": "{{ref:data:value_1}} == 222"
                },
                "true_flow": [
                  {
                    "type": "command",
                    "command": "true branch command"
                  }
                ],
                "false_flow": [
                  {
                    "type": "command",
                    "command": "false branch command"
                  }
                ]
              },
              {
                "type": "loop",
                "condition": {
                  "description": "Repeat if true",
                  "expression": "{{ref:data:value_1}} == 222"
                },
                "true_flow": [
                  {
                    "type": "command",
                    "command": "looped command"
                  }
                ]
              }
            ]
          },
          {
            "type": "alternative_flow",
            "label": "Alternative Flow",
            "description": "alternative path description",
            "condition": {
              "description": "Trigger when X",
              "expression": "{{ref:data:condition_data}} == true"
            },
            "flow_content": []
          },
          {
            "type": "exception_flow",
            "label": "Exception Flow",
            "description": "handle error cases",
            "condition": {
              "description": "Trigger when error",
              "expression": "{{ref:data:error_flag}} == true"
            },
            "flow_content": []
          }
        ]
      },
      "use_cases": [
        {
          "name": "use_case_1",
          "description": "description for use case 1",
          "code": "example code for this use case"
        }
      ],
      "code": {
        "language": "python",
        "content": "full code implementation here",
        "test_cases": [
          {
            "name": "test_case_1",
            "description": "test description",
            "code": "assert some_func() == expected"
          }
        ]
      },
      "diagrams": [
        {
          "name": "architecture_diagram",
          "language": "mermaid",
          "diagram": "graph TD; A-->B;"
        }
      ]
    }
  }
}
```
