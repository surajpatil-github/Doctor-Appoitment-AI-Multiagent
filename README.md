# 🩺 Doctor Appointment AI Multiagent

An **AI-powered, multi-agent doctor appointment system** built using **LangGraph, LangChain, FastAPI, and Streamlit**.  
The app helps users:

- Check doctor availability by **doctor name** or **specialization**
- **Book, cancel, or reschedule** appointments
- Use an ID number to keep track of bookings

Everything is driven by a **Supervisor Agent + ReAct sub-agents**, orchestrated as a graph using LangGraph.

---

## 🚀 Key Features

- 🧠 **Supervisor Agent** that understands the user’s query and routes it to the right worker agent  
- 🤝 **Information Agent (ReAct)** for checking doctor availability and answering FAQs  
- 📅 **Booking Agent (ReAct)** for setting, cancelling, and rescheduling appointments  
- 🧾 CSV-based storage (`doctor_availability.csv`) for schedules and availability  
- 🌐 **FastAPI backend** exposing `/execute` endpoint  
- 💻 **Streamlit frontend** for a simple Doctor Appointment UI  
- 🔐 Uses **OpenAI GPT-4o** (via `langchain_openai`) with API key loaded from `.env`

---

## 🧩 Architecture – Supervisor + ReAct Agents

This project follows a **multi-agent architecture**:

### 1. Supervisor Agent (Router)

- Implemented in `agent.py` as `DoctorAppointmentAgent.supervisor_node`
- Uses the `Router` Pydantic schema with fields:
  - `next`: `"information_node"`, `"booking_node"`, or `"FINISH"`
  - `reasoning`: explanation of why it routed there
- The supervisor:
  - Reads the conversation + user ID
  - Decides **whether the user wants information or an actual booking action**
  - Routes control to:
    - `information_node` (info agent)
    - `booking_node` (booking agent)
    - or `FINISH` to end the conversation

The routing logic is powered by the LLM with **structured output**, so the model must always return one of the allowed next steps.

### 2. ReAct Agents (Information & Booking)

Both worker nodes use **LangGraph’s `create_react_agent`**, which follows the **ReAct (Reasoning + Acting)** pattern:

> The agent thinks (reasoning), calls tools (acting), sees tool results, and then produces a final answer.

#### 🛈 Information Agent (`information_node`)

- System prompt:  
  “You are a specialized agent to provide information related to availability of doctors or any FAQs related to the hospital…”
- Built using:

  ```python
  information_agent = create_react_agent(
      model=self.llm_model,
      tools=[check_availability_by_doctor, check_availability_by_specialization],
      prompt=system_prompt,
  )
