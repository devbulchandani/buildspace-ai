# Requirements Document

## Introduction

CodeMentor AI is an MVP that proves one core concept: "AI can understand your codebase and guide your learning better than tutorials or autocomplete." The system uses Model Context Protocol (MCP) to read actual project files and provides Socratic-style mentorship that asks guiding questions instead of writing code. This minimal viable product focuses on demonstrating contextual code understanding and conceptual guidance for hackathon evaluation.

## Glossary

- **CodeMentor System**: The MVP learning platform with backend API, MCP integration, and simple frontend
- **MCP Server**: Model Context Protocol server that reads project files (read_file, get_project_structure)
- **Curriculum Generator**: Component that creates learning plans with 3-5 milestones
- **Socratic Mentor Agent**: AI component that asks guiding questions instead of providing code solutions
- **Verification Engine**: Simple checker that verifies file existence and basic annotations
- **Learning Plan**: A project idea with 3-5 milestones for a specific technology
- **Milestone**: A learning objective that teaches one specific concept
- **User**: Developer using the CodeMentor System

## Requirements

### Requirement 1

**User Story:** As a developer, I want to generate a learning plan with a project idea, so that I have a clear roadmap for learning a new technology.

#### Acceptance Criteria

1. WHEN a User submits technology name, timeframe, and skill level, THEN the Curriculum Generator SHALL create a Learning Plan with 3 to 5 milestones
2. WHEN the Curriculum Generator creates a Learning Plan, THEN the CodeMentor System SHALL include learning objectives for each milestone
3. WHEN a Learning Plan is generated, THEN the CodeMentor System SHALL define a project idea that incorporates all milestones
4. WHEN a Learning Plan is created, THEN the CodeMentor System SHALL persist the plan to the H2 database

### Requirement 2

**User Story:** As a developer, I want the AI to read my project files, so that I receive contextually relevant guidance without explaining my code.

#### Acceptance Criteria

1. WHEN the MCP Server receives a read_file request, THEN the MCP Server SHALL return the complete file contents
2. WHEN the MCP Server receives a get_project_structure request, THEN the MCP Server SHALL return the folder hierarchy with file names
3. WHEN the Socratic Mentor Agent needs project context, THEN the CodeMentor System SHALL invoke MCP tools to retrieve relevant files

### Requirement 3

**User Story:** As a developer, I want to receive Socratic guidance through questions, so that I develop genuine understanding instead of copying code.

#### Acceptance Criteria

1. WHEN a User asks for help with code, THEN the Socratic Mentor Agent SHALL respond with guiding questions rather than complete code solutions
2. WHEN the Socratic Mentor Agent detects missing concepts, THEN the Socratic Mentor Agent SHALL point out what is missing and ask how to address it
3. WHEN the Socratic Mentor Agent generates responses, THEN the CodeMentor System SHALL include the current project context from MCP

### Requirement 4

**User Story:** As a developer, I want simple verification of my progress, so that I know when I have completed a milestone.

#### Acceptance Criteria

1. WHEN the Verification Engine checks a milestone, THEN the Verification Engine SHALL verify that required files exist
2. WHEN the Verification Engine evaluates code, THEN the Verification Engine SHALL check for presence of required annotations
3. WHEN verification passes, THEN the Verification Engine SHALL mark the milestone as completed

### Requirement 5

**User Story:** As a developer, I want to view my learning plan and chat with the mentor, so that I can track progress and get guidance.

#### Acceptance Criteria

1. WHEN a User accesses the dashboard, THEN the CodeMentor System SHALL display the Learning Plan with milestone status
2. WHEN a User opens the chat interface, THEN the CodeMentor System SHALL display conversation history
3. WHEN a User sends a message, THEN the Socratic Mentor Agent SHALL respond with project-aware guidance

### Requirement 6

**User Story:** As a developer, I want simple authentication, so that my learning plan is saved to my account.

#### Acceptance Criteria

1. WHEN a User logs in with valid credentials, THEN the CodeMentor System SHALL create an authenticated session
2. WHEN a User makes requests, THEN the CodeMentor System SHALL associate data with the authenticated user
3. WHEN authentication fails, THEN the CodeMentor System SHALL return an error message

### Requirement 7

**User Story:** As a system, I want to handle errors gracefully, so that users receive helpful feedback when issues occur.

#### Acceptance Criteria

1. WHEN the MCP Server cannot access files, THEN the CodeMentor System SHALL return an error message indicating file access issues
2. WHEN the AI service is unavailable, THEN the CodeMentor System SHALL return a service unavailable error
3. WHEN invalid input is provided, THEN the CodeMentor System SHALL return validation error messages
