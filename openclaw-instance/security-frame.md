graph TD
    A[AI Coding Agent] --> B[Workspace (Branch)]
    B --> C[Static Analysis / Virus Scan]
    C --> D{Risk Score}
    D -->|Low Risk| E[Auto Merge to Main Project]
    D -->|High Risk| F[Human Review]
    F --> G[Merge / Reject]
    G --> H[Main Project (User_1's Project)]
    E --> H