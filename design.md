# Design Document

## Overview

CodeMentor AI is an MVP that proves: "AI can understand your codebase and guide your learning better than tutorials or autocomplete." The system has three simple layers: a Spring Boot backend with REST APIs, an MCP server that reads project files (read_file, get_project_structure), and a minimal React frontend with 4 screens.

The core innovation is using MCP to give Gemini 1.5 Pro actual visibility into the user's code, enabling contextual Socratic mentorship that asks guiding questions instead of writing code. This MVP focuses on demonstrating the concept for hackathon evaluation, not production features.

## Architecture

### System Components

```mermaid
graph TB
    subgraph "Frontend Layer"
        UI[React Dashboard]
        WS[WebSocket Client]
    end
    
    subgraph "Backend Layer"
        API[Spring Boot REST API]
        AUTH[JWT Authentication]
        MENTOR[Socratic Mentor Agent]
        VERIFY[Verification Engine]
        CURRIC[Curriculum Generator]
        SCHED[Background Scheduler]
    end
    
    subgraph "AI Layer"
        LC[LangChain4j]
        GEMINI[Gemini 1.5 Pro]
    end
    
    subgraph "MCP Layer"
        MCP[MCP Server]
        FS[File System Access]
    end
    
    subgraph "Data Layer"
        DB[(PostgreSQL)]
    end
    
    UI --> API
    WS --> API
    API --> AUTH
    API --> MENTOR
    API --> VERIFY
    API --> CURRIC
    API --> SCHED
    MENTOR --> LC
    VERIFY --> MCP
    CURRIC --> LC
    SCHED --> MCP
    LC --> GEMINI
    MCP --> FS
    API --> DB
    
    style MCP fill:#e1f5ff
    style GEMINI fill:#fff4e1
## Architecture

### System Components (MVP)

```mermaid
graph TB
    subgraph "Frontend - 4 Screens"
        LOGIN[Login]
        CREATE[Create Plan]
        DASH[Dashboard]
        CHAT[Chat Interface]
    end
    
    subgraph "Backend - Spring Boot"
        API[REST API]
        AUTH[Simple Auth]
        MENTOR[Socratic Mentor]
        VERIFY[Simple Verification]
        CURRIC[Curriculum Generator]
    end
    
    subgraph "AI Layer"
        LC[LangChain4j]
        GEMINI[Gemini 1.5 Pro]
    end
    
    subgraph "MCP Layer - 2 Tools"
        MCP[MCP Server]
        READ[read_file]
        STRUCT[get_project_structure]
    end
    
    subgraph "Data"
        H2[(H2 Database)]
    end
    
    LOGIN --> API
    CREATE --> API
    DASH --> API
    CHAT --> API
    
    API --> AUTH
    API --> MENTOR
    API --> VERIFY
    API --> CURRIC
    
    MENTOR --> LC
    CURRIC --> LC
    VERIFY --> MCP
    MENTOR --> MCP
    
    LC --> GEMINI
    MCP --> READ
    MCP --> STRUCT
    
    API --> H2
    
    style MCP fill:#e1f5ff
    style GEMINI fill:#fff4e1
```

### Technology Stack (MVP)

**Backend:**
- Java 17 with Spring Boot 3.2
- LangChain4j for AI orchestration
- Simple session-based auth (no JWT complexity)
- Spring Data JPA
- H2 in-memory database

**AI Integration:**
- Gemini 1.5 Pro via LangChain4j
- Model Context Protocol (MCP) - 2 tools only

**MCP Server:**
- Node.js or Java (simple implementation)
- Tools: read_file, get_project_structure

**Frontend:**
- React 18 with Vite
- TailwindCSS for styling
- Axios for API calls
- Simple routing (4 screens)

**Database:**
- H2 in-memory (no PostgreSQL needed for MVP)

## Components and Interfaces

### Backend Components (MVP)

#### 1. Curriculum Generator

**Responsibility**: Generate learning plans with 3-5 milestones.

**Interface**:
```java
public interface CurriculumGenerator {
    LearningPlan generatePlan(String technology, int days, SkillLevel level);
}
```

**Key Methods**:
- `generatePlan()`: Creates learning plan with project idea and 3-5 milestones

**Dependencies**: LangChain4j, Gemini 1.5 Pro

#### 2. Socratic Mentor Agent

**Responsibility**: Provide guidance through questions, not code solutions.

**Interface**:
```java
public interface SocraticMentor {
    String respondToQuery(String userMessage, String projectPath, List<String> history);
}
```

**Key Methods**:
- `respondToQuery()`: Generates Socratic response using MCP context

**Dependencies**: LangChain4j, MCP Server, Gemini 1.5 Pro

#### 3. Simple Verification Engine

**Responsibility**: Check if files exist and contain required annotations.

**Interface**:
```java
public interface VerificationEngine {
    boolean verifyMilestone(Milestone milestone, String projectPath);
}
```

**Key Methods**:
- `verifyMilestone()`: Checks file existence and basic annotations

**Dependencies**: MCP Server

#### 4. MCP Integration Service

**Responsibility**: Call MCP tools to read files and structure.

**Interface**:
```java
public interface McpService {
    String readFile(String filePath);
    Map<String, List<String>> getProjectStructure(String projectPath);
}
```

**Key Methods**:
- `readFile()`: Returns file contents as string
- `getProjectStructure()`: Returns folder hierarchy

**Dependencies**: MCP Server (external process)

### MCP Server Components (MVP)

#### MCP Tool Definitions (Only 2 Tools)

**1. read_file**
```json
{
  "name": "read_file",
  "description": "Read specific file contents",
  "inputSchema": {
    "type": "object",
    "properties": {
      "filePath": {"type": "string"}
    },
    "required": ["filePath"]
  }
}
```

**2. get_project_structure**
```json
{
  "name": "get_project_structure",
  "description": "Get folder hierarchy and file names",
  "inputSchema": {
    "type": "object",
    "properties": {
      "projectPath": {"type": "string"}
    },
    "required": ["projectPath"]
  }
}
```

### Frontend Components (MVP - 4 Screens)

#### 1. Login Screen

**Responsibility**: Simple authentication.

**Features**:
- Email/password form
- Mock auth acceptable for MVP
- Store user session

#### 2. Create Learning Plan Screen

**Responsibility**: Input form for plan generation.

**Features**:
- Technology input
- Duration (days)
- Skill level dropdown
- Submit to generate plan

#### 3. Dashboard Screen

**Responsibility**: Show learning plan and milestones.

**Features**:
- Project name and description
- List of 3-5 milestones
- Status indicators (completed/in-progress)
- Link to chat

#### 4. Chat Interface Screen

**Responsibility**: Conversation with Socratic Mentor.

**Features**:
- Message history
- Input box
- Send button
- Display mentor questions

## Data Models

### Core Entities (Simplified for MVP)

#### User
```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private String passwordHash;
    
    @OneToMany(mappedBy = "user")
    private List<LearningPlan> learningPlans;
}
```

#### LearningPlan
```java
@Entity
@Table(name = "learning_plans")
public class LearningPlan {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @Column(nullable = false)
    private String technology;
    
    @Column(nullable = false)
    private String projectName;
    
    @Column(columnDefinition = "TEXT")
    private String projectDescription;
    
    private Integer durationDays;
    
    @Enumerated(EnumType.STRING)
    private SkillLevel skillLevel;
    
    @OneToMany(mappedBy = "learningPlan", cascade = CascadeType.ALL)
    private List<Milestone> milestones;
    
    @Column(nullable = false)
    private String projectPath;
}
```

#### Milestone
```java
@Entity
@Table(name = "milestones")
public class Milestone {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "learning_plan_id", nullable = false)
    private LearningPlan learningPlan;
    
    @Column(nullable = false)
    private Integer sequenceNumber;
    
    @Column(nullable = false)
    private String title;
    
    @Column(columnDefinition = "TEXT")
    private String description;
    
    @ElementCollection
    private List<String> learningObjectives;
    
    @Column(nullable = false)
    private Boolean completed = false;
}
```

#### ChatMessage
```java
@Entity
@Table(name = "chat_messages")
public class ChatMessage {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @Enumerated(EnumType.STRING)
    private MessageRole role; // USER or ASSISTANT
    
    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;
    
    @CreationTimestamp
    private LocalDateTime createdAt;
}
```

### Enumerations

```java
public enum SkillLevel {
    BEGINNER,
    INTERMEDIATE,
    ADVANCED
}

public enum MessageRole {
    USER,
    ASSISTANT
}
```


## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

**Note for MVP**: Property-based testing is mentioned as future work for production. The MVP will focus on basic unit tests to validate core functionality for the hackathon demo.

### Property 1: Learning plan generation completeness

*For any* valid technology name, timeframe, and skill level, generating a learning plan should produce a plan with 3-5 milestones where each milestone contains at least one learning objective and the plan includes a non-empty project description.

**Validates: Requirements 1.1, 1.2, 1.3**
**MVP Status: Future work - basic unit tests will verify this**

### Property 2: Learning plan persistence

*For any* generated learning plan, saving it to the H2 database and then retrieving it by ID should return a plan with identical technology, project name, and milestones.

**Validates: Requirements 1.4**
**MVP Status: Future work - basic unit tests will verify this**

### Property 3: MCP file reading

*For any* valid file path, calling read_file should return the complete file contents as a string.

**Validates: Requirements 2.1**
**MVP Status: Future work - basic unit tests will verify this**

### Property 4: MCP structure reading

*For any* valid project path, calling get_project_structure should return a map containing all folders and files in the hierarchy.

**Validates: Requirements 2.2**
**MVP Status: Future work - basic unit tests will verify this**

### Property 5: Socratic responses include context

*For any* user message, the Socratic Mentor should invoke MCP tools to retrieve project context before generating a response.

**Validates: Requirements 2.3, 3.3**
**MVP Status: Future work - basic unit tests will verify this**

### Property 6: Verification checks files

*For any* milestone with required files, verification should return true if and only if all required files exist in the project.

**Validates: Requirements 4.1, 4.2, 4.3**
**MVP Status: Future work - basic unit tests will verify this**

### Property 7: Authentication session creation

*For any* valid login credentials, the system should create a session and associate subsequent requests with that user.

**Validates: Requirements 6.1, 6.2**
**MVP Status: Future work - basic unit tests will verify this**

### Property 8: Error messages for file access

*For any* MCP file access failure, the system should return an error message indicating the file could not be accessed.

**Validates: Requirements 7.1**
**MVP Status: Future work - basic unit tests will verify this**


## Error Handling

### Error Categories (MVP)

#### 1. MCP Communication Errors

**Scenarios**:
- MCP Server unavailable
- File not found
- Permission denied

**Handling Strategy**:
- Return user-friendly error: "Unable to read project files"
- Log error details server-side
- Continue with limited functionality

#### 2. AI Service Errors

**Scenarios**:
- Gemini API timeout
- Rate limiting
- Invalid response

**Handling Strategy**:
- Return error: "AI service temporarily unavailable"
- Log API errors
- Suggest retry

#### 3. Validation Errors

**Scenarios**:
- Missing required fields
- Invalid formats

**Handling Strategy**:
- Return 400 Bad Request with field errors
- Use Bean Validation (@Valid, @NotNull)

### Simple Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(McpException.class)
    public ResponseEntity<String> handleMcpError(McpException ex) {
        log.error("MCP error", ex);
        return ResponseEntity.status(503).body("Unable to read project files");
    }
    
    @ExceptionHandler(AiException.class)
    public ResponseEntity<String> handleAiError(AiException ex) {
        log.error("AI error", ex);
        return ResponseEntity.status(503).body("AI service temporarily unavailable");
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }
}
```

## Testing Strategy

### Overview (MVP)

For the hackathon MVP, we'll focus on basic unit tests to validate core functionality. Property-based testing with jqwik is mentioned as future work for production deployment.

### Unit Testing

**Framework**: JUnit 5 with Spring Boot Test

**Scope**:
- Core business logic
- API endpoints
- MCP integration
- Basic error scenarios

**Key Test Areas**:

1. **Curriculum Generator**
   - Test plan generation returns 3-5 milestones
   - Test each milestone has learning objectives
   - Test project description is non-empty

2. **Socratic Mentor**
   - Test responses are questions (contain "?")
   - Test MCP context is retrieved
   - Test conversation history is included

3. **Verification Engine**
   - Test file existence checking
   - Test annotation detection
   - Test milestone completion logic

4. **MCP Integration**
   - Test read_file returns content
   - Test get_project_structure returns hierarchy
   - Test error handling for missing files

5. **API Controllers**
   - Test authentication
   - Test plan creation
   - Test milestone retrieval
   - Test chat endpoints

**Example Unit Tests**:
```java
@Test
void shouldGeneratePlanWith3To5Milestones() {
    LearningPlan plan = curriculumGenerator.generatePlan("Spring Boot", 7, SkillLevel.BEGINNER);
    assertThat(plan.getMilestones()).hasSizeBetween(3, 5);
}

@Test
void shouldIncludeProjectContextInMentorResponse() {
    String response = socraticMentor.respondToQuery("How do I start?", "/path/to/project", List.of());
    verify(mcpService).readFile(anyString());
}

@Test
void shouldMarkMilestoneCompleteWhenFilesExist() {
    // Setup milestone with required files
    // Create those files
    boolean result = verificationEngine.verifyMilestone(milestone, projectPath);
    assertThat(result).isTrue();
}
```

### Integration Testing

**Scope**:
- End-to-end API workflows
- H2 database integration
- Full user journey

**Example**:
```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class LearningPlanIntegrationTest {
    
    @Test
    void shouldCreateAndRetrieveLearningPlan() {
        // Login
        // Create plan
        // Retrieve plan
        // Verify milestones
    }
}
```

### Future Work: Property-Based Testing

For production deployment, implement property-based testing with jqwik:

- Generate random learning plans and verify persistence
- Generate random file structures and verify MCP reading
- Generate random user inputs and verify validation
- Run 100+ iterations per property

**Note**: This is out of scope for the MVP hackathon demo but should be added before production use.

### Test Coverage Goals (MVP)

- Unit tests for all core services
- Integration tests for main user flows
- Basic error scenario coverage
- Focus on demonstrating the core concept works

