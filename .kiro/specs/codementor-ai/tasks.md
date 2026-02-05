# Implementation Plan - MVP

## MVP Goal
Prove: "AI can understand your codebase and guide your learning better than tutorials or autocomplete."

## MVP System Flow
```
User → Create Learning Plan → AI Generates 3-5 Milestones → User Writes Code → 
MCP Reads Code → Gemini Analyzes → Socratic Feedback → Milestone Completed
```

---

- [ ] 1. Set up Spring Boot project structure
  - Create Spring Boot 3.2 project with Java 17
  - Add dependencies: Spring Web, Spring Data JPA, LangChain4j, H2 database
  - Configure application.properties for H2 and Gemini API key
  - Set up package structure: controllers, services, repositories, models, config
  - _Requirements: All requirements depend on proper setup_

- [ ] 2. Implement simple data models
  - Create User entity (id, email, passwordHash)
  - Create LearningPlan entity (id, user, technology, projectName, projectDescription, durationDays, skillLevel, projectPath)
  - Create Milestone entity (id, learningPlan, sequenceNumber, title, description, learningObjectives, completed)
  - Create ChatMessage entity (id, user, role, content, createdAt)
  - Define SkillLevel enum (BEGINNER, INTERMEDIATE, ADVANCED)
  - Define MessageRole enum (USER, ASSISTANT)
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 5.1, 5.2, 5.3_

- [ ] 3. Create simple MCP server (Node.js or Java)
  - Set up MCP server project
  - Implement read_file tool - reads and returns file contents
  - Implement get_project_structure tool - returns folder hierarchy as JSON
  - Test tools with sample project directory
  - Document how to start MCP server
  - _Requirements: 2.1, 2.2_

- [ ] 4. Implement MCP integration service in backend
  - Create McpService interface with readFile() and getProjectStructure() methods
  - Implement HTTP client to call MCP server tools
  - Add simple error handling for file not found
  - Test integration with MCP server
  - _Requirements: 2.1, 2.2, 7.1_

- [ ] 5. Implement Curriculum Generator with Gemini
  - Configure LangChain4j with Gemini 1.5 Pro API
  - Create CurriculumGenerator service
  - Design prompt: "Generate a learning plan for {technology} over {days} days at {level} level. Include: project idea, 3-5 milestones with titles, descriptions, and learning objectives."
  - Parse AI response into LearningPlan object with milestones
  - Save generated plan to H2 database
  - _Requirements: 1.1, 1.2, 1.3, 1.4_

- [ ] 6. Implement Socratic Mentor Agent
  - Create SocraticMentor service
  - Design Socratic prompt: "You are a Socratic mentor. Ask guiding questions, don't provide code solutions. Point out missing concepts."
  - Implement respondToQuery() method that:
    - Retrieves project structure via MCP
    - Reads relevant files via MCP
    - Includes conversation history
    - Sends context to Gemini
    - Returns Socratic response
  - Save messages to database
  - _Requirements: 2.3, 3.1, 3.2, 3.3, 5.3_

- [ ] 7. Implement simple verification engine
  - Create VerificationEngine service
  - Implement verifyMilestone() method that:
    - Uses MCP to get project structure
    - Checks if required files exist
    - Uses MCP to read files and check for required annotations (simple string matching)
    - Returns true/false
  - Update milestone.completed field when verification passes
  - _Requirements: 4.1, 4.2, 4.3_

- [ ] 8. Create REST API controllers
  - Create AuthController with POST /api/auth/login (simple session-based, mock auth OK for MVP)
  - Create PlanController with:
    - POST /api/plans - create learning plan
    - GET /api/plans/{id} - get plan with milestones
  - Create ChatController with:
    - POST /api/chat - send message, get Socratic response
    - GET /api/chat/history - get conversation history
  - Create VerificationController with:
    - POST /api/milestones/{id}/verify - trigger verification
  - Add basic input validation
  - _Requirements: 1.1, 1.4, 3.1, 4.3, 5.1, 5.3, 6.1, 6.2_

- [ ]* 8.1 Write unit tests for core services
  - Test CurriculumGenerator returns 3-5 milestones
  - Test SocraticMentor calls MCP for context
  - Test VerificationEngine checks file existence
  - Test API endpoints return correct status codes

- [ ] 9. Checkpoint - Ensure backend works
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 10. Set up React frontend project
  - Create React project with Vite
  - Install dependencies: TailwindCSS, axios, react-router-dom
  - Configure TailwindCSS
  - Set up routing for 4 screens
  - Create basic layout component
  - _Requirements: 5.1, 5.2, 5.3_

- [ ] 11. Implement Login screen
  - Create Login component with email/password form
  - Call POST /api/auth/login
  - Store user session (localStorage is fine for MVP)
  - Redirect to Create Plan screen on success
  - _Requirements: 6.1, 6.3_

- [ ] 12. Implement Create Learning Plan screen
  - Create form with fields: technology, duration (days), skill level dropdown
  - Add input validation
  - Call POST /api/plans on submit
  - Show loading state during generation
  - Navigate to Dashboard on success
  - _Requirements: 1.1, 1.2, 1.3_

- [ ] 13. Implement Dashboard screen
  - Fetch learning plan via GET /api/plans/{id}
  - Display project name and description
  - Show list of 3-5 milestones with:
    - Sequence number
    - Title
    - Status (completed/in-progress)
  - Add "Verify" button for each milestone
  - Add "Chat with Mentor" button linking to chat screen
  - _Requirements: 5.1, 5.3_

- [ ] 14. Implement Chat Interface screen
  - Create chat UI with message list and input box
  - Fetch conversation history via GET /api/chat/history
  - Display user and assistant messages with different styling
  - Send messages via POST /api/chat
  - Show loading indicator while waiting for response
  - Display Socratic questions from mentor
  - _Requirements: 3.1, 3.2, 5.2, 5.3_

- [ ] 15. Add basic error handling
  - Show error messages for failed API calls
  - Display validation errors on forms
  - Add retry buttons where appropriate
  - _Requirements: 7.1, 7.2, 7.3_

- [ ] 16. Final testing and polish
  - Test complete user flow: login → create plan → view dashboard → chat → verify milestone
  - Fix any bugs
  - Add loading states
  - Improve UI styling
  - Prepare demo script

- [ ] 17. Final checkpoint
  - Ensure all tests pass, ask the user if questions arise.

---

## Out of Scope for MVP (Future Work)

The following features are mentioned in the design but NOT implemented for the hackathon MVP:

- Property-based testing with jqwik
- JWT tokens (using simple session auth instead)
- WebSockets for real-time updates
- Background schedulers
- Complex verification with AST parsing
- PostgreSQL (using H2 instead)
- Multi-user roles
- Production security hardening
- Charts and fancy visualizations
- Extensive error handling
- Full MCP tool suite (only read_file and get_project_structure)

These can be added post-hackathon for production deployment.
