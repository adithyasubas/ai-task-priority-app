# ai-task-priority-app
# **Defining Logic**

- **Deadline** (tasks with sooner deadlines are higher priority)
- **Urgency** (marked as critical, important, or optional)
- **Effort Required** (short vs. long tasks)
- **Dependencies** (if Task B depends on Task A, prioritise A first)
- **Personal Preferences** (learning from past choices)

# **Choosing Tech Stack**

**Backend (Task Processing & AI Model)**

- **Python + FastAPI** → Fast and lightweight API
- **Scikit-learn / OpenAI API** → Simple ML model or GPT for intelligent ranking
- **PostgreSQL / DynamoDB** → Store tasks & priorities

**Frontend (User Input & Display)**

- **React (for web) or React Native (for mobile)**
- **Material-UI or Tailwind CSS** for a clean interface

**Cloud Deployment (Scalable & Reliable)**

- **AWS Lambda or EC2** → Run API backend
- **AWS SageMaker (optional)** → Train custom ML models
- **S3 + CloudFront** → Host frontend
- **DynamoDB / RDS** → Store user data
- **AWS Cognito** → User authentication

# **Develop AI Model**

Simple Rule-Based AI

using weights to assign the priority of the tasks

```python
**def calculate_priority(deadline, urgency, effort, dependencies):
	score = (10 / (deadline.days + 1)) + (urgency * 2) - (effort * 0.5) + (dependencies * 1.5)
	return score**
```

- **Urgency Levels**: Low (1), Medium (2), High (3)
- **Effort**: Estimated hours
- **Dependencies**: Count of other tasks blocking this one

# **Build API for Task Priority**

Using FastAPI, we can create an API endpoint to accept the tasks and return the weighted prioritisation

```python
from fastapi import FastAPI
from pydantic import BaseModel
from datetime import datetime, timedelta
import numpy as np

app = FastAPI()

class Task(BaseModel):
    name: str
    deadline: datetime
    urgency: int
    effort: int
    dependencies: int

def calculate_priority(task: Task):
    days_left = (task.deadline - datetime.now()).days
    score = (10 / (days_left + 1)) + (task.urgency * 2) - (task.effort * 0.5) + (task.dependencies * 1.5)
    return max(0, score)  # Ensure non-negative priority score

@app.post("/prioritize")
def prioritize_tasks(tasks: list[Task]):
    prioritized_tasks = sorted(tasks, key=calculate_priority, reverse=True)
    return {"prioritized_tasks": [task.name for task in prioritized_tasks]}
```

Deploying this API on AWS Lambda or EC2

# **Build Simple UI for Task Input and Display**

In React, fetch the prioritised tasks from the API and display them

```jsx
import { useState } from "react";

const App = () => {
  const [tasks, setTasks] = useState([]);
  const [priorityTasks, setPriorityTasks] = useState([]);

  const fetchPrioritizedTasks = async () => {
    const response = await fetch("/prioritize", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(tasks),
    });
    const data = await response.json();
    setPriorityTasks(data.prioritized_tasks);
  };

  return (
    <div>
      <h1>AI Task Prioritization</h1>
      <button onClick={fetchPrioritizedTasks}>Prioritize Tasks</button>
      <ul>
        {priorityTasks.map((task, index) => (
          <li key={index}>{task}</li>
        ))}
      </ul>
    </div>
  );
};

export default App;
```

Hosting this on AWS S3 and CloudFront for a simple web-based app

# **Deploy & Scale**

- **Backend:** AWS Lambda (serverless) or EC2
- **Frontend:** AWS S3 + CloudFront
- **Database:** DynamoDB (NoSQL) or RDS PostgreSQL
