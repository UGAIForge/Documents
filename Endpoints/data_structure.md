---

## 🗂️ Agent Configuration Data Structure

### 📄 Example JSON Structure of an Agent

```json
{
  "agentName": "Agent name",
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
    "workder_xxx": {
      "name": "worker name",
      "description": "worker description",
      "entry": false,
      "SPL":{
        "input": "this is input",
        "output": "output to this value {{ref:}}",
        "workFlows":[
          {
            "type": "mainFlow",
            "description": "main flow description",
            "condition": {
              "description": "condition description",
              "value": "{{ref:}} == 222"
            },
            "flowContent": [
              {
                "type": "command",
                "content": "command content, {{ref:}}"
              },
              {
                "type": "ifElse",
                "content": {
                  "condition": {
                    "description": "condition description",
                    "value": "{{ref:}} == 222"
                  },
                  "trueFlow": [
                    {
                      "type": "command",
                      "content": "command content, {{ref:}}"
                    }
                  ],
                  "falseFlow": [
                    {
                      "type": "command",
                      "content": "command content, {{path}}"
                    }
                  ]
                }
              },
              {
                "type": "loop",
                "content": {
                  "condition": {
                    "description": "condition description",
                    "value": "{{value 1}} == 222"
                  },
                  "trueFlow": [
                    {
                      "type": "command",
                      "content": "command content, {{data_source_uuid}}"
                    }
                  ],
                  "falseFlow": []
                }
              }
            ]
          }
        ]
      },
      "useCases": [
        {
          "name": "use case name",
          "description": "use case description",
          "code": "code"
        }
      ],
      "code": {
        "language": "language",
        "code": "code",
        "testCases": [
          {
            "name": "test case name",
            "description": "test case description",
            "code": "code"
          }
        ]
      },
      "diagrams": [
        {
          "name": "diagram name",
          "language": "language",
          "diagram": "diagram"
        }
      ]
    }
  }
}
```
