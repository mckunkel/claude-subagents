# Technical Architecture - Real-time Collaborative Whiteboard

## Executive Summary
This technical specification defines a scalable, real-time collaborative whiteboard architecture capable of supporting sub-200ms drawing latency and 10-100 concurrent users per workspace. The architecture emphasizes real-time synchronization, horizontal scalability, and robust data consistency.

## Architecture Overview

### Architecture Pattern
**Event-Driven Architecture with Real-time Synchronization**
- **Rationale**: Essential for real-time collaboration with operational transformation for conflict resolution
- **Complexity**: High
- **Team Size Recommendation**: 8-10 developers across frontend/backend/DevOps

### System Architecture Diagram
```
┌─────────────────┐    ┌──────────────┐    ┌─────────────────┐
│   Web Client    │◄──►│     CDN      │◄──►│  Load Balancer  │
│  (React SPA)    │    │ (CloudFront) │    │   (ALB/NLB)     │
└─────────────────┘    └──────────────┘    └─────────────────┘
                                                    │
                                           ┌─────────────────┐
                                           │  API Gateway    │
                                           │  (WebSocket +   │
                                           │   REST APIs)    │
                                           └─────────────────┘
                                                    │
                                    ┌───────────────┼───────────────┐
                                    │               │               │
                           ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
                           │   Auth      │ │ Workspace   │ │ Real-time   │
                           │  Service    │ │  Service    │ │ Sync Service│
                           └─────────────┘ └─────────────┘ └─────────────┘
                                    │               │               │
                                    └───────────────┼───────────────┘
                                                    │
                                         ┌─────────────────┐
                                         │   Database      │
                                         │  (PostgreSQL)   │
                                         │   + Redis       │
                                         └─────────────────┘
```

## Technology Stack Analysis

### Frontend Technology Stack
```json
{
  "framework": "React 18 with TypeScript",
  "build_tool": "Vite",
  "canvas_library": "Fabric.js + Custom WebGL acceleration",
  "real_time": "Socket.io-client",
  "state_management": "Zustand with immer",
  "styling": "Tailwind CSS + CSS Modules",
  "testing": "Vitest + React Testing Library + Playwright",
  "rationale": "React provides mature ecosystem, Fabric.js handles canvas complexity, Zustand offers simple state management for real-time updates"
}
```

### Backend Technology Stack
```json
{
  "runtime": "Node.js 20 LTS",
  "framework": "Express.js with TypeScript",
  "real_time": "Socket.io server with Redis adapter",
  "database": "PostgreSQL 15 with connection pooling",
  "cache": "Redis 7 for sessions and real-time state",
  "api_documentation": "OpenAPI 3.0 with Swagger UI",
  "testing": "Jest + Supertest",
  "rationale": "Node.js optimal for real-time applications, Socket.io provides robust WebSocket handling with fallbacks, PostgreSQL ensures data consistency"
}
```

### Infrastructure Stack
```json
{
  "cloud_provider": "AWS",
  "compute": "ECS Fargate with auto-scaling",
  "database": "RDS PostgreSQL with Multi-AZ",
  "cache": "ElastiCache Redis cluster",
  "cdn": "CloudFront with WebSocket support",
  "load_balancer": "Application Load Balancer (ALB)",
  "storage": "S3 for exported files",
  "monitoring": "CloudWatch + DataDog",
  "rationale": "AWS provides comprehensive real-time services, Fargate enables easy scaling, CloudFront supports WebSocket connections"
}
```

## System Design

### Core Services Architecture

#### 1. Real-time Sync Service
- **Responsibility**: WebSocket connections, operational transformation, conflict resolution
- **Technology**: Node.js + Socket.io + Redis Pub/Sub
- **Scaling**: Horizontal with sticky sessions via Redis adapter
- **Key Features**:
  - Operational Transformation (OT) for conflict-free collaboration
  - Connection pooling and management
  - Real-time cursor tracking
  - Drawing operation broadcasting

#### 2. Workspace Service
- **Responsibility**: Workspace CRUD, permissions, member management
- **Technology**: Node.js + Express + PostgreSQL
- **Scaling**: Horizontal with database connection pooling
- **Key Features**:
  - Workspace creation and management
  - Role-based access control (RBAC)
  - Member invitation and management
  - Version history tracking

#### 3. Auth Service
- **Responsibility**: User authentication, session management, JWT tokens
- **Technology**: Node.js + Express + Redis sessions
- **Scaling**: Stateless horizontal scaling
- **Key Features**:
  - JWT authentication with refresh tokens
  - Session management in Redis
  - OAuth integration (Google, GitHub)
  - Rate limiting and security

#### 4. Export Service
- **Responsibility**: Canvas export to PNG/SVG/PDF formats
- **Technology**: Node.js + Puppeteer/Canvas API
- **Scaling**: Background job processing with queues
- **Key Features**:
  - High-quality canvas rendering
  - Multiple format support
  - Asynchronous processing
  - S3 storage integration

### Real-time Data Synchronization

#### Operational Transformation Algorithm
```typescript
interface DrawingOperation {
  id: string;
  type: 'draw' | 'erase' | 'shape' | 'text';
  userId: string;
  timestamp: number;
  transform: Matrix;
  data: any;
  vectorClock: Record<string, number>;
}

class OperationalTransform {
  // Transform operation A against operation B for conflict resolution
  transform(opA: DrawingOperation, opB: DrawingOperation): DrawingOperation {
    // Implementation of OT algorithm for canvas operations
    // Handles concurrent drawing, moving, deleting operations
  }
  
  // Apply operation to canvas state
  apply(canvas: CanvasState, operation: DrawingOperation): CanvasState {
    // Update canvas state with operation
  }
}
```

#### WebSocket Event System
```typescript
// Client to Server Events
interface ClientEvents {
  'join-workspace': (workspaceId: string) => void;
  'drawing-operation': (operation: DrawingOperation) => void;
  'cursor-move': (position: { x: number, y: number }) => void;
  'user-selection': (elementIds: string[]) => void;
}

// Server to Client Events  
interface ServerEvents {
  'workspace-joined': (state: WorkspaceState) => void;
  'operation-broadcast': (operation: DrawingOperation) => void;
  'user-cursor': (userId: string, position: { x: number, y: number }) => void;
  'user-joined': (user: User) => void;
  'user-left': (userId: string) => void;
}
```

### Database Design

#### Core Tables Schema
```sql
-- Workspaces
CREATE TABLE workspaces (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    owner_id UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    settings JSONB DEFAULT '{}'
);

-- Workspace members with roles
CREATE TABLE workspace_members (
    workspace_id UUID REFERENCES workspaces(id),
    user_id UUID REFERENCES users(id),
    role VARCHAR(50) DEFAULT 'editor', -- admin, editor, viewer
    joined_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (workspace_id, user_id)
);

-- Canvas operations for persistence and history
CREATE TABLE canvas_operations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    workspace_id UUID NOT NULL REFERENCES workspaces(id),
    user_id UUID NOT NULL REFERENCES users(id),
    operation_type VARCHAR(50) NOT NULL,
    operation_data JSONB NOT NULL,
    vector_clock JSONB NOT NULL,
    timestamp TIMESTAMP DEFAULT NOW(),
    INDEX (workspace_id, timestamp)
);

-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    avatar_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT NOW(),
    last_active TIMESTAMP DEFAULT NOW()
);
```

#### Redis Data Structures
```typescript
// Active workspace sessions
interface WorkspaceSession {
  workspaceId: string;
  connectedUsers: Map<string, UserSession>;
  canvasState: CanvasSnapshot;
  operationQueue: DrawingOperation[];
}

// User session data
interface UserSession {
  userId: string;
  socketId: string;
  cursor: { x: number, y: number };
  activeTools: string[];
  lastActivity: number;
}

// Redis Keys:
// ws:{workspaceId}:sessions - Hash of connected users
// ws:{workspaceId}:state - Current canvas state snapshot
// ws:{workspaceId}:ops - List of recent operations (sliding window)
// user:{userId}:workspaces - Set of active workspace connections
```

## Performance Architecture

### Latency Optimization Strategy

#### 1. WebSocket Optimization
- **Connection Pooling**: Reuse WebSocket connections with keep-alive
- **Binary Protocol**: Use MessagePack for operation serialization
- **Delta Compression**: Send only changed data, not full states
- **Predictive Loading**: Pre-load likely next operations

#### 2. Canvas Rendering Optimization
```typescript
class OptimizedCanvas {
  private renderQueue: RenderOperation[] = [];
  private lastFrame = 0;
  
  // Batch rendering operations for 60fps
  scheduleRender(operation: RenderOperation) {
    this.renderQueue.push(operation);
    if (!this.animationFrame) {
      this.animationFrame = requestAnimationFrame(this.flushRenderQueue);
    }
  }
  
  // Use OffscreenCanvas for background processing
  async renderOffscreen(operations: DrawingOperation[]) {
    const offscreen = new OffscreenCanvas(width, height);
    const ctx = offscreen.getContext('2d');
    // Render operations without blocking main thread
  }
}
```

#### 3. Database Performance Tuning
```sql
-- Optimized indexes for common queries
CREATE INDEX CONCURRENTLY idx_canvas_ops_workspace_time 
ON canvas_operations (workspace_id, timestamp DESC);

CREATE INDEX CONCURRENTLY idx_workspace_members_lookup
ON workspace_members (user_id, workspace_id);

-- Partitioning for canvas operations by time
CREATE TABLE canvas_operations_2024 PARTITION OF canvas_operations
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

### Scalability Architecture

#### Horizontal Scaling Strategy
- **Stateless Services**: All services designed for horizontal scaling
- **Database Read Replicas**: Read-heavy operations use replica databases
- **Redis Clustering**: Sharded Redis for session and state management
- **CDN Integration**: Global content delivery with WebSocket support

#### Auto-scaling Configuration
```yaml
# ECS Auto Scaling Policy
services:
  real-time-sync:
    min_capacity: 2
    max_capacity: 50
    target_cpu: 70%
    target_connections: 1000
    scale_out_cooldown: 60s
    scale_in_cooldown: 300s
    
  workspace-service:
    min_capacity: 2
    max_capacity: 20
    target_cpu: 60%
    target_response_time: 200ms
```

## Security Architecture

### Authentication & Authorization
```typescript
interface SecurityConfig {
  authentication: {
    jwt_secret: string;
    token_expiry: '15m';
    refresh_token_expiry: '7d';
    algorithm: 'HS256';
  };
  
  authorization: {
    roles: ['admin', 'editor', 'viewer'];
    permissions: {
      admin: ['read', 'write', 'delete', 'manage_users'];
      editor: ['read', 'write'];
      viewer: ['read'];
    };
  };
  
  rate_limiting: {
    drawing_operations: '100/minute';
    api_requests: '1000/hour';
    websocket_connections: '10/minute';
  };
}
```

### Data Protection
- **Encryption in Transit**: TLS 1.3 for all communications
- **Encryption at Rest**: AES-256 for database and file storage
- **WebSocket Security**: Origin validation, CSRF protection
- **API Security**: Input validation, SQL injection prevention

### Security Monitoring
```typescript
interface SecurityEvents {
  'suspicious_activity': {
    userId: string;
    activity: string;
    severity: 'low' | 'medium' | 'high';
    metadata: Record<string, any>;
  };
  
  'rate_limit_exceeded': {
    ip: string;
    endpoint: string;
    limit: number;
    current: number;
  };
}
```

## Architecture Decision Records

### ADR-001: Real-time Communication Protocol
**Status**: Accepted  
**Date**: 2024-08-04

**Context**: Need to choose between WebSocket, WebRTC, and Server-Sent Events for real-time collaboration.

**Decision**: Use WebSocket with Socket.io for real-time communication.

**Rationale**:
- Sub-200ms latency requirement achievable with WebSocket
- Socket.io provides robust fallback mechanisms
- Mature ecosystem and debugging tools
- Built-in room/namespace support for workspace isolation

**Consequences**:
- **Positive**: Reliable real-time communication, excellent browser support
- **Negative**: Server-side connection scaling complexity
- **Mitigation**: Redis adapter for horizontal scaling, connection pooling

### ADR-002: Canvas Rendering Technology
**Status**: Accepted  
**Date**: 2024-08-04

**Context**: Choose between HTML5 Canvas, SVG, and WebGL for drawing performance.

**Decision**: HTML5 Canvas with Fabric.js and WebGL acceleration for complex operations.

**Rationale**:
- Canvas API optimal for real-time drawing operations
- Fabric.js provides object management and interaction
- WebGL acceleration for complex transformations
- Good performance at 60fps with proper optimization

**Consequences**:
- **Positive**: Excellent drawing performance, rich interaction model
- **Negative**: Complexity in handling canvas state synchronization
- **Mitigation**: Operational transformation for conflict resolution

### ADR-003: Database Choice for Real-time Data
**Status**: Accepted  
**Date**: 2024-08-04

**Context**: Choose between PostgreSQL, MongoDB, and specialized time-series databases.

**Decision**: PostgreSQL for primary data with Redis for real-time state.

**Rationale**:
- ACID compliance essential for workspace consistency
- JSONB support for flexible operation storage
- Excellent performance with proper indexing
- Redis perfect for real-time session management

**Consequences**:
- **Positive**: Strong consistency, flexible queries, mature ecosystem
- **Negative**: Requires careful query optimization
- **Mitigation**: Connection pooling, read replicas, query optimization

## Non-Functional Requirements

### Performance Targets
```json
{
  "latency": {
    "drawing_operations": "< 200ms 95th percentile",
    "api_responses": "< 100ms average",
    "websocket_connection": "< 50ms establishment"
  },
  "throughput": {
    "concurrent_users_per_workspace": 100,
    "total_system_users": 10000,
    "operations_per_second": 5000
  },
  "scalability": {
    "horizontal_scaling": "Auto-scaling based on CPU/memory/connections",
    "database_scaling": "Read replicas + connection pooling",
    "storage_scaling": "S3 with CloudFront CDN"
  }
}
```

### Reliability Requirements
```json
{
  "availability": {
    "uptime_target": "99.9%",
    "planned_downtime": "< 4 hours/month",
    "recovery_time_objective": "< 15 minutes"
  },
  "data_integrity": {
    "backup_frequency": "Continuous with point-in-time recovery",
    "backup_retention": "30 days",
    "disaster_recovery": "Multi-AZ deployment"
  },
  "monitoring": {
    "application_metrics": "Response times, error rates, user activity",
    "infrastructure_metrics": "CPU, memory, network, disk I/O",
    "business_metrics": "Active users, workspace usage, export activity"
  }
}
```

## Implementation Guidelines

### Development Workflow
```yaml
development_process:
  branching_strategy: "GitFlow with feature branches"
  code_review: "Required for all changes, 2 approvers minimum"
  testing_requirements:
    unit_coverage: ">= 90%"
    integration_coverage: ">= 80%"
    e2e_coverage: "Critical user paths"
  
  ci_cd_pipeline:
    stages: ["lint", "test", "build", "security_scan", "deploy"]
    deployment_environments: ["dev", "staging", "production"]
    rollback_strategy: "Blue-green deployment with automatic rollback"
```

### Monitoring and Observability
```typescript
interface MonitoringStack {
  application_monitoring: 'DataDog APM';
  infrastructure_monitoring: 'CloudWatch + DataDog';
  log_aggregation: 'DataDog Logs';
  error_tracking: 'Sentry';
  uptime_monitoring: 'Pingdom';
  performance_monitoring: 'DataDog RUM';
}

// Key metrics to track
interface KPIs {
  technical: {
    drawing_latency: number;        // < 200ms target
    websocket_connection_time: number; // < 50ms target
    error_rate: number;            // < 0.1% target
    system_availability: number;   // 99.9% target
  };
  
  business: {
    daily_active_users: number;
    workspace_creation_rate: number;
    collaboration_engagement: number;
    export_usage_frequency: number;
  };
}
```

## Risk Assessment and Mitigation

### Technical Risks
```typescript
interface RiskAssessment {
  high_priority: [
    {
      risk: "WebSocket connection scaling bottlenecks",
      impact: "High - affects core collaboration feature",
      probability: "Medium",
      mitigation: "Redis clustering, connection pooling, horizontal scaling"
    },
    {
      risk: "Canvas synchronization conflicts",
      impact: "High - data consistency issues",
      probability: "Medium", 
      mitigation: "Operational transformation algorithm, conflict resolution UI"
    }
  ];
  
  medium_priority: [
    {
      risk: "Database performance degradation",
      impact: "Medium - slower response times",
      probability: "Low",
      mitigation: "Query optimization, connection pooling, read replicas"
    },
    {
      risk: "Third-party dependency failures",
      impact: "Medium - feature unavailability",
      probability: "Low",
      mitigation: "Circuit breaker pattern, graceful degradation"
    }
  ];
}
```

## Handoff to Development Teams

### Frontend Development Requirements
- Canvas-based drawing interface with Fabric.js
- Real-time collaboration UI with live cursors
- WebSocket integration for operation synchronization
- Responsive design for tablet/mobile support
- Export functionality integration

### Backend Development Requirements  
- WebSocket server with operational transformation
- RESTful APIs for workspace management
- Authentication and authorization system
- Real-time data synchronization logic
- Export service implementation

### Infrastructure Requirements
- AWS ECS deployment with auto-scaling
- PostgreSQL RDS with Multi-AZ setup
- Redis ElastiCache cluster configuration
- CloudFront CDN with WebSocket support
- Monitoring and logging infrastructure

This technical architecture provides a comprehensive foundation for building a scalable, performant real-time collaborative whiteboard system that meets all specified requirements and constraints.