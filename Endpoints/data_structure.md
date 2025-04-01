---

## 🗂️ Agent Configuration Data Structure

### 📄 Example JSON Structure of an Agent

```json
{
  "agentName": "Agent name",
  "description": "Agent description",
  "context": [
    {
      "type": "persona",
      "content": [
        { "key": "key1", "value": "value1" }
      ]
    },
    {
      "type": "audience",
      "content": [
        { "key": "key2", "value": "value2" }
      ]
    }
  ],
  "workers": [
    {
      "name": "worker name",
      "description": "worker description",
      "entry": false,
      "SPL": {
        "workFlows": [
          {
            "type": "mainFlow",
            "description": "main flow description",
            "condition": {
              "description": "condition description",
              "value": "{{data_uuid}} == 222"
            },
            "flowContent": [
              {
                "type": "command",
                "content": "command content, {{data_source_uuid}}"
              },
              {
                "type": "ifElse",
                "content": {
                  "condition": {
                    "description": "condition description",
                    "value": "{{value 1}} == 222"
                  },
                  "trueFlow": [
                    { "type": "command", "content": "command content, {{data_source_uuid}}" }
                  ],
                  "falseFlow": [
                    { "type": "command", "content": "command content, {{path}}" }
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
                    { "type": "command", "content": "command content, {{data_source_uuid}}" }
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
  ]
}
```
