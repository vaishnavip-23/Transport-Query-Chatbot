### Singapore Transport Query Agent

This is an AI agent that answers your questions about Singapore's public transport system. Think of it as your personal transport guide that can tell you when the next bus is coming, whether the MRT is busy, or what's happening with traffic right now.

> **Note on Timezone:** All timestamps and time-based outputs use Singapore Time (SGT, UTC+8) to ensure consistency with data returned from the APIs. This allows direct comparison without timezone conversion of the data.

#### How it Works
The agent follows a simple approach:
1. Takes your question as **Input** (e.g., "When's the next bus at Orchard?")
2. **Figures out** what information it needs to answer the question
3. **Fetches live data** from Singapore's Land Transport Authority (LTA) APIs
4. **Gives you** a clear, easy-to-understand answer

#### What Data Does It Use?
I connected the agent to 7 different data sources(api's) from LTA DataMall so it can answer all sorts of questions:

1. **Bus Arrival Times** - Real-time info on when buses are coming to a stop
2. **Bus Stop Details** - Information about bus stops (codes, names, locations)
3. **Bus Routes** - Which services run on a route and when they operate
4. **Train Service Alerts** - Any delays, disruptions, or service updates on the MRT/LRT
5. **Station Crowd Levels** - How busy each MRT/LRT station is right now
6. **Traffic Incidents** - Real-time traffic problems like accidents or roadworks
7. **Traffic Speed Data** - Which roads are moving fast or slow at the moment

#### Why I Built It This Way

I designed the agent with simplicity in mind. Each part does one job well, making it easy to understand and fix if something goes wrong.

**Here's my design approach:**

- **Step-by-step workflow using LangGraph**: I used **LangGraph** (a framework for building multi-step AI workflows) with 5 nodes, each handling a specific responsibility:
  1. Add Context (figure out the time, peak hour, weekday/weekend)
  2. Parse Query (understand what the user wants and which APIs to call)
  3. Call APIs (fetch all the transport data we need)
  4. Format Data (clean and organize the raw data)
  5. Generate Response (write and proofread the final answer)

![Workflow Architecture](workflow_architecture.png)

LangGraph passes a shared state object through each node, so every stage has access to everything learned so far. This keeps the code organized and testable—cleaner than dumping all logic in one function.
- **Structured responses**: The AI returns organized data (like "bus number", "arrival time") instead of rambling text
- **Smart parsing**: Before asking for data, the agent extracts key details (bus stop names, service numbers) to avoid confusion
- **Caching for speed**: Bus stop information doesn't change often, so I store it in memory. This saves time and reduces API calls
- **Final polish**: The agent proofreads every answer before sending it to make sure it reads naturally

#### Error Handling

The agent gracefully handles API failures and missing data during off-peak hours, using cached data and fallback suggestions when live data is unavailable.

#### What I'm Assuming

To keep this project focused, I've made a few reasonable assumptions:

- **LTA's APIs are working**: I assume Singapore's transport data is *always available* and returns the expected information
- **Questions are in English**: Users ask their questions in natural English (not other languages or coded messages)
- **Questions make sense**: I'm assuming users ask reasonable questions (not trying to break the system on purpose)
- **Reasonable message size**: Questions stay within the AI model's token/context limits (no crazy long prompts)
- **Peak hours are fixed**: Peak hours are hardcoded as 7:00 AM–9:59 AM and 5:00 PM–7:59 PM on weekdays (Monday–Friday). Off-peak covers all other times and weekends. In production, these could be made configurable or pulled from a database.

- > **Real world finding** - LTA only publishes some data during peak hours (roughly 7-10 AM, 5-8 PM on weekdays). Off-peak and weekend data is sparse or unavailable.

#### How I'd Deploy This in the Real World

If I were launching this as a real product, here's what I'd do:

**Core Architecture**
- **Backend**: Build a backend service (using FastAPI) that exposes `/query` endpoints. This lets frontend apps, mobile clients, and other services make HTTP requests to the agent without needing to run the code directly.
- **Secure the secrets**: Never hardcode API keys in code. Store them safely in a secret manager so they won't be exposed to the public.
- **Caching**: Use **Redis** as a cache for bus stops (~6000 records) and station mappings. It is very fast.
- **Refresh Data**: Automatically refresh bus stop data every 6-12 hours (data rarely changes intra-day)
- **LLM Fallback**: If OpenAI is down or rate-limited, automatically fall back to **Google Gemini** or **Anthropic Claude**. Keeps the service running even if one provider has issues.
- **API Fallback**: If LTA APIs are slow/unavailable, return cached data with a timestamp warning ("This is data from 2 hours ago")
- **Rate Limiting**: Don't overwhelm LTA servers with multiple simultaneous requests. Queue requests and process them at a safe rate.
- **Logging**: Track every query (user ID, question, response time, which APIs called)
- **Metrics**: Monitor latency, error rates, LLM fallback usage, API success rates
