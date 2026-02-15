# Requirements Document: Cortex AI-Powered Learning Intelligence Engine

## Introduction

Cortex is an AI-powered learning intelligence system that addresses a critical gap in educational technology: students often believe they understand concepts but fail exams due to hidden knowledge gaps and inefficient study strategies. Unlike traditional AI tools that merely summarize or explain content, Cortex actively diagnoses the structure of student understanding through reverse teaching, builds a dynamic knowledge dependency graph, detects root conceptual gaps, and generates mathematically optimized learning paths based on understanding weaknesses and exam importance.

The system employs four core engines working in concert: a Knowledge Graph Builder that extracts and structures concepts from syllabi, a Reverse Teaching Evaluation Engine that assesses understanding through student explanations, a Weakness Detection system that visualizes understanding as an interactive heatmap, and a Learning Path Compression Engine that prioritizes study topics using graph theory and strategic optimization algorithms.

## Glossary

- **Cortex_System**: The complete AI-powered learning intelligence platform
- **Knowledge_Graph_Builder**: Engine that extracts concepts from syllabi and builds dependency graphs
- **Reverse_Teaching_Engine**: Engine that evaluates student understanding through their explanations
- **Weakness_Detector**: Engine that identifies understanding gaps and generates visual heatmaps
- **Learning_Path_Engine**: Engine that generates optimized study sequences
- **Concept_Node**: A single learning concept in the knowledge graph
- **Dependency_DAG**: Directed Acyclic Graph representing prerequisite relationships between concepts
- **Understanding_Score**: Computed metric (0-1) representing mastery of a concept
- **Exam_Weight**: Numerical value representing concept importance based on question frequency
- **Priority_Score**: Computed metric determining study order
- **Completeness_Score**: Metric measuring coverage of concept explanation
- **Coherence_Score**: Metric measuring logical consistency of explanation
- **Root_Gap**: A fundamental misunderstanding in prerequisite concepts
- **Adversarial_Question**: Edge case question generated to test understanding depth
- **Understanding_Heatmap**: Visual representation of knowledge mastery across all concepts
- **Learning_Mode**: Strategy for path optimization (Survival, Rank, or Interview)
- **User**: Student using the Cortex system
- **Syllabus**: Course curriculum document (PDF or text format)
- **Session**: Single interaction period where user explains concepts

## Requirements

### Requirement 1: Knowledge Graph Construction from Syllabus

**User Story:** As a student, I want to upload my course syllabus, so that the system can automatically identify all concepts and their relationships.

#### Acceptance Criteria

1. WHEN a User uploads a syllabus in PDF or text format, THE Knowledge_Graph_Builder SHALL extract all distinct concepts using LLM analysis
2. WHEN concepts are extracted, THE Knowledge_Graph_Builder SHALL construct a Dependency_DAG representing prerequisite relationships between concepts
3. WHERE previous year exam papers are provided, THE Knowledge_Graph_Builder SHALL assign Exam_Weight values to each Concept_Node based on question frequency analysis
4. WHERE no exam papers are provided, THE Knowledge_Graph_Builder SHALL assign default uniform Exam_Weight values to all Concept_Nodes
5. WHEN the Dependency_DAG is constructed, THE Knowledge_Graph_Builder SHALL validate that the graph contains no cycles
6. WHEN graph construction completes, THE Cortex_System SHALL persist the Dependency_DAG to the database with all Concept_Nodes and edges

### Requirement 2: Reverse Teaching Interaction

**User Story:** As a student, I want to explain concepts back to the AI in my own words, so that the system can evaluate my true understanding.

#### Acceptance Criteria

1. WHEN a User selects a Concept_Node to explain, THE Reverse_Teaching_Engine SHALL accept explanations via text or voice input
2. WHEN a User provides a voice explanation, THE Reverse_Teaching_Engine SHALL transcribe the audio to text using speech-to-text processing
3. WHEN a User submits an explanation, THE Reverse_Teaching_Engine SHALL generate Adversarial_Questions targeting edge cases and boundary conditions
4. WHEN Adversarial_Questions are generated, THE Reverse_Teaching_Engine SHALL present them to the User for response
5. WHEN the User answers Adversarial_Questions, THE Reverse_Teaching_Engine SHALL evaluate the responses for correctness and depth

### Requirement 3: Understanding Evaluation and Scoring

**User Story:** As a student, I want the system to accurately measure my understanding of each concept, so that I know where I truly stand.

#### Acceptance Criteria

1. WHEN a User completes a reverse teaching Session, THE Reverse_Teaching_Engine SHALL compute a Completeness_Score using vector embeddings and cosine similarity between the explanation and reference content
2. WHEN computing understanding, THE Reverse_Teaching_Engine SHALL calculate a Coherence_Score using LLM-based logical consistency evaluation
3. WHEN all component scores are computed, THE Reverse_Teaching_Engine SHALL calculate Understanding_Score as (Completeness_Score × 0.4) + (Question_Accuracy × 0.4) + (Coherence_Score × 0.2)
4. WHEN evaluating explanations, THE Reverse_Teaching_Engine SHALL flag misconceptions detected in the User explanation
5. WHEN prerequisite concepts have low Understanding_Scores, THE Reverse_Teaching_Engine SHALL flag prerequisite gaps for the current Concept_Node
6. WHEN Understanding_Score is computed, THE Cortex_System SHALL persist the score to the database with timestamp and Session metadata

### Requirement 4: Root Cause Gap Detection

**User Story:** As a student, I want the system to identify the fundamental gaps causing my misunderstandings, so that I can address root problems rather than symptoms.

#### Acceptance Criteria

1. WHEN a Concept_Node has Understanding_Score below 0.6, THE Weakness_Detector SHALL traverse prerequisite nodes in the Dependency_DAG
2. WHEN traversing prerequisites, THE Weakness_Detector SHALL identify Root_Gaps as prerequisite Concept_Nodes with Understanding_Score below 0.5
3. WHEN multiple Root_Gaps exist, THE Weakness_Detector SHALL rank them by dependency centrality in the graph
4. WHEN Root_Gaps are identified, THE Weakness_Detector SHALL associate them with the affected downstream Concept_Nodes
5. WHEN gap analysis completes, THE Cortex_System SHALL store Root_Gap associations in the database

### Requirement 5: Interactive Understanding Heatmap Visualization

**User Story:** As a student, I want to see a visual map of my understanding across all concepts, so that I can quickly identify my weak areas.

#### Acceptance Criteria

1. WHEN a User requests the Understanding_Heatmap, THE Cortex_System SHALL render an interactive graph visualization using the Dependency_DAG structure
2. WHEN rendering the heatmap, THE Cortex_System SHALL color-code each Concept_Node based on Understanding_Score (green for ≥0.7, yellow for 0.4-0.69, red for <0.4)
3. WHEN a User hovers over a Concept_Node, THE Cortex_System SHALL display detailed metrics including Understanding_Score, Completeness_Score, Coherence_Score, and identified misconceptions
4. WHEN a User clicks a Concept_Node, THE Cortex_System SHALL highlight prerequisite dependencies and downstream dependent concepts
5. WHEN the heatmap updates, THE Cortex_System SHALL animate transitions to reflect real-time understanding changes
6. WHEN displaying the heatmap, THE Cortex_System SHALL show edge connections representing prerequisite relationships

### Requirement 6: Optimized Learning Path Generation

**User Story:** As a student, I want the system to tell me exactly what to study in what order, so that I can maximize my exam preparation efficiency.

#### Acceptance Criteria

1. WHEN generating a learning path, THE Learning_Path_Engine SHALL compute Priority_Score for each Concept_Node as (Exam_Weight × 0.5) + (Dependency_Centrality × 0.3) + (Weakness_Factor × 0.2)
2. WHERE Weakness_Factor is defined as (1 - Understanding_Score)
3. WHERE Dependency_Centrality is computed as the number of downstream dependent concepts
4. WHEN a User selects Survival mode, THE Learning_Path_Engine SHALL weight Exam_Weight at 0.7 instead of 0.5 in Priority_Score calculation
5. WHEN a User selects Rank mode, THE Learning_Path_Engine SHALL weight Dependency_Centrality at 0.5 instead of 0.3 in Priority_Score calculation
6. WHEN a User selects Interview mode, THE Learning_Path_Engine SHALL prioritize foundational concepts by weighting nodes with zero prerequisites higher
7. WHEN Priority_Scores are computed, THE Learning_Path_Engine SHALL generate an ordered learning path respecting prerequisite constraints
8. WHEN generating the path, THE Learning_Path_Engine SHALL ensure no concept appears before its prerequisites
9. WHEN the learning path is generated, THE Learning_Path_Engine SHALL estimate time requirements for each Concept_Node based on Understanding_Score and concept complexity
10. WHEN time estimates are computed, THE Learning_Path_Engine SHALL generate a weekly roadmap distributing concepts across available study time

### Requirement 7: Concept Coverage Analysis

**User Story:** As a student, I want to know if my explanation covered all important aspects of a concept, so that I can identify what I missed.

#### Acceptance Criteria

1. WHEN analyzing concept coverage, THE Reverse_Teaching_Engine SHALL generate vector embeddings for the User explanation
2. WHEN embeddings are generated, THE Reverse_Teaching_Engine SHALL generate vector embeddings for reference concept content
3. WHEN comparing coverage, THE Reverse_Teaching_Engine SHALL compute cosine similarity between User explanation embeddings and reference embeddings
4. WHEN similarity is below 0.7, THE Reverse_Teaching_Engine SHALL identify specific missing aspects by comparing embedding clusters
5. WHEN missing aspects are identified, THE Reverse_Teaching_Engine SHALL present them to the User as coverage gaps

### Requirement 8: Multi-Subject Support

**User Story:** As a student, I want to manage multiple subjects simultaneously, so that I can use Cortex for all my courses.

#### Acceptance Criteria

1. WHEN a User creates an account, THE Cortex_System SHALL allow creation of multiple Subject entities
2. WHEN managing subjects, THE Cortex_System SHALL maintain separate Dependency_DAGs for each Subject
3. WHEN computing learning paths, THE Cortex_System SHALL allow cross-subject path generation for exam preparation
4. WHEN displaying the Understanding_Heatmap, THE Cortex_System SHALL allow filtering by Subject
5. WHEN storing data, THE Cortex_System SHALL associate all Concept_Nodes, Understanding_Scores, and Sessions with their respective Subject

### Requirement 9: Session History and Progress Tracking

**User Story:** As a student, I want to see my learning progress over time, so that I can track my improvement and stay motivated.

#### Acceptance Criteria

1. WHEN a User completes a reverse teaching Session, THE Cortex_System SHALL log the Session with timestamp, Concept_Node, Understanding_Score, and duration
2. WHEN a User requests progress history, THE Cortex_System SHALL retrieve all Sessions ordered by timestamp
3. WHEN displaying progress, THE Cortex_System SHALL show Understanding_Score trends over time for each Concept_Node
4. WHEN a Concept_Node is re-evaluated, THE Cortex_System SHALL maintain historical Understanding_Scores rather than overwriting
5. WHEN displaying Session history, THE Cortex_System SHALL show identified misconceptions and Root_Gaps from each Session

### Requirement 10: User Authentication and Data Persistence

**User Story:** As a student, I want my learning data to be saved securely, so that I can access it from any device and continue my progress.

#### Acceptance Criteria

1. WHEN a new User registers, THE Cortex_System SHALL create a User account with secure password hashing
2. WHEN a User logs in, THE Cortex_System SHALL authenticate credentials and establish a secure session
3. WHEN storing User data, THE Cortex_System SHALL persist all data to PostgreSQL database with proper foreign key relationships
4. WHEN a User accesses the system, THE Cortex_System SHALL retrieve only data associated with that User account
5. WHEN database operations fail, THE Cortex_System SHALL maintain data integrity through transaction rollback

### Requirement 11: Adversarial Question Generation

**User Story:** As a student, I want the AI to challenge my understanding with difficult questions, so that I can discover gaps I didn't know existed.

#### Acceptance Criteria

1. WHEN generating Adversarial_Questions, THE Reverse_Teaching_Engine SHALL use LLM prompting to create edge case scenarios
2. WHEN creating questions, THE Reverse_Teaching_Engine SHALL target boundary conditions and common misconceptions for the Concept_Node
3. WHEN a User explanation is incomplete, THE Reverse_Teaching_Engine SHALL generate questions specifically targeting the missing aspects
4. WHEN generating questions, THE Reverse_Teaching_Engine SHALL create between 3 and 5 Adversarial_Questions per Session
5. WHEN questions are generated, THE Reverse_Teaching_Engine SHALL ensure questions are relevant to the specific Concept_Node being evaluated

### Requirement 12: Real-Time Understanding Updates

**User Story:** As a student, I want to see my understanding map update immediately after each session, so that I can see the impact of my learning in real-time.

#### Acceptance Criteria

1. WHEN a Session completes, THE Cortex_System SHALL update the Understanding_Score for the evaluated Concept_Node immediately
2. WHEN Understanding_Score changes, THE Cortex_System SHALL recompute Priority_Scores for all affected Concept_Nodes in the learning path
3. WHEN Priority_Scores change, THE Cortex_System SHALL regenerate the learning path if the order changes significantly
4. WHEN the Understanding_Heatmap is displayed, THE Cortex_System SHALL reflect the most recent Understanding_Scores without requiring page refresh
5. WHEN downstream concepts are affected by prerequisite score changes, THE Cortex_System SHALL update their visual indicators in the heatmap

### Requirement 13: Exam Weight Calibration

**User Story:** As a student, I want the system to accurately reflect which topics are most important for my exam, so that I focus on high-value content.

#### Acceptance Criteria

1. WHEN analyzing exam papers, THE Knowledge_Graph_Builder SHALL count question frequency for each Concept_Node
2. WHEN computing Exam_Weight, THE Knowledge_Graph_Builder SHALL normalize frequencies to a 0-1 scale
3. WHEN multiple exam papers are provided, THE Knowledge_Graph_Builder SHALL compute weighted average based on paper recency
4. WHEN a User manually adjusts Exam_Weight for a Concept_Node, THE Cortex_System SHALL persist the manual override
5. WHEN Exam_Weight changes, THE Cortex_System SHALL recompute Priority_Scores and regenerate the learning path

### Requirement 14: Graph Validation and Integrity

**User Story:** As a student, I want to trust that the knowledge graph accurately represents concept dependencies, so that I learn prerequisites before advanced topics.

#### Acceptance Criteria

1. WHEN constructing the Dependency_DAG, THE Knowledge_Graph_Builder SHALL validate that no cycles exist in the graph
2. WHEN adding a new edge, THE Knowledge_Graph_Builder SHALL check that the edge does not create a cycle
3. WHEN a cycle is detected, THE Knowledge_Graph_Builder SHALL reject the edge and log an error
4. WHEN validating the graph, THE Knowledge_Graph_Builder SHALL ensure all Concept_Nodes are reachable from at least one root node
5. WHEN graph integrity is compromised, THE Cortex_System SHALL prevent learning path generation until integrity is restored

### Requirement 15: API Integration and Extensibility

**User Story:** As a developer, I want clean API endpoints for all core functions, so that the system can be extended or integrated with other tools.

#### Acceptance Criteria

1. THE Cortex_System SHALL expose RESTful API endpoints for syllabus upload, concept explanation submission, and learning path retrieval
2. WHEN API requests are received, THE Cortex_System SHALL validate request authentication using session tokens
3. WHEN API requests contain invalid data, THE Cortex_System SHALL return appropriate HTTP error codes with descriptive messages
4. WHEN API operations succeed, THE Cortex_System SHALL return JSON responses with consistent schema
5. THE Cortex_System SHALL implement rate limiting to prevent API abuse
