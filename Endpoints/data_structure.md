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
        "input": {
          "required": "required input",
          "optional": "optional input"
        },
        "output": {
          "required": "output to this value {{ref:data:data_xxx}}",
          "optional": "optional output"
        },
        "guardrails": "guardrail 1",
        "model":{
          "name": "gpt-4o",
          "temperature": {
              "min": 0,
              "max": 1,
              "default": 0.7
          },
          "max_output_tokens": {
              "min": 1,
              "max": 8192,
              "default": 4096
          },
          "context_window": {
              "min": 1,
              "max": 200000,
              "default": 100000
          }
        }
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
                "description": "Check value",                                  
                "if_flow": [
                  {
                    "type": "command",
                    "command": "if branch command",
                    "expression": "{{ref:data:value_1}} == 222"
                  }
                ],
                "else_if_flow": [
                  {
                    "type": "command",
                    "command": "else if branch command",
                    "expression": "{{ref:data:value_1}} == 222",
                  }
                ],
                "else_flow": [
                  {
                    "type": "command",
                    "command": "else branch command"
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
