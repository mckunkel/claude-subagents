# Full Stack SubAgents - Complete Development Example

## Project Overview
This example demonstrates how all 8 Full Stack SubAgents work together to build a complete e-commerce application from initial concept to production deployment.

**Project**: "TechShop" - Modern e-commerce platform for electronic gadgets
**Timeline**: Complete development lifecycle in coordinated phases
**Technology Stack**: Determined by team collaboration

## Phase 1: Project Initiation and Planning

### Input Requirements
```
Project Brief: "We need a modern e-commerce platform for selling electronic gadgets. 
Features needed: product catalog, shopping cart, user authentication, payment processing, 
order management, inventory tracking, and admin dashboard. Target launch: 3 months."

Additional Context:
- Target audience: Tech enthusiasts aged 25-45
- Expected traffic: 10,000 users/month initially
- Budget: $50,000 development budget
- Compliance: PCI DSS for payments, GDPR for EU users
```

### 1. Product Manager Agent Workflow

**Agent Invocation:**
```bash
Use the product-manager subagent to analyze the project requirements and create a comprehensive development roadmap
```

**Agent Tasks:**
- Analyze stakeholder requirements and business objectives
- Create user personas and user journey maps
- Define feature prioritization and MVP scope
- Establish success metrics and KPIs
- Generate project timeline and milestone plan

**Expected Output:**
```json
{
  "project_scope": {
    "mvp_features": [
      "User registration/authentication",
      "Product catalog with search/filter",
      "Shopping cart and wishlist",
      "Stripe payment integration",
      "Order tracking system",
      "Basic admin panel"
    ],
    "future_features": [
      "Advanced analytics dashboard",
      "Inventory management system",
      "Multi-vendor support",
      "Mobile app",
      "AI-powered recommendations"
    ]
  },
  "user_personas": {
    "primary": {
      "name": "Tech Enthusiast Tom",
      "age": 32,
      "needs": "Latest gadgets, detailed specs, reviews, fast checkout"
    },
    "secondary": {
      "name": "Budget-Conscious Betty",
      "age": 28,
      "needs": "Price comparison, deals, warranties, secure payment"
    }
  },
  "timeline": {
    "phase_1": "Requirements & Design (2 weeks)",
    "phase_2": "Core Development (6 weeks)",
    "phase_3": "Testing & Deployment (2 weeks)",
    "phase_4": "Launch & Optimization (2 weeks)"
  },
  "success_metrics": {
    "conversion_rate": "> 2.5%",
    "page_load_time": "< 3 seconds",
    "user_satisfaction": "> 4.2/5",
    "uptime": "> 99.5%"
  }
}
```

### 2. Technical Architect Agent Workflow

**Agent Invocation:**
```bash
Use the technical-architect subagent to design the system architecture based on the product requirements
```

**Agent Tasks:**
- Analyze functional and non-functional requirements
- Design microservices architecture
- Select optimal technology stack
- Define data models and API contracts
- Create security and scalability plan

**Expected Output:**
```json
{
  "architecture": {
    "pattern": "Microservices with API Gateway",
    "frontend": "React 18 with Next.js 14",
    "backend": "Node.js with Express and TypeScript",
    "database": "PostgreSQL with Redis caching",
    "search": "Elasticsearch for product search",
    "payment": "Stripe API integration",
    "storage": "AWS S3 for product images",
    "hosting": "AWS ECS with Load Balancer"
  },
  "services": {
    "api_gateway": "NGINX with rate limiting",
    "user_service": "Authentication and profiles",
    "product_service": "Catalog and inventory",
    "order_service": "Cart and order processing",
    "payment_service": "Stripe integration",
    "notification_service": "Email and SMS alerts",
    "admin_service": "Management dashboard"
  },
  "data_models": {
    "users": "PostgreSQL with bcrypt hashing",
    "products": "PostgreSQL with full-text search",
    "orders": "PostgreSQL with ACID compliance",
    "cache": "Redis for session and product data"
  },
  "security": {
    "authentication": "JWT with refresh tokens",
    "authorization": "Role-based access control",
    "data_encryption": "TLS 1.3, encrypted sensitive fields",
    "api_security": "Rate limiting, input validation, CORS"
  }
}
```

## Phase 2: Development Implementation

### 3. Frontend Developer Agent Workflow

**Agent Invocation:**
```bash
Use the frontend-developer subagent to implement the user interface based on the technical architecture
```

**Agent Tasks:**
- Create responsive React components with TypeScript
- Implement state management with Redux Toolkit
- Build product catalog with search and filtering
- Develop shopping cart and checkout flow
- Ensure accessibility (WCAG 2.1 AA compliance)
- Optimize for performance (Core Web Vitals)

**Expected Output:**
```typescript
// Key Frontend Components Generated

// Product Catalog Component
export const ProductCatalog: React.FC = () => {
  const { products, loading, filters } = useAppSelector(state => state.products);
  const dispatch = useAppDispatch();
  
  return (
    <div className="product-catalog" role="main" aria-label="Product Catalog">
      <SearchBar onSearch={(query) => dispatch(searchProducts(query))} />
      <FilterPanel filters={filters} onFilterChange={dispatch(updateFilters)} />
      <ProductGrid products={products} loading={loading} />
      <Pagination />
    </div>
  );
};

// Shopping Cart Component
export const ShoppingCart: React.FC = () => {
  const { items, total } = useCart();
  
  return (
    <div className="shopping-cart" role="region" aria-label="Shopping Cart">
      {items.map(item => (
        <CartItem key={item.id} item={item} />
      ))}
      <CartSummary total={total} />
      <CheckoutButton disabled={items.length === 0} />
    </div>
  );
};

// Performance Optimizations
// - React.memo for expensive components
// - useCallback for event handlers
// - Code splitting with React.lazy
// - Image optimization with next/image
// - Service Worker for caching
```

### 4. Backend Developer Agent Workflow

**Agent Invocation:**
```bash
Use the backend-developer subagent to build the server-side APIs and business logic
```

**Agent Tasks:**
- Create RESTful APIs with Express and TypeScript
- Implement database models with Prisma ORM
- Build authentication and authorization system
- Integrate Stripe payment processing
- Create order management workflows
- Implement search functionality with Elasticsearch

**Expected Output:**
```typescript
// Key Backend Services Generated

// Product Service API
@Controller('/api/products')
export class ProductController {
  @Get('/')
  @UseGuards(RateLimitGuard)
  async getProducts(
    @Query() query: ProductQueryDto,
    @Req() req: Request
  ): Promise<ProductListResponse> {
    const { page, limit, search, category, priceRange } = query;
    
    const products = await this.productService.findProducts({
      page,
      limit,
      search,
      filters: { category, priceRange }
    });
    
    return {
      products: products.items,
      pagination: products.pagination,
      facets: products.facets
    };
  }
  
  @Post('/')
  @UseGuards(JwtAuthGuard, AdminGuard)
  async createProduct(@Body() createProductDto: CreateProductDto) {
    return this.productService.create(createProductDto);
  }
}

// Order Processing Service
export class OrderService {
  async processOrder(orderData: CreateOrderDto): Promise<Order> {
    const transaction = await this.prisma.$transaction(async (tx) => {
      // 1. Validate product availability
      const products = await this.validateProducts(orderData.items, tx);
      
      // 2. Calculate total with taxes and shipping
      const total = this.calculateTotal(products, orderData.shippingAddress);
      
      // 3. Process payment with Stripe
      const payment = await this.stripeService.processPayment({
        amount: total.totalCents,
        currency: 'usd',
        paymentMethodId: orderData.paymentMethodId
      });
      
      // 4. Create order record
      const order = await tx.order.create({
        data: {
          userId: orderData.userId,
          items: products,
          total: total.total,
          status: 'confirmed',
          paymentId: payment.id
        }
      });
      
      // 5. Update inventory
      await this.updateInventory(products, tx);
      
      // 6. Send confirmation email
      this.emailService.sendOrderConfirmation(order);
      
      return order;
    });
    
    return transaction;
  }
}

// Database Schema (Prisma)
model Product {
  id          String   @id @default(cuid())
  name        String
  description String
  price       Decimal  @db.Decimal(10,2)
  category    String
  inventory   Int
  images      String[]
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@map("products")
}

model Order {
  id          String      @id @default(cuid())
  userId      String
  items       OrderItem[]
  total       Decimal     @db.Decimal(10,2)
  status      OrderStatus
  paymentId   String
  createdAt   DateTime    @default(now())
  
  user        User        @relation(fields: [userId], references: [id])
  
  @@map("orders")
}
```

### 5. DevOps Engineer Agent Workflow

**Agent Invocation:**
```bash
Use the devops-engineer subagent to set up infrastructure, CI/CD pipelines, and deployment automation
```

**Agent Tasks:**
- Design AWS infrastructure with Terraform
- Create Docker containers for all services
- Set up CI/CD pipeline with GitHub Actions
- Configure monitoring and logging
- Implement security scanning and compliance
- Create staging and production environments

**Expected Output:**
```yaml
# Docker Compose for Development
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://backend:3001
    depends_on:
      - backend
      
  backend:
    build: ./backend
    ports:
      - "3001:3001"
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/techshop
      - REDIS_URL=redis://redis:6379
      - STRIPE_SECRET_KEY=${STRIPE_SECRET_KEY}
    depends_on:
      - postgres
      - redis
      
  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=techshop
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
      
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

# Terraform Infrastructure
resource "aws_ecs_cluster" "techshop" {
  name = "techshop-cluster"
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}

resource "aws_ecs_service" "backend" {
  name            = "techshop-backend"
  cluster         = aws_ecs_cluster.techshop.id
  task_definition = aws_ecs_task_definition.backend.arn
  desired_count   = 2
  
  load_balancer {
    target_group_arn = aws_lb_target_group.backend.arn
    container_name   = "backend"
    container_port   = 3001
  }
  
  depends_on = [aws_lb_listener.backend]
}

# GitHub Actions CI/CD Pipeline
name: Deploy TechShop
on:
  push:
    branches: [main]
    
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run test:coverage
      - run: npm run lint
      - run: npm run type-check
      
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        with:
          args: --severity-threshold=high
          
  deploy:
    needs: [test, security-scan]
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to AWS ECS
        run: |
          aws ecs update-service \
            --cluster techshop-cluster \
            --service techshop-backend \
            --force-new-deployment
```

## Phase 3: Quality Assurance & Reliability

### 6. Quality Assurance Agent Workflow

**Agent Invocation:**
```bash
Use the qa-engineer subagent to create comprehensive testing strategies and automation
```

**Agent Tasks:**
- Create unit tests for all components and services
- Build integration tests for API endpoints
- Develop end-to-end tests for user workflows
- Set up performance testing with load scenarios
- Create accessibility testing automation
- Implement security testing protocols

**Expected Output:**
```typescript
// Unit Tests (Jest + React Testing Library)
describe('ProductCatalog', () => {
  it('renders products correctly', async () => {
    const mockProducts = [
      { id: '1', name: 'iPhone 15', price: 999 },
      { id: '2', name: 'MacBook Pro', price: 2499 }
    ];
    
    render(<ProductCatalog products={mockProducts} />);
    
    expect(screen.getByText('iPhone 15')).toBeInTheDocument();
    expect(screen.getByText('$999')).toBeInTheDocument();
  });
  
  it('handles search functionality', async () => {
    const mockSearch = jest.fn();
    render(<ProductCatalog onSearch={mockSearch} />);
    
    const searchInput = screen.getByRole('searchbox');
    await user.type(searchInput, 'iPhone');
    
    expect(mockSearch).toHaveBeenCalledWith('iPhone');
  });
});

// Integration Tests (Supertest)
describe('Product API', () => {
  it('GET /api/products returns paginated results', async () => {
    const response = await request(app)
      .get('/api/products?page=1&limit=10')
      .expect(200);
      
    expect(response.body.products).toHaveLength(10);
    expect(response.body.pagination.total).toBeGreaterThan(0);
  });
  
  it('POST /api/products requires admin auth', async () => {
    await request(app)
      .post('/api/products')
      .send({ name: 'Test Product', price: 100 })
      .expect(401);
  });
});

// E2E Tests (Cypress)
describe('Checkout Flow', () => {
  it('completes purchase successfully', () => {
    cy.visit('/products');
    cy.get('[data-cy=product-card]').first().click();
    cy.get('[data-cy=add-to-cart]').click();
    cy.get('[data-cy=cart-icon]').click();
    cy.get('[data-cy=checkout-button]').click();
    
    // Fill checkout form
    cy.get('[data-cy=email]').type('test@example.com');
    cy.get('[data-cy=card-number]').type('4242424242424242');
    cy.get('[data-cy=expiry]').type('12/25');
    cy.get('[data-cy=cvc]').type('123');
    
    cy.get('[data-cy=place-order]').click();
    cy.get('[data-cy=order-confirmation]').should('be.visible');
  });
});

// Performance Tests (Artillery)
config:
  target: 'https://techshop-api.com'
  phases:
    - duration: 60
      arrivalRate: 10
    - duration: 120
      arrivalRate: 50
      
scenarios:
  - name: Browse products
    flow:
      - get:
          url: '/api/products'
      - think: 2
      - get:
          url: '/api/products/{{ $randomString() }}'
          
  - name: Complete purchase
    flow:
      - post:
          url: '/api/auth/login'
          json:
            email: 'test@example.com'
            password: 'password123'
      - post:
          url: '/api/orders'
          json:
            items: [{ productId: '1', quantity: 1 }]
            paymentMethodId: 'pm_test'
```

### 7. Site Reliability Agent Workflow

**Agent Invocation:**
```bash
Use the sre-engineer subagent to implement monitoring, alerting, and performance optimization
```

**Agent Tasks:**
- Set up comprehensive monitoring with Prometheus and Grafana
- Create alerting rules for critical metrics
- Implement log aggregation and analysis
- Design incident response procedures
- Create performance optimization recommendations
- Establish SLA monitoring and reporting

**Expected Output:**
```yaml
# Prometheus Configuration
global:
  scrape_interval: 15s
  
scrape_configs:
  - job_name: 'techshop-backend'
    static_configs:
      - targets: ['backend:3001']
    metrics_path: '/metrics'
    
  - job_name: 'postgres-exporter'
    static_configs:
      - targets: ['postgres-exporter:9187']

# Alerting Rules
groups:
  - name: techshop-alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }}% for the last 5 minutes"
          
      - alert: DatabaseConnectionsHigh
        expr: postgres_connections > 80
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "Database connections high"
          
      - alert: ResponseTimeHigh
        expr: histogram_quantile(0.95, http_request_duration_seconds) > 2
        for: 5m
        labels:
          severity: warning

# Grafana Dashboard Configuration
{
  "dashboard": {
    "title": "TechShop Performance Dashboard",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{endpoint}}"
          }
        ]
      },
      {
        "title": "Response Time (95th percentile)",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, http_request_duration_seconds)",
            "legendFormat": "95th percentile"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "singlestat",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m]) * 100",
            "legendFormat": "Error %"
          }
        ]
      }
    ]
  }
}

# Incident Response Runbook
## High Error Rate Alert
1. Check application logs for error patterns
2. Verify database connectivity and performance
3. Check external service dependencies (Stripe, AWS)
4. Scale up application instances if needed
5. Create incident ticket if issue persists > 10 minutes

## Database Performance Issues
1. Check active connections and slow queries
2. Verify database disk space and memory usage
3. Consider enabling read replicas for heavy queries
4. Optimize slow queries identified in logs
```

### 8. Development Coordinator Agent Workflow

**Agent Invocation:**
```bash
Use the dev-coordinator subagent to orchestrate the complete workflow and manage project coordination
```

**Agent Tasks:**
- Coordinate handoffs between all development phases
- Track project milestones and deliverables
- Manage dependency resolution and integration
- Generate progress reports and stakeholder updates
- Optimize development workflow and identify bottlenecks
- Ensure quality gates are met at each phase

**Expected Output:**
```json
{
  "project_status": {
    "phase": "Production Deployment",
    "completion": "95%",
    "timeline": "On track for 3-month delivery",
    "budget": "Within $50,000 budget constraint"
  },
  
  "team_performance": {
    "product_manager": {
      "deliverables": "Complete - requirements, roadmap, success metrics",
      "handoff_quality": "Excellent",
      "stakeholder_satisfaction": "4.8/5"
    },
    "technical_architect": {
      "deliverables": "Complete - architecture, tech stack, security design",
      "design_quality": "High scalability and maintainability",
      "integration_success": "100%"
    },
    "frontend_developer": {
      "deliverables": "Complete - React app, accessibility, performance optimization",
      "code_quality": "95% test coverage, A+ Lighthouse scores",
      "user_experience": "Exceeds design requirements"
    },
    "backend_developer": {
      "deliverables": "Complete - APIs, database, payment integration",
      "performance": "Sub-100ms average response time",
      "security": "Passes all security scans"
    },
    "devops_engineer": {
      "deliverables": "Complete - infrastructure, CI/CD, monitoring",
      "reliability": "99.9% uptime SLA met",
      "deployment_automation": "Zero-downtime deployments"
    },
    "qa_engineer": {
      "deliverables": "Complete - test automation, performance testing",
      "coverage": "98% code coverage, 100% critical path coverage",
      "quality_metrics": "Zero critical bugs in production"
    },
    "sre_engineer": {
      "deliverables": "Complete - monitoring, alerting, incident response",
      "observability": "Full stack visibility implemented",
      "performance": "All SLAs within targets"
    }
  },
  
  "quality_gates": {
    "code_review": "100% of code reviewed by AI agents",
    "security_scan": "Zero high-severity vulnerabilities",
    "performance": "Core Web Vitals all green",
    "accessibility": "WCAG 2.1 AA compliance verified",
    "testing": "All automated tests passing",
    "deployment": "Successful production deployment"
  },
  
  "stakeholder_report": {
    "executive_summary": "TechShop e-commerce platform successfully delivered on time and within budget. All technical requirements met with performance exceeding initial targets.",
    "key_achievements": [
      "Sub-3 second page load times achieved",
      "99.9% uptime maintained during launch",
      "Zero security vulnerabilities in production",
      "Mobile-first responsive design completed",
      "PCI DSS compliance certified"
    ],
    "metrics": {
      "development_velocity": "22 story points per sprint (target: 20)",
      "bug_rate": "0.1 bugs per feature (target: < 0.5)",
      "deployment_frequency": "Daily deployments achieved",
      "lead_time": "3 days from feature to production"
    }
  },
  
  "lessons_learned": {
    "successes": [
      "AI agent coordination reduced development time by 40%",
      "Automated testing prevented 15 potential production bugs",
      "Infrastructure as code eliminated environment inconsistencies"
    ],
    "improvements": [
      "Earlier performance testing could optimize database queries",
      "More frequent stakeholder demos would reduce late-stage changes"
    ]
  },
  
  "next_phase_recommendations": {
    "immediate": [
      "Launch user feedback collection system",
      "Monitor conversion rates and user behavior",
      "Prepare for expected traffic scaling"
    ],
    "short_term": [
      "Implement AI-powered product recommendations",
      "Add advanced analytics dashboard",
      "Develop mobile application"
    ],
    "long_term": [
      "Multi-vendor marketplace capabilities",
      "International expansion features",
      "Advanced inventory management system"
    ]
  }
}
```

## Final Deliverables

### Complete Application Stack
- **Frontend**: Modern React application with TypeScript, optimized for performance and accessibility
- **Backend**: Scalable Node.js APIs with comprehensive business logic and integrations
- **Database**: Well-designed PostgreSQL schema with Redis caching
- **Infrastructure**: Production-ready AWS deployment with monitoring and alerting
- **Testing**: Comprehensive test suite with 98% coverage
- **Documentation**: Complete technical documentation and user guides

### Key Metrics Achieved
- **Performance**: Page load times under 3 seconds
- **Reliability**: 99.9% uptime SLA
- **Security**: Zero high-severity vulnerabilities
- **Quality**: 98% automated test coverage
- **Accessibility**: WCAG 2.1 AA compliant
- **Timeline**: Delivered on schedule within 3 months
- **Budget**: Completed within $50,000 constraint

### Technology Stack Summary
```
Frontend: React 18 + Next.js 14 + TypeScript + Tailwind CSS
Backend: Node.js + Express + TypeScript + Prisma ORM
Database: PostgreSQL + Redis + Elasticsearch
Payment: Stripe API integration
Infrastructure: AWS ECS + RDS + ElastiCache + S3
Monitoring: Prometheus + Grafana + CloudWatch
CI/CD: GitHub Actions + Docker + Terraform
Testing: Jest + Cypress + Artillery + Lighthouse
```

This example demonstrates how the Full Stack SubAgents work together to deliver a complete, production-ready application while maintaining high quality standards and meeting all project constraints. Each agent brings specialized expertise while the coordinator ensures seamless integration and optimal outcomes.