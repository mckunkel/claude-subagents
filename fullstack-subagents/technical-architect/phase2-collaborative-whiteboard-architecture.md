# Phase 2: Technical Architecture - Real-time Collaborative Whiteboard

## Architecture Overview

### System Pattern Selection
**Chosen Pattern**: Event-Driven Architecture with WebSocket-based Real-time Communication
- **Rationale**: Optimized for real-time collaboration with sub-200ms latency requirement
- **Complexity**: Medium-High
- **Team Size**: 6-8 developers
- **Scalability**: Designed for 10-100 concurrent users per workspace

### High-Level Architecture Diagram
```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Web Client    │────▶│   Load Balancer │────▶│   API Gateway   │
│  (Canvas App)   │     │   (Nginx/AWS)   │     │ (Authentication)│
└─────────────────┘     └─────────────────┘     └─────────────────┘
         │                                                │
         │ WebSocket                                      │ HTTP/REST
         ▼                                                ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ WebSocket Server│────▶│  Event Bus      │────▶│ Canvas Service  │
│  (Socket.io)    │     │   (Redis)       │     │  (Drawing API)  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ Session Store   │     │ Canvas Database │     │ User Service    │
│    (Redis)      │     │  (PostgreSQL)   │     │  (Auth/Teams)   │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Technology Stack Selection

### Frontend Technology Stack
```json
{
  "framework": "React 18 with Next.js 14",
  "canvas_library": "Fabric.js 5.3+ or Konva.js 9+",
  "real_time": "Socket.io-client 4.7+",
  "state_management": "Zustand with Immer",
  "styling": "Tailwind CSS + HeadlessUI",
  "build_tool": "Vite 5.0+",
  "typescript": "TypeScript 5.2+",
  "testing": "Vitest + React Testing Library"
}
```

**Rationale**:
- **React 18**: Concurrent rendering for smooth canvas operations
- **Fabric.js**: Mature canvas library with collaborative editing support
- **Socket.io**: Proven real-time communication with fallback mechanisms
- **Zustand**: Lightweight state management, perfect for real-time updates
- **Next.js**: SSR capabilities for SEO and initial load optimization

### Backend Technology Stack
```json
{
  "runtime": "Node.js 20 LTS",
  "framework": "Express.js 4.18+ with TypeScript",
  "websocket": "Socket.io 4.7+",
  "database": "PostgreSQL 15+ with Connection Pooling",
  "cache": "Redis 7+ (Session store + Event bus)",
  "message_queue": "Redis Streams for event processing",
  "authentication": "JWT with refresh tokens",
  "api_documentation": "OpenAPI 3.0 with Swagger UI",
  "orm": "Prisma 5.0+ for type-safe database access"
}
```

**Rationale**:
- **Node.js**: Excellent WebSocket performance and JavaScript ecosystem
- **Socket.io**: Handles connection management, reconnection, and clustering
- **PostgreSQL**: ACID compliance for version history, JSON support for canvas data
- **Redis**: Sub-millisecond operations for real-time session management
- **Prisma**: Type-safe database operations with excellent TypeScript integration

### Infrastructure & DevOps
```json
{
  "cloud_provider": "AWS",
  "compute": "ECS Fargate with Application Load Balancer",
  "database": "RDS PostgreSQL Multi-AZ",
  "cache": "ElastiCache Redis Cluster",
  "storage": "S3 for exports and static assets",
  "cdn": "CloudFront for global asset delivery",
  "monitoring": "CloudWatch + Prometheus + Grafana",
  "ci_cd": "GitHub Actions with AWS CodeDeploy"
}
```

## Detailed System Design

### Real-time Collaboration Architecture

#### WebSocket Communication Protocol
```typescript
// Event Types for Real-time Collaboration
interface CanvasEvent {
  type: 'draw' | 'shape' | 'text' | 'cursor' | 'selection';
  sessionId: string;
  userId: string;
  timestamp: number;
  data: DrawingData | CursorData | SelectionData;
}

interface DrawingData {
  tool: 'pen' | 'line' | 'rectangle' | 'circle' | 'text';
  coordinates: Point[];
  style: {
    strokeColor: string;
    strokeWidth: number;
    fillColor?: string;
  };
  objectId: string;
}

interface CursorData {
  x: number;
  y: number;
  userName: string;
  color: string;
}
```

#### Conflict Resolution Strategy
**Operational Transformation (OT) Implementation**:
1. **Local Operations**: Apply immediately to local canvas
2. **Remote Operations**: Transform against concurrent operations
3. **Server Coordination**: Server maintains operation ordering
4. **Conflict Resolution**: Last-write-wins with timestamp tiebreaker for simple operations

```typescript
class OperationalTransform {
  transformDrawOperation(localOp: DrawOperation, remoteOp: DrawOperation): DrawOperation {
    // Transform operations based on temporal ordering and object dependencies
    if (localOp.timestamp < remoteOp.timestamp) {
      return this.applyTransform(localOp, remoteOp);
    }
    return localOp;
  }
}
```

### Database Schema Design

#### Canvas Data Model
```sql
-- Canvas Sessions
CREATE TABLE canvas_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    workspace_id UUID NOT NULL,
    name VARCHAR(255) NOT NULL,
    created_by UUID NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    canvas_data JSONB, -- Fabric.js canvas state
    version INTEGER DEFAULT 1,
    is_active BOOLEAN DEFAULT true
);

-- Canvas Objects (for granular version control)
CREATE TABLE canvas_objects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID REFERENCES canvas_sessions(id),
    object_id VARCHAR(255) NOT NULL, -- Client-generated object ID
    object_type VARCHAR(50) NOT NULL, -- 'shape', 'text', 'drawing'
    object_data JSONB NOT NULL,
    created_by UUID NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    version INTEGER DEFAULT 1
);

-- Version History
CREATE TABLE canvas_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID REFERENCES canvas_sessions(id),
    version_number INTEGER NOT NULL,
    changes JSONB NOT NULL, -- Delta changes
    created_by UUID NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    description TEXT
);

-- User Sessions (for real-time presence)
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    canvas_session_id UUID REFERENCES canvas_sessions(id),
    user_id UUID NOT NULL,
    socket_id VARCHAR(255) NOT NULL,
    cursor_position JSONB,
    last_seen TIMESTAMP DEFAULT NOW(),
    is_active BOOLEAN DEFAULT true
);
```

#### Indexing Strategy
```sql
-- Performance optimization indexes
CREATE INDEX idx_canvas_sessions_workspace ON canvas_sessions(workspace_id, created_at DESC);
CREATE INDEX idx_canvas_objects_session ON canvas_objects(session_id, created_at);
CREATE INDEX idx_canvas_versions_session ON canvas_versions(session_id, version_number);
CREATE INDEX idx_user_sessions_canvas ON user_sessions(canvas_session_id, is_active);
CREATE INDEX idx_canvas_data_gin ON canvas_sessions USING GIN(canvas_data);
```

### API Design Specification

#### REST API Endpoints
```yaml
openapi: 3.0.0
info:
  title: Collaborative Whiteboard API
  version: 1.0.0

paths:
  /api/v1/canvas/{sessionId}:
    get:
      summary: Get canvas session data
      responses:
        200:
          content:
            application/json:
              schema:
                type: object
                properties:
                  sessionId: string
                  canvasData: object
                  version: integer
                  participants: array
    
    put:
      summary: Update canvas session
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                canvasData: object
                version: integer
  
  /api/v1/canvas/{sessionId}/export:
    post:
      summary: Export canvas to various formats
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                format: 
                  type: string
                  enum: [png, svg, pdf]
                options: object
```

#### WebSocket Event API
```typescript
// Client to Server Events
interface ClientEvents {
  'join-session': (sessionId: string) => void;
  'canvas-operation': (operation: CanvasOperation) => void;
  'cursor-move': (position: CursorPosition) => void;
  'user-selection': (selection: SelectionData) => void;
}

// Server to Client Events  
interface ServerEvents {
  'session-joined': (sessionData: SessionData) => void;
  'operation-applied': (operation: CanvasOperation) => void;
  'user-joined': (user: UserInfo) => void;
  'user-left': (userId: string) => void;
  'cursor-updated': (userId: string, position: CursorPosition) => void;
  'error': (error: ErrorInfo) => void;
}
```

### Performance Optimization Strategy

#### Caching Architecture
```typescript
// Multi-level caching strategy
interface CacheStrategy {
  // Level 1: Redis (Session data, hot canvas objects)
  sessionCache: {
    ttl: 3600, // 1 hour
    keyPattern: 'session:{sessionId}',
    data: 'canvas_state + active_users'
  };
  
  // Level 2: CDN (Static exports, images)
  cdnCache: {
    ttl: 86400, // 24 hours
    patterns: ['*.png', '*.svg', '*.pdf'],
    regions: ['us-east-1', 'eu-west-1', 'ap-southeast-1']
  };
  
  // Level 3: Browser (Canvas state, user preferences)
  browserCache: {
    mechanism: 'IndexedDB',
    storage: 'offline_canvas_data + user_settings'
  };
}
```

#### WebSocket Scaling Strategy
```typescript
// Socket.io Cluster Configuration
const io = new Server(server, {
  adapter: createAdapter({
    host: 'redis-cluster.cache.amazonaws.com',
    port: 6379
  }),
  transports: ['websocket', 'polling'],
  pingTimeout: 60000,
  pingInterval: 25000
});

// Horizontal scaling with sticky sessions
app.use(session({
  store: new RedisStore({
    client: redisClient
  }),
  cookie: {
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}));
```

## Non-Functional Requirements Implementation

### Performance Requirements
- **Latency Target**: Sub-200ms for canvas operations
- **Throughput**: 1000 concurrent operations/second per workspace
- **Implementation**: 
  - WebSocket connection pooling
  - Operation batching for bulk updates
  - Differential synchronization (send only changes)
  - Client-side prediction with server reconciliation

### Scalability Strategy
```typescript
// Auto-scaling configuration
const scaling = {
  horizontal: {
    metric: 'concurrent_connections',
    thresholds: {
      scaleUp: 80, // connections per instance
      scaleDown: 20
    },
    instances: {
      min: 2,
      max: 10
    }
  },
  database: {
    readReplicas: 2,
    connectionPooling: {
      max: 100,
      idle: 10
    }
  }
};
```

### Security Implementation
```typescript
// Security architecture
const security = {
  authentication: {
    jwt: {
      accessToken: { ttl: '15m' },
      refreshToken: { ttl: '7d' }
    },
    websocket: 'JWT in connection handshake'
  },
  authorization: {
    model: 'RBAC (Role-Based Access Control)',
    roles: ['owner', 'editor', 'viewer'],
    permissions: {
      owner: ['read', 'write', 'delete', 'invite'],
      editor: ['read', 'write'],
      viewer: ['read']
    }
  },
  dataProtection: {
    encryption: {
      transit: 'TLS 1.3',
      rest: 'AES-256'
    },
    validation: 'JSON Schema validation on all inputs'
  }
};
```

## Architecture Decision Records

### ADR-001: Canvas Library Selection
**Status**: Accepted  
**Date**: 2024-08-04  

**Context**: Need to choose between Fabric.js, Konva.js, and custom Canvas API implementation for collaborative drawing.

**Decision**: Use Fabric.js 5.3+ as the primary canvas library.

**Rationale**:
- Excellent collaborative editing support with object-based architecture
- Built-in serialization/deserialization for canvas state
- Strong community and extensive documentation
- Good performance with large numbers of objects
- Event system that integrates well with real-time updates

**Consequences**:
- **Positive**: Faster development, proven collaboration features, rich object model
- **Negative**: Larger bundle size (~200KB), learning curve for team
- **Mitigation**: Code splitting, team training, performance monitoring

### ADR-002: Real-time Communication Protocol
**Status**: Accepted  
**Date**: 2024-08-04  

**Context**: Need to choose between WebSocket implementations for real-time collaboration.

**Decision**: Use Socket.io 4.7+ for WebSocket communication.

**Rationale**:
- Automatic fallback to HTTP polling for connectivity issues
- Built-in clustering support for horizontal scaling
- Room-based architecture perfect for canvas sessions
- Mature ecosystem with extensive documentation
- Excellent TypeScript support

**Consequences**:
- **Positive**: Reliable connections, easy scaling, proven performance
- **Negative**: Slightly higher overhead than raw WebSockets
- **Mitigation**: Performance testing, connection optimization

### ADR-003: State Management Architecture
**Status**: Accepted  
**Date**: 2024-08-04  

**Context**: Need to handle complex state synchronization between local canvas state and remote updates.

**Decision**: Implement Operational Transformation with client-side prediction.

**Rationale**:
- Provides smooth user experience with immediate feedback
- Handles concurrent editing conflicts intelligently
- Proven pattern for collaborative editing applications
- Maintains consistency across all clients

**Consequences**:
- **Positive**: Excellent user experience, conflict resolution, consistency
- **Negative**: Complex implementation, debugging challenges
- **Mitigation**: Comprehensive testing, operation logging, rollback mechanisms

## Risk Assessment & Mitigation

### High Priority Risks
1. **WebSocket Connection Stability**
   - **Impact**: High (core functionality)
   - **Probability**: Medium
   - **Mitigation**: Connection retry logic, fallback to HTTP polling, health monitoring

2. **Canvas Performance with Large Sessions**
   - **Impact**: High (user experience)
   - **Probability**: Medium  
   - **Mitigation**: Virtualization for large canvases, object culling, progressive loading

3. **Real-time Synchronization Conflicts**
   - **Impact**: Medium (data consistency)
   - **Probability**: High
   - **Mitigation**: Robust operational transformation, conflict logging, manual resolution UI

### Medium Priority Risks
1. **Database Scaling Bottlenecks**
   - **Impact**: Medium (affects user growth)
   - **Probability**: Low
   - **Mitigation**: Read replicas, query optimization, caching layers

2. **Third-party Integration Failures**
   - **Impact**: Medium (feature availability)
   - **Probability**: Medium
   - **Mitigation**: Graceful degradation, circuit breaker pattern, retry mechanisms

## Handoff to Development Teams

### Frontend Development Requirements
1. **Canvas Implementation**: Fabric.js integration with real-time synchronization
2. **WebSocket Client**: Socket.io client with reconnection and error handling
3. **State Management**: Zustand stores for canvas state and user sessions
4. **UI Components**: Drawing tools, user presence indicators, export controls
5. **Performance**: Optimized rendering for 10-100 concurrent users

### Backend Development Requirements  
1. **WebSocket Server**: Socket.io server with clustering support
2. **Canvas API**: RESTful API for canvas CRUD operations
3. **Real-time Sync**: Operational transformation implementation
4. **Database Layer**: Prisma ORM with optimized queries
5. **Export Service**: Canvas to PNG/SVG/PDF conversion

### Success Criteria for Next Phases
- **Frontend**: Working canvas with real-time collaboration and sub-200ms latency
- **Backend**: Scalable WebSocket infrastructure supporting 100 concurrent users
- **Integration**: Seamless data flow between frontend and backend systems
- **Performance**: All operations meet the sub-200ms latency requirement

---

**Phase 2 Complete**: Technical architecture designed for real-time collaborative whiteboard. Ready for Frontend and Backend development implementation.