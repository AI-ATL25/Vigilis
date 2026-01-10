# Vigilis

**AI-Powered Emergency Dispatch and Incident Management System**

Vigilis is a comprehensive emergency services platform that combines real-time incident tracking, AI-powered dispatch assistance, and intelligent decision support to help 911 dispatchers and emergency personnel respond more effectively to critical situations.

## Overview

Vigilis integrates multiple systems to provide a unified emergency management solution:

- **Real-time Incident Tracking**: MongoDB-based incident database with live updates
- **AI Assistant (Vigilis Agent)**: Google Gemini-powered conversational AI that answers dispatcher queries
- **Police Car Management**: Track and dispatch police vehicles with real-time location monitoring
- **Intelligent Suggestions**: Vector-based semantic search retrieves insights from similar past incidents
- **Live Location Streaming**: Redis-backed real-time GPS tracking with WebSocket support
- **Fill Agent**: AI-powered automatic field detection and incident data enrichment

## Architecture

### Backend (`/backend`)
- **Framework**: FastAPI (Python 3.11)
- **Database**: MongoDB Atlas (primary storage)
- **Cache/Streaming**: Redis (real-time location data)
- **AI Models**: Google Gemini (via google.adk and google-genai)

### Frontend (`/frontend/vigilis`)
- **Framework**: Next.js 16 with React 19
- **Styling**: Tailwind CSS 4
- **Maps**: Mapbox GL JS with react-map-gl
- **State Management**: TanStack Query (React Query)

## Key Features

### 1. Incident Management
- **Automatic Incident Creation**: New incidents auto-created from 911 call transcripts
- **Multi-conversation Support**: Track multiple communication streams (911 calls, radio, etc.)
- **AI Summarization**: Automatic incident status summaries
- **Dynamic Field Updates**: AI agent monitors transcripts and updates location/severity fields

### 2. AI-Powered Dispatch Assistant
The **Vigilis Agent** provides intelligent assistance to dispatchers:
- Natural language queries about incident details
- Real-time access to incident data via function calling
- Contextual responses based on incident history
- Integration with the incident database

### 3. Historical Insights
- **Vector Embeddings**: Incidents embedded using Gemini embedding model (768 dimensions)
- **Semantic Search**: Finds similar past incidents based on content similarity
- **Data-Driven Suggestions**: Provides actionable recommendations based on historical outcomes
- **Pattern Recognition**: Identifies successful strategies from past incidents

### 4. Police Car Dispatch System
- **Fleet Management**: Track all police units with officer details
- **Status Tracking**: Multiple states (inactive, dispatched, en_route, on_scene, returning)
- **Dispatch History**: Complete audit trail of unit deployments
- **Real-time Updates**: Live location streaming via Redis and WebSocket

### 5. Real-Time Location Services
- **High-Frequency Updates**: Redis stores GPS data updated every second
- **Car Simulator**: Built-in movement simulation for testing
- **Geospatial Queries**: Find nearby units within a radius
- **WebSocket Streaming**: Subscribe to individual car location updates
- **Sync Service**: Periodic synchronization from Redis to MongoDB

## API Endpoints

### Incident Endpoints
```
GET  /incidents              - Get all active incidents
POST /incident/update_transcript - Add transcript to incident
GET  /incident/chat_elements/{id} - Get chat history
POST /incident/context       - Get full incident document
POST /incident/summary       - Get AI-generated summary
POST /incident/suggestions   - Get AI recommendations
POST /incident/report        - Generate comprehensive report
POST /incident/post_story    - Conclude incident and save to knowledge base
PUT  /incident/status        - Update incident status
```

### Police Car Endpoints
```
POST   /police/cars         - Create new police car
GET    /police/cars         - Get all police cars (filter by status)
GET    /police/cars/{id}    - Get specific car
POST   /police/dispatch     - Dispatch car to incident
POST   /police/conclude     - Conclude dispatch
PUT    /police/status       - Update car status
PUT    /police/location     - Update car location
GET    /police/available    - Get available cars
GET    /police/incident/{id} - Get cars for incident
DELETE /police/cars/{id}    - Delete police car
```

### Real-Time Location Endpoints
```
GET  /police/realtime/{car_id} - Get real-time location from Redis
GET  /police/realtime           - Get all real-time locations
POST /police/nearby             - Find nearby cars within radius
WS   /ws/track/{car_id}         - WebSocket stream for car location
```

### AI Assistant Endpoint
```
POST /chat - Chat with Vigilis AI assistant
```

## Setup & Installation

### Prerequisites
- Python 3.11+
- Node.js 18+
- MongoDB Atlas account
- Redis instance (for real-time features)
- Google Gemini API key

### Backend Setup

1. **Clone the repository**
```bash
git clone <repository-url>
cd Vigilis
```

2. **Create virtual environment**
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. **Install dependencies**
```bash
pip install -r requirements.txt
```

4. **Configure environment variables**
Create a `.env` file in the root directory:
```env
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/
GEMINI_API_KEY=your_gemini_api_key_here
GOOGLE_GENAI_USE_VERTEXAI=0
REDIS_HOST=localhost
REDIS_PORT=6379
WEBSOCKET_SECRET=vigilis_secret_2024
```

5. **Run the backend**
```bash
cd backend
uvicorn api:app --host 0.0.0.0 --port 8000 --reload
```

### Frontend Setup

1. **Navigate to frontend directory**
```bash
cd frontend/vigilis
```

2. **Install dependencies**
```bash
npm install
```

3. **Configure environment**
Create `.env.local`:
```env
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_MAPBOX_TOKEN=your_mapbox_token_here
```

4. **Run development server**
```bash
npm run dev
```

The frontend will be available at `http://localhost:3000`

### MongoDB Setup

1. **Create vector search index** for incident knowledge base:
```javascript
{
  "fields": [
    {
      "type": "vector",
      "path": "final_summary_embedding",
      "numDimensions": 768,
      "similarity": "cosine"
    }
  ]
}
```

2. **Collections required**:
   - `active_incidents` - Current incidents
   - `incident_knowledge_base` - Historical concluded incidents
   - `police_cars` - Police fleet data

### Optional: MongoDB Change Stream Watcher

For local development without MongoDB triggers:
```bash
python run_local_watcher.py
```

This watches for database changes and triggers the fill agent automatically.

## Deployment

### Deploy to Render

See [RENDER_DEPLOY.md](./RENDER_DEPLOY.md) for detailed deployment instructions.

**Quick steps**:
1. Push to GitHub
2. Connect repository to Render
3. Set environment variables
4. Deploy with: `uvicorn backend.api:app --host 0.0.0.0 --port $PORT`

### Production Considerations
- Configure CORS origins in `backend/api.py`
- Set up MongoDB replica set for change streams
- Use managed Redis service (e.g., Redis Cloud, AWS ElastiCache)
- Enable MongoDB Atlas network access for production IPs
- Use secure WebSocket secrets

## Key Components

### Fill Agent (`backend/fill_agent/`)
Automatically analyzes incident transcripts and updates dynamic fields:
- Detects location changes from conversation
- Updates severity levels based on new information
- Triggers on transcript updates

### Polizia Agent (`backend/polizia_agent/`)
The main conversational AI assistant:
- Uses Google ADK's `LlmAgent`
- Function calling for database queries
- Context-aware responses

### Redis Tracking System (`backend/redis_tracking/`)
Real-time location management:
- `car_simulator.py` - Simulates vehicle movement
- `location_sync.py` - Syncs Redis to MongoDB
- `redis_client.py` - Redis connection management

### Suggestion System (`backend/suggest.py`)
Historical incident analysis:
- Embeds current incident summary
- Performs vector search in knowledge base
- Generates actionable suggestions from similar cases

## Data Models

### Incident Document
```python
{
    "incident_id": str,
    "title": str,
    "severity": str,
    "status": "active" | "concluded",
    "created_at": ISO8601,
    "location": {
        "address_text": str,
        "geojson": {
            "type": "Point",
            "coordinates": [lng, lat]
        }
    },
    "transcripts": {
        "conversation_id": [str]
    },
    "current_summary": str,
    "chat_elements": [dict],
    "last_summary_update_at": ISO8601
}
```

### Police Car Document
```python
{
    "car_id": str,
    "car_model": str,
    "unit_number": str,
    "officer": {
        "name": str,
        "badge": str,
        "rank": str
    },
    "status": "inactive" | "dispatched" | "en_route" | "on_scene" | "returning",
    "incident_id": str | None,
    "location": {
        "lat": float,
        "lng": float,
        "address": str
    },
    "dispatch_history": [dict]
}
```

## Technology Stack

### Backend
- **FastAPI** - Modern async web framework
- **PyMongo** - MongoDB driver
- **Redis-py** - Redis client
- **Google Generative AI** - Gemini API client
- **Google ADK** - Agent Development Kit
- **Uvicorn** - ASGI server
- **Pydantic** - Data validation
- **WebSockets** - Real-time communication

### Frontend
- **Next.js** - React framework with SSR
- **TypeScript** - Type-safe JavaScript
- **Mapbox GL JS** - Interactive maps
- **Framer Motion** - Animation library
- **TanStack Query** - Server state management
- **Tailwind CSS** - Utility-first CSS

### AI & ML
- **Google Gemini** - Large language model
- **Gemini Embeddings** - Text vectorization (768-dim)
- **MongoDB Vector Search** - Semantic similarity search

## Development Tools

### Database Seeding
```bash
python seed_db.py
```

### Testing Redis System
```bash
cd backend/redis_tracking
python test_e2e.py
python test_redis_system.py
python test_websocket.py
```

### Debug WebSocket
```bash
cd backend/redis_tracking
python debug_websocket.py
```

## Contributing

This project uses:
- **Python 3.11** with type hints
- **Black** for Python code formatting
- **ESLint** for frontend linting
- **Git** for version control

## License

ISC

## Support

For issues or questions, please refer to the API documentation at the root endpoint (`/`) when the server is running.

---

**Built with ❤️ for emergency services personnel**
