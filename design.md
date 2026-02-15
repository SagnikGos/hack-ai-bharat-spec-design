# Design Document: Cortex AI-Powered Learning Intelligence Engine

## Overview

Cortex is a sophisticated AI-powered learning intelligence platform that revolutionizes how students understand and address their knowledge gaps. The system employs a multi-engine architecture combining graph theory, natural language processing, vector embeddings, and strategic optimization algorithms to create a comprehensive learning intelligence system.

The core innovation lies in the reverse teaching methodology: instead of passively consuming content, students actively explain concepts back to the AI, which then evaluates understanding depth through adversarial questioning and multi-dimensional scoring. This approach reveals hidden misconceptions and prerequisite gaps that traditional assessment methods miss.

The system architecture consists of four primary engines working in concert:

1. **Knowledge Graph Builder**: Transforms unstructured syllabi into structured dependency graphs with exam-weighted importance
2. **Reverse Teaching Evaluation Engine**: Assesses understanding through explanation analysis, adversarial questioning, and multi-metric scoring
3. **Weakness Detection & Heatmap Engine**: Identifies root cause gaps and visualizes understanding across the entire knowledge space
4. **Learning Path Compression Engine**: Generates mathematically optimized study sequences based on exam importance, dependency centrality, and weakness factors

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                     Frontend (Next.js)                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Syllabus   │  │   Reverse    │  │  Heatmap     │     │
│  │   Upload     │  │   Teaching   │  │  Visualizer  │     │
│  │   Interface  │  │   Interface  │  │  (React Flow)│     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Backend API (Node.js/Next.js API Routes)        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Knowledge Graph Builder Engine               │  │
│  │  • LLM Concept Extraction                           │  │
│  │  • Dependency DAG Construction                      │  │
│  │  • Exam Weight Analysis                             │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │      Reverse Teaching Evaluation Engine              │  │
│  │  • Explanation Analysis (Vector Embeddings)         │  │
│  │  • Adversarial Question Generator                   │  │
│  │  • Multi-Metric Scoring System                      │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │      Weakness Detection & Heatmap Engine             │  │
│  │  • Root Gap Detection (Graph Traversal)             │  │
│  │  • Understanding Score Aggregation                  │  │
│  │  • Heatmap Data Generation                          │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Learning Path Compression Engine             │  │
│  │  • Priority Score Calculation                       │  │
│  │  • Topological Sort with Constraints                │  │
│  │  • Time Estimation & Roadmap Generation             │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              External Services & Storage                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   OpenAI     │  │  Supabase/   │  │   Vector     │     │
│  │   API        │  │  PostgreSQL  │  │   Store      │     │
│  │  (LLM +      │  │  (Relational │  │  (Embeddings)│     │
│  │  Embeddings) │  │   Data)      │  │              │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

1. **Syllabus Ingestion Flow**:
   - User uploads syllabus (PDF/text) → Knowledge Graph Builder extracts concepts via LLM → Dependency relationships identified → Exam weights computed from past papers → DAG persisted to database

2. **Reverse Teaching Flow**:
   - User selects concept → Provides explanation (text/voice) → Reverse Teaching Engine analyzes completeness → Generates adversarial questions → User answers → Multi-metric scoring → Understanding score persisted

3. **Weakness Detection Flow**:
   - Understanding scores retrieved → Graph traversal identifies low-scoring prerequisites → Root gaps flagged → Heatmap data generated → Frontend renders interactive visualization

4. **Learning Path Generation Flow**:
   - User selects mode (Survival/Rank/Interview) → Priority scores computed for all concepts → Topological sort with priority weighting → Time estimates calculated → Weekly roadmap generated

## Components and Interfaces

### 1. Knowledge Graph Builder Engine

**Purpose**: Transform unstructured syllabus documents into structured knowledge dependency graphs with exam-weighted importance.

**Core Components**:

#### ConceptExtractor
```typescript
interface ConceptExtractor {
  extractConcepts(syllabusText: string): Promise<Concept[]>
  identifyDependencies(concepts: Concept[]): Promise<DependencyEdge[]>
}

interface Concept {
  id: string
  name: string
  description: string
  subjectId: string
}

interface DependencyEdge {
  prerequisiteId: string
  dependentId: string
  strength: number // 0-1, how critical the prerequisite is
}
```

**Implementation Strategy**:
- Use LLM with structured prompting to extract concepts from syllabus text
- Prompt engineering: "Extract all distinct learning concepts from this syllabus. For each concept, identify its prerequisites."
- Parse LLM response into structured Concept and DependencyEdge objects
- Validate extracted concepts for completeness and clarity

#### DependencyGraphBuilder
```typescript
interface DependencyGraphBuilder {
  buildDAG(concepts: Concept[], edges: DependencyEdge[]): KnowledgeGraph
  validateAcyclic(graph: KnowledgeGraph): boolean
  computeCentrality(graph: KnowledgeGraph): Map<string, number>
}

interface KnowledgeGraph {
  nodes: Map<string, ConceptNode>
  adjacencyList: Map<string, string[]> // conceptId -> prerequisite IDs
  reverseAdjacencyList: Map<string, string[]> // conceptId -> dependent IDs
}

interface ConceptNode {
  concept: Concept
  examWeight: number
  centralityScore: number
  prerequisites: string[]
  dependents: string[]
}
```

**Implementation Strategy**:
- Build adjacency list representation for efficient graph traversal
- Maintain reverse adjacency list for downstream dependency queries
- Use depth-first search (DFS) to detect cycles during construction
- Compute centrality as count of reachable downstream nodes (BFS from each node)

#### ExamWeightAnalyzer
```typescript
interface ExamWeightAnalyzer {
  analyzeExamPapers(papers: ExamPaper[]): Map<string, number>
  normalizeWeights(rawWeights: Map<string, number>): Map<string, number>
  assignDefaultWeights(concepts: Concept[]): Map<string, number>
}

interface ExamPaper {
  year: number
  questions: Question[]
}

interface Question {
  text: string
  marks: number
  relatedConcepts: string[] // concept IDs
}
```

**Implementation Strategy**:
- Use LLM to map exam questions to concept IDs
- Count question frequency and sum marks for each concept
- Normalize to 0-1 scale: weight = (concept_marks / max_concept_marks)
- Apply recency weighting: recent papers weighted higher (exponential decay)
- Default weights: uniform distribution (1 / num_concepts) when no papers provided

### 2. Reverse Teaching Evaluation Engine

**Purpose**: Evaluate student understanding through explanation analysis, adversarial questioning, and multi-dimensional scoring.

**Core Components**:

#### ExplanationAnalyzer
```typescript
interface ExplanationAnalyzer {
  analyzeCompleteness(explanation: string, referenceConcept: Concept): Promise<CompletenessResult>
  detectMisconceptions(explanation: string, referenceConcept: Concept): Promise<Misconception[]>
  evaluateCoherence(explanation: string): Promise<CoherenceResult>
}

interface CompletenessResult {
  score: number // 0-1
  missingAspects: string[]
  coverageMap: Map<string, number> // aspect -> coverage score
}

interface Misconception {
  description: string
  severity: 'low' | 'medium' | 'high'
  relatedConcept: string
}

interface CoherenceResult {
  score: number // 0-1
  logicalFlaws: string[]
  clarityScore: number
}
```

**Implementation Strategy**:
- **Completeness Analysis**:
  - Generate embeddings for user explanation using OpenAI embeddings API
  - Generate embeddings for reference concept content (from syllabus or knowledge base)
  - Split reference content into key aspects (using LLM)
  - Compute cosine similarity between explanation and each aspect
  - Completeness score = average similarity across all aspects
  - Flag aspects with similarity < 0.7 as missing

- **Misconception Detection**:
  - Use LLM with prompt: "Analyze this explanation for misconceptions about [concept]. List any incorrect statements or flawed reasoning."
  - Parse LLM response to extract misconceptions with severity
  - Cross-reference with common misconception database (if available)

- **Coherence Evaluation**:
  - Use LLM with prompt: "Rate the logical coherence of this explanation on a scale of 0-1. Identify any logical flaws or unclear statements."
  - Parse numerical score and flaw descriptions
  - Clarity score based on sentence structure complexity and vocabulary appropriateness

#### AdversarialQuestionGenerator
```typescript
interface AdversarialQuestionGenerator {
  generateQuestions(concept: Concept, explanation: string, missingAspects: string[]): Promise<AdversarialQuestion[]>
  evaluateAnswer(question: AdversarialQuestion, answer: string): Promise<AnswerEvaluation>
}

interface AdversarialQuestion {
  id: string
  text: string
  type: 'edge_case' | 'boundary_condition' | 'misconception_probe' | 'missing_aspect'
  targetAspect: string
  expectedKeyPoints: string[]
}

interface AnswerEvaluation {
  correct: boolean
  score: number // 0-1
  feedback: string
  keyPointsCovered: string[]
}
```

**Implementation Strategy**:
- **Question Generation**:
  - Prompt LLM: "Generate 3-5 challenging questions about [concept] that test edge cases and boundary conditions. Focus on: [missing aspects]"
  - Ensure questions target specific gaps identified in completeness analysis
  - Include at least one question probing common misconceptions
  - Store expected key points for each question (extracted via LLM)

- **Answer Evaluation**:
  - Generate embeddings for user answer and expected key points
  - Compute similarity scores for each key point
  - Correct if average similarity > 0.7
  - Use LLM for nuanced evaluation: "Does this answer correctly address the question? Provide feedback."

#### UnderstandingScoreCalculator
```typescript
interface UnderstandingScoreCalculator {
  computeUnderstandingScore(
    completeness: number,
    questionAccuracy: number,
    coherence: number
  ): number
  
  aggregateSessionScore(session: TeachingSession): UnderstandingScore
}

interface TeachingSession {
  conceptId: string
  userId: string
  explanation: string
  completenessResult: CompletenessResult
  coherenceResult: CoherenceResult
  adversarialQuestions: AdversarialQuestion[]
  answers: Map<string, string> // questionId -> answer
  answerEvaluations: Map<string, AnswerEvaluation>
  timestamp: Date
}

interface UnderstandingScore {
  conceptId: string
  userId: string
  score: number // 0-1
  completenessScore: number
  questionAccuracy: number
  coherenceScore: number
  misconceptions: Misconception[]
  prerequisiteGaps: string[] // concept IDs
  timestamp: Date
}
```

**Implementation Strategy**:
- **Score Calculation**:
  ```
  Understanding_Score = (Completeness × 0.4) + (Question_Accuracy × 0.4) + (Coherence × 0.2)
  ```
  - Completeness: from ExplanationAnalyzer
  - Question_Accuracy: average of all answer evaluation scores
  - Coherence: from ExplanationAnalyzer

- **Prerequisite Gap Detection**:
  - Query graph for prerequisite concepts
  - Retrieve understanding scores for prerequisites
  - Flag prerequisites with score < 0.5 as gaps
  - Include in UnderstandingScore object

### 3. Weakness Detection & Heatmap Engine

**Purpose**: Identify root cause understanding gaps and generate visual representations of knowledge mastery.

**Core Components**:

#### RootGapDetector
```typescript
interface RootGapDetector {
  detectRootGaps(conceptId: string, userId: string, graph: KnowledgeGraph): Promise<RootGap[]>
  rankGapsByImpact(gaps: RootGap[], graph: KnowledgeGraph): RootGap[]
}

interface RootGap {
  conceptId: string
  understandingScore: number
  affectedConcepts: string[] // downstream concepts impacted
  centralityScore: number
  priority: number // computed ranking
}
```

**Implementation Strategy**:
- **Gap Detection Algorithm**:
  ```
  1. Start with concept C where understanding_score < 0.6
  2. Traverse prerequisites using BFS
  3. For each prerequisite P:
     - If understanding_score(P) < 0.5, mark as root gap
     - Add all downstream dependents of P to affected_concepts
  4. Return all identified root gaps
  ```

- **Impact Ranking**:
  ```
  gap_priority = (1 - understanding_score) × 0.5 + centrality_score × 0.5
  ```
  - Higher priority = more critical to address
  - Centrality score indicates how many concepts depend on this gap

#### HeatmapDataGenerator
```typescript
interface HeatmapDataGenerator {
  generateHeatmapData(userId: string, subjectId: string): Promise<HeatmapData>
  computeNodeColors(scores: Map<string, number>): Map<string, string>
}

interface HeatmapData {
  nodes: HeatmapNode[]
  edges: HeatmapEdge[]
  metadata: HeatmapMetadata
}

interface HeatmapNode {
  id: string
  label: string
  color: string // hex color based on understanding score
  understandingScore: number
  completenessScore: number
  coherenceScore: number
  misconceptions: string[]
  position: { x: number, y: number } // for layout
}

interface HeatmapEdge {
  source: string
  target: string
  strength: number
}

interface HeatmapMetadata {
  totalConcepts: number
  masteredConcepts: number // score >= 0.7
  weakConcepts: number // score < 0.4
  averageScore: number
}
```

**Implementation Strategy**:
- **Color Coding**:
  ```
  score >= 0.7: green (#10B981)
  0.4 <= score < 0.7: yellow (#F59E0B)
  score < 0.4: red (#EF4444)
  no score yet: gray (#9CA3AF)
  ```

- **Layout Algorithm**:
  - Use hierarchical layout (Dagre algorithm) for DAG visualization
  - Root concepts (no prerequisites) at top
  - Dependent concepts flow downward
  - React Flow library handles rendering and interaction

### 4. Learning Path Compression Engine

**Purpose**: Generate mathematically optimized study sequences that maximize learning efficiency.

**Core Components**:

#### PriorityScoreCalculator
```typescript
interface PriorityScoreCalculator {
  computePriorityScores(
    graph: KnowledgeGraph,
    understandingScores: Map<string, number>,
    mode: LearningMode
  ): Map<string, number>
}

type LearningMode = 'survival' | 'rank' | 'interview'

interface PriorityWeights {
  examWeight: number
  centralityWeight: number
  weaknessWeight: number
}
```

**Implementation Strategy**:
- **Base Priority Formula**:
  ```
  Priority_Score = (Exam_Weight × w1) + (Centrality × w2) + (Weakness × w3)
  
  where:
  - Weakness = 1 - Understanding_Score
  - Centrality = normalized count of downstream dependents
  ```

- **Mode-Specific Weights**:
  ```
  Survival Mode (exam-focused):
    w1 = 0.7, w2 = 0.2, w3 = 0.1
  
  Rank Mode (comprehensive mastery):
    w1 = 0.3, w2 = 0.5, w3 = 0.2
  
  Interview Mode (foundational):
    w1 = 0.2, w2 = 0.3, w3 = 0.5
    + bonus for root concepts (no prerequisites)
  ```

#### PathOptimizer
```typescript
interface PathOptimizer {
  generateOptimalPath(
    graph: KnowledgeGraph,
    priorityScores: Map<string, number>,
    understandingScores: Map<string, number>
  ): LearningPath
  
  validatePathConstraints(path: LearningPath, graph: KnowledgeGraph): boolean
}

interface LearningPath {
  concepts: ConceptStep[]
  totalEstimatedHours: number
  weeklyRoadmap: WeekPlan[]
  metadata: PathMetadata
}

interface ConceptStep {
  conceptId: string
  conceptName: string
  priorityScore: number
  estimatedHours: number
  prerequisites: string[]
  resources: string[]
}

interface WeekPlan {
  weekNumber: number
  concepts: string[]
  totalHours: number
  goals: string[]
}

interface PathMetadata {
  mode: LearningMode
  generatedAt: Date
  totalConcepts: number
  masteredConcepts: number
}
```

**Implementation Strategy**:
- **Path Generation Algorithm**:
  ```
  1. Compute priority scores for all concepts
  2. Perform topological sort on DAG (Kahn's algorithm)
  3. Within each topological level, sort by priority score (descending)
  4. Filter out concepts with understanding_score >= 0.8 (already mastered)
  5. Generate ordered list respecting prerequisites
  ```

- **Topological Sort with Priority**:
  ```
  1. Initialize in-degree map for all nodes
  2. Add all nodes with in-degree 0 to priority queue (sorted by priority score)
  3. While queue not empty:
     - Pop highest priority node
     - Add to path
     - Decrement in-degree of dependents
     - Add dependents with in-degree 0 to queue
  4. If path length < total nodes, cycle detected (error)
  ```

#### TimeEstimator
```typescript
interface TimeEstimator {
  estimateConceptTime(
    concept: Concept,
    understandingScore: number,
    complexity: number
  ): number
  
  generateWeeklyRoadmap(
    path: LearningPath,
    availableHoursPerWeek: number
  ): WeekPlan[]
}
```

**Implementation Strategy**:
- **Time Estimation Formula**:
  ```
  base_hours = concept_complexity × 2  // complexity on 1-5 scale
  weakness_multiplier = 1 + (1 - understanding_score)
  estimated_hours = base_hours × weakness_multiplier
  ```

- **Weekly Roadmap Generation**:
  ```
  1. Start with week 1, hours_remaining = available_hours_per_week
  2. For each concept in path:
     - If concept.estimated_hours <= hours_remaining:
       - Add to current week
       - hours_remaining -= concept.estimated_hours
     - Else:
       - Move to next week
       - hours_remaining = available_hours_per_week
       - Add concept to new week
  3. Generate goals for each week based on concepts
  ```

## Data Models

### Database Schema

```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Subjects table
CREATE TABLE subjects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  syllabus_text TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Concept nodes table
CREATE TABLE concept_nodes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  exam_weight DECIMAL(3,2) DEFAULT 0.5,
  centrality_score DECIMAL(3,2) DEFAULT 0.0,
  complexity INTEGER DEFAULT 3, -- 1-5 scale
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(subject_id, name)
);

-- Concept edges table (dependency relationships)
CREATE TABLE concept_edges (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  prerequisite_id UUID REFERENCES concept_nodes(id) ON DELETE CASCADE,
  dependent_id UUID REFERENCES concept_nodes(id) ON DELETE CASCADE,
  strength DECIMAL(3,2) DEFAULT 1.0, -- how critical the prerequisite is
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(prerequisite_id, dependent_id),
  CHECK (prerequisite_id != dependent_id)
);

-- Understanding scores table
CREATE TABLE understanding_scores (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  concept_id UUID REFERENCES concept_nodes(id) ON DELETE CASCADE,
  score DECIMAL(3,2) NOT NULL CHECK (score >= 0 AND score <= 1),
  completeness_score DECIMAL(3,2),
  coherence_score DECIMAL(3,2),
  question_accuracy DECIMAL(3,2),
  misconceptions JSONB DEFAULT '[]',
  prerequisite_gaps JSONB DEFAULT '[]',
  session_id UUID,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Create index for latest score queries
CREATE INDEX idx_understanding_scores_latest 
ON understanding_scores(user_id, concept_id, created_at DESC);

-- Session logs table
CREATE TABLE session_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  concept_id UUID REFERENCES concept_nodes(id) ON DELETE CASCADE,
  explanation TEXT NOT NULL,
  completeness_result JSONB,
  coherence_result JSONB,
  adversarial_questions JSONB,
  answer_evaluations JSONB,
  duration_seconds INTEGER,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Exam weights table (for tracking weight sources)
CREATE TABLE exam_weights (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  concept_id UUID REFERENCES concept_nodes(id) ON DELETE CASCADE,
  source VARCHAR(50), -- 'exam_paper', 'manual', 'default'
  weight DECIMAL(3,2) NOT NULL,
  exam_year INTEGER,
  question_count INTEGER,
  total_marks INTEGER,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Learning paths table (cached paths)
CREATE TABLE learning_paths (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  mode VARCHAR(20) NOT NULL, -- 'survival', 'rank', 'interview'
  path_data JSONB NOT NULL, -- serialized LearningPath object
  total_hours DECIMAL(5,2),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Create indexes for performance
CREATE INDEX idx_concept_edges_prerequisite ON concept_edges(prerequisite_id);
CREATE INDEX idx_concept_edges_dependent ON concept_edges(dependent_id);
CREATE INDEX idx_session_logs_user_concept ON session_logs(user_id, concept_id);
CREATE INDEX idx_learning_paths_user_subject ON learning_paths(user_id, subject_id);
```

### Vector Embeddings Storage

For efficient similarity search, embeddings will be stored in a separate vector store:

```typescript
interface EmbeddingRecord {
  id: string
  conceptId: string
  textType: 'reference' | 'explanation' | 'aspect'
  text: string
  embedding: number[] // 1536-dimensional vector (OpenAI ada-002)
  metadata: {
    userId?: string
    sessionId?: string
    aspectName?: string
  }
  createdAt: Date
}
```

**Storage Options**:
- **Option 1**: Supabase with pgvector extension (recommended for simplicity)
- **Option 2**: Pinecone (dedicated vector database, better for scale)
- **Option 3**: In-memory with periodic persistence (for MVP)

## Correctness Properties


*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property Reflection

After analyzing all acceptance criteria, several redundancies were identified:

- **Requirements 1.5 and 14.1** both test acyclic graph invariant → Combined into single property
- **Requirements 6.7 and 6.8** both test prerequisite ordering → Combined into single property
- **Requirements 6.1, 6.2, 6.3** test priority score formula components → Combined into comprehensive formula property
- **Requirements 6.4, 6.5, 6.6** test mode-specific weights → Combined into mode-based weighting property
- **Requirements 7.1, 7.2, 7.3** test embedding and similarity pipeline → Combined into coverage analysis property
- **Requirements 3.6, 4.5, 9.1** all test persistence → Combined into general persistence round-trip property

### Core Properties

#### Property 1: Graph Acyclicity Invariant
*For any* Dependency_DAG constructed or modified by the Knowledge_Graph_Builder, the graph SHALL contain no cycles, ensuring all prerequisite relationships form a valid directed acyclic graph.

**Validates: Requirements 1.5, 14.1**

#### Property 2: Graph Persistence Round-Trip
*For any* Dependency_DAG with Concept_Nodes and edges, persisting to the database then retrieving SHALL produce an equivalent graph structure with all nodes, edges, and weights preserved.

**Validates: Requirements 1.6**

#### Property 3: Exam Weight Proportionality
*For any* set of exam papers provided, the Exam_Weight assigned to each Concept_Node SHALL be proportional to the frequency and marks of questions related to that concept, normalized to the 0-1 range.

**Validates: Requirements 1.3, 13.1, 13.2**

#### Property 4: Exam Weight Recency Weighting
*For any* multiple exam papers with different years, the computed Exam_Weight SHALL apply higher weight to more recent papers using exponential decay.

**Validates: Requirements 13.3**

#### Property 5: Cycle Detection on Edge Addition
*For any* valid Dependency_DAG and any new edge (prerequisite, dependent), adding the edge SHALL be rejected if and only if it would create a cycle in the graph.

**Validates: Requirements 14.2, 14.3**

#### Property 6: Graph Connectivity
*For any* valid Dependency_DAG, all Concept_Nodes SHALL be reachable from at least one root node (node with no prerequisites).

**Validates: Requirements 14.4**

#### Property 7: Explanation Acceptance
*For any* text explanation provided by a User for a Concept_Node, the Reverse_Teaching_Engine SHALL accept and process the explanation without rejection.

**Validates: Requirements 2.1**

#### Property 8: Adversarial Question Generation
*For any* User explanation of a Concept_Node, the Reverse_Teaching_Engine SHALL generate between 3 and 5 Adversarial_Questions targeting edge cases and identified gaps.

**Validates: Requirements 2.3, 11.4**

#### Property 9: Question Targeting Missing Aspects
*For any* User explanation with identified missing aspects (completeness < 0.7 for specific aspects), the generated Adversarial_Questions SHALL include at least one question targeting each missing aspect.

**Validates: Requirements 11.3**

#### Property 10: Answer Evaluation Completeness
*For any* Adversarial_Question and User answer, the Reverse_Teaching_Engine SHALL produce an AnswerEvaluation with a score in the range [0, 1] and feedback text.

**Validates: Requirements 2.5**

#### Property 11: Completeness Score via Cosine Similarity
*For any* User explanation and reference concept content, the Completeness_Score SHALL be computed as the average cosine similarity between the explanation embedding and embeddings of all key aspects of the reference content.

**Validates: Requirements 3.1, 7.1, 7.2, 7.3**

#### Property 12: Coherence Score Range
*For any* User explanation evaluated for coherence, the Coherence_Score SHALL be in the range [0, 1] where higher scores indicate greater logical consistency.

**Validates: Requirements 3.2**

#### Property 13: Understanding Score Formula
*For any* completed teaching Session with Completeness_Score C, Question_Accuracy Q, and Coherence_Score H, the Understanding_Score SHALL equal (C × 0.4) + (Q × 0.4) + (H × 0.2).

**Validates: Requirements 3.3**

#### Property 14: Prerequisite Gap Flagging
*For any* Concept_Node being evaluated, if any prerequisite Concept_Node has Understanding_Score < 0.5 for the same User, those prerequisites SHALL be flagged as prerequisite gaps.

**Validates: Requirements 3.5**

#### Property 15: Misconception Detection
*For any* User explanation evaluated by the Reverse_Teaching_Engine, if misconceptions are detected by LLM analysis, they SHALL be flagged and included in the Session results.

**Validates: Requirements 3.4**

#### Property 16: Score Persistence Round-Trip
*For any* Understanding_Score computed for a User and Concept_Node, persisting to the database then retrieving SHALL return the same score value, timestamp, and all component scores (completeness, coherence, question accuracy).

**Validates: Requirements 3.6, 9.1**

#### Property 17: Root Gap Identification
*For any* Concept_Node with Understanding_Score < 0.6, traversing its prerequisites SHALL identify all prerequisite Concept_Nodes with Understanding_Score < 0.5 as Root_Gaps.

**Validates: Requirements 4.1, 4.2**

#### Property 18: Root Gap Ranking by Centrality
*For any* set of Root_Gaps identified, they SHALL be ranked in descending order by their dependency centrality score (number of downstream dependent concepts).

**Validates: Requirements 4.3**

#### Property 19: Root Gap Downstream Association
*For any* Root_Gap identified, all Concept_Nodes that depend on it (directly or transitively) SHALL be associated as affected concepts.

**Validates: Requirements 4.4**

#### Property 20: Heatmap Color Coding
*For any* Concept_Node with Understanding_Score S, the heatmap visualization SHALL assign color green if S ≥ 0.7, yellow if 0.4 ≤ S < 0.7, red if S < 0.4, and gray if no score exists.

**Validates: Requirements 5.2**

#### Property 21: Heatmap Node Data Completeness
*For any* Concept_Node in the heatmap visualization, hovering SHALL display data including Understanding_Score, Completeness_Score, Coherence_Score, and all identified misconceptions.

**Validates: Requirements 5.3**

#### Property 22: Heatmap Dependency Highlighting
*For any* Concept_Node clicked in the heatmap, the system SHALL identify and highlight all prerequisite nodes (ancestors in DAG) and all dependent nodes (descendants in DAG).

**Validates: Requirements 5.4**

#### Property 23: Heatmap Edge Representation
*For any* Dependency_DAG visualized as a heatmap, all edges in the graph SHALL be displayed as connections between corresponding nodes.

**Validates: Requirements 5.6**

#### Property 24: Priority Score Calculation Formula
*For any* Concept_Node with Exam_Weight E, Dependency_Centrality D, and Understanding_Score U, the Priority_Score SHALL equal (E × w1) + (D × w2) + ((1 - U) × w3) where weights depend on the selected Learning_Mode.

**Validates: Requirements 6.1, 6.2, 6.3**

#### Property 25: Mode-Specific Priority Weights
*For any* Learning_Mode selected, the priority weights SHALL be: Survival (w1=0.7, w2=0.2, w3=0.1), Rank (w1=0.3, w2=0.5, w3=0.2), or Interview (w1=0.2, w2=0.3, w3=0.5).

**Validates: Requirements 6.4, 6.5, 6.6**

#### Property 26: Learning Path Prerequisite Ordering
*For any* generated Learning_Path, no Concept_Node SHALL appear in the path before any of its prerequisite Concept_Nodes, ensuring valid topological ordering.

**Validates: Requirements 6.7, 6.8**

#### Property 27: Time Estimation Based on Understanding
*For any* Concept_Node with complexity C and Understanding_Score U, the estimated study time SHALL equal (C × 2) × (1 + (1 - U)) hours, where complexity is on a 1-5 scale.

**Validates: Requirements 6.9**

#### Property 28: Weekly Roadmap Time Distribution
*For any* Learning_Path with available hours H per week, the weekly roadmap SHALL distribute concepts such that no week exceeds H hours and all concepts are included.

**Validates: Requirements 6.10**

#### Property 29: Coverage Gap Identification
*For any* User explanation with cosine similarity < 0.7 to reference content, the system SHALL identify and present specific missing aspects by comparing embedding clusters.

**Validates: Requirements 7.4, 7.5**

#### Property 30: Multi-Subject Data Isolation
*For any* two distinct Subjects belonging to the same User, their Dependency_DAGs, Concept_Nodes, and Understanding_Scores SHALL be completely independent with no cross-contamination.

**Validates: Requirements 8.2, 8.5**

#### Property 31: Subject Creation
*For any* authenticated User, creating multiple Subject entities SHALL succeed without limit, and each Subject SHALL be independently manageable.

**Validates: Requirements 8.1**

#### Property 32: Subject Filtering
*For any* Understanding_Heatmap request with a Subject filter, only Concept_Nodes belonging to that Subject SHALL be included in the visualization.

**Validates: Requirements 8.4**

#### Property 33: Session History Ordering
*For any* User requesting session history, all Sessions SHALL be returned ordered by timestamp in descending order (most recent first).

**Validates: Requirements 9.2**

#### Property 34: Historical Score Preservation
*For any* Concept_Node re-evaluated by a User, all previous Understanding_Scores SHALL be preserved in the database, creating a complete history rather than overwriting.

**Validates: Requirements 9.4**

#### Property 35: Session History Completeness
*For any* Session in the history, the displayed data SHALL include all identified misconceptions and Root_Gaps from that Session.

**Validates: Requirements 9.5**

#### Property 36: Password Hashing Security
*For any* new User registration, the password SHALL be hashed using a secure algorithm (bcrypt or argon2) before storage, and the plain-text password SHALL never be stored.

**Validates: Requirements 10.1**

#### Property 37: Authentication Correctness
*For any* login attempt, authentication SHALL succeed if and only if the provided credentials match a User account with correct password hash verification.

**Validates: Requirements 10.2**

#### Property 38: User Data Isolation
*For any* authenticated User accessing the system, all retrieved data (Subjects, Concept_Nodes, Understanding_Scores, Sessions) SHALL belong only to that User, with no access to other Users' data.

**Validates: Requirements 10.4**

#### Property 39: Database Foreign Key Integrity
*For any* data persisted to the database, all foreign key relationships SHALL be enforced, preventing orphaned records and maintaining referential integrity.

**Validates: Requirements 10.3**

#### Property 40: Transaction Rollback on Failure
*For any* database operation that fails mid-transaction, all changes within that transaction SHALL be rolled back, leaving the database in its pre-transaction state.

**Validates: Requirements 10.5**

#### Property 41: Real-Time Score Updates
*For any* completed Session, the Understanding_Score for the evaluated Concept_Node SHALL be updated in the database immediately, visible to subsequent queries without delay.

**Validates: Requirements 12.1**

#### Property 42: Cascading Priority Recalculation
*For any* Understanding_Score change for a Concept_Node, Priority_Scores SHALL be recomputed for that node and all nodes that depend on it (directly or transitively).

**Validates: Requirements 12.2**

#### Property 43: Path Regeneration on Significant Change
*For any* Priority_Score changes that would alter the learning path order by more than 2 positions for any concept, the Learning_Path SHALL be regenerated.

**Validates: Requirements 12.3**

#### Property 44: Heatmap Data Freshness
*For any* Understanding_Heatmap request, the displayed Understanding_Scores SHALL reflect the most recent scores from the database without requiring manual refresh.

**Validates: Requirements 12.4**

#### Property 45: Manual Exam Weight Override Persistence
*For any* User-provided manual Exam_Weight adjustment for a Concept_Node, the override SHALL be persisted and used in all subsequent Priority_Score calculations until changed again.

**Validates: Requirements 13.4**

#### Property 46: Weight Change Cascading
*For any* Exam_Weight change (manual or automatic), Priority_Scores SHALL be recomputed and the Learning_Path SHALL be regenerated if ordering changes.

**Validates: Requirements 13.5**

#### Property 47: Invalid Graph Blocks Path Generation
*For any* Dependency_DAG that fails integrity validation (contains cycles or unreachable nodes), Learning_Path generation SHALL be prevented and return an error.

**Validates: Requirements 14.5**

#### Property 48: API Authentication Enforcement
*For any* API request to protected endpoints, the request SHALL be rejected with 401 Unauthorized if no valid session token is provided.

**Validates: Requirements 15.2**

#### Property 49: API Error Response Format
*For any* API request with invalid data, the response SHALL include an appropriate HTTP error code (4xx) and a JSON body with a descriptive error message.

**Validates: Requirements 15.3**

#### Property 50: API Response Schema Consistency
*For any* successful API operation, the JSON response SHALL conform to the documented schema for that endpoint, with consistent field names and types.

**Validates: Requirements 15.4**

#### Property 51: API Rate Limiting
*For any* User making API requests, if the request rate exceeds the configured limit (e.g., 100 requests per minute), subsequent requests SHALL be rejected with 429 Too Many Requests.

**Validates: Requirements 15.5**

## Error Handling

### Error Categories

1. **Input Validation Errors**
   - Invalid syllabus format (corrupted PDF, unsupported encoding)
   - Empty or malformed explanations
   - Invalid concept IDs or user IDs
   - Out-of-range scores or weights

2. **External Service Errors**
   - OpenAI API failures (rate limits, timeouts, service unavailable)
   - Database connection failures
   - Vector store unavailability

3. **Business Logic Errors**
   - Cycle detection in graph construction
   - Unreachable nodes in dependency graph
   - Missing prerequisites for path generation
   - Insufficient data for score calculation

4. **Authentication & Authorization Errors**
   - Invalid credentials
   - Expired session tokens
   - Unauthorized access to other users' data

### Error Handling Strategies

#### Graceful Degradation
- If OpenAI API fails during concept extraction, retry with exponential backoff (3 attempts)
- If embeddings generation fails, fall back to keyword-based similarity
- If adversarial question generation fails, use template-based questions

#### Transaction Management
- All database operations that modify multiple tables use transactions
- Rollback on any failure to maintain consistency
- Log all rollback events for debugging

#### User-Facing Error Messages
- Never expose internal error details or stack traces
- Provide actionable guidance: "Unable to process syllabus. Please ensure the PDF is not password-protected."
- Include error codes for support reference

#### Retry Logic
```typescript
interface RetryConfig {
  maxAttempts: number
  initialDelayMs: number
  backoffMultiplier: number
  retryableErrors: ErrorType[]
}

// Example: OpenAI API calls
const openAIRetryConfig: RetryConfig = {
  maxAttempts: 3,
  initialDelayMs: 1000,
  backoffMultiplier: 2,
  retryableErrors: ['RATE_LIMIT', 'TIMEOUT', 'SERVICE_UNAVAILABLE']
}
```

#### Circuit Breaker Pattern
- Implement circuit breaker for OpenAI API calls
- After 5 consecutive failures, open circuit for 60 seconds
- During open circuit, return cached results or degraded functionality
- Half-open state: allow one test request before fully closing

### Error Response Format

```typescript
interface ErrorResponse {
  error: {
    code: string // e.g., "INVALID_SYLLABUS", "CYCLE_DETECTED"
    message: string // User-friendly message
    details?: any // Optional additional context
    timestamp: string
    requestId: string // For support tracking
  }
}
```

## Testing Strategy

### Dual Testing Approach

The Cortex system requires both unit testing and property-based testing to ensure comprehensive correctness:

- **Unit Tests**: Verify specific examples, edge cases, and error conditions
- **Property Tests**: Verify universal properties across all inputs using randomized test data

Both approaches are complementary and necessary. Unit tests catch concrete bugs in specific scenarios, while property tests verify general correctness across the input space.

### Property-Based Testing Configuration

**Library Selection**: 
- **TypeScript/JavaScript**: fast-check (recommended)
- **Python**: Hypothesis (if any Python services are added)

**Test Configuration**:
- Minimum 100 iterations per property test (due to randomization)
- Each property test must reference its design document property
- Tag format: `// Feature: cortex-learning-engine, Property {number}: {property_text}`

**Example Property Test Structure**:
```typescript
import fc from 'fast-check'

// Feature: cortex-learning-engine, Property 1: Graph Acyclicity Invariant
describe('Graph Acyclicity Invariant', () => {
  it('should maintain acyclic property for any graph construction', () => {
    fc.assert(
      fc.property(
        fc.array(conceptArbitrary, { minLength: 1, maxLength: 20 }),
        fc.array(edgeArbitrary),
        (concepts, edges) => {
          const graph = buildDAG(concepts, edges)
          expect(hasNoCycles(graph)).toBe(true)
        }
      ),
      { numRuns: 100 }
    )
  })
})
```

### Unit Testing Strategy

Unit tests should focus on:

1. **Specific Examples**: Concrete test cases that demonstrate correct behavior
   - Example: "User explains 'photosynthesis' with specific text, receives expected completeness score"

2. **Edge Cases**: Boundary conditions and special cases
   - Empty explanations
   - Single-node graphs
   - Maximum complexity concepts
   - Zero understanding scores

3. **Error Conditions**: Failure scenarios and error handling
   - Invalid syllabus formats
   - API failures
   - Database connection errors
   - Cycle detection

4. **Integration Points**: Component interactions
   - Knowledge Graph Builder → Database persistence
   - Reverse Teaching Engine → OpenAI API
   - Learning Path Engine → Graph traversal

**Balance**: Avoid writing too many unit tests for scenarios that property tests already cover. Focus unit tests on specific examples and integration points.

### Test Coverage Goals

- **Code Coverage**: Minimum 80% line coverage
- **Property Coverage**: All 51 correctness properties implemented as property tests
- **Edge Case Coverage**: All identified edge cases have dedicated unit tests
- **Integration Coverage**: All external service integrations have integration tests

### Testing Pyramid

```
        /\
       /  \
      / E2E \          (10%) - Full system tests
     /______\
    /        \
   /Integration\       (20%) - Component interaction tests
  /____________\
 /              \
/   Unit + PBT   \     (70%) - Unit tests + Property-based tests
/__________________\
```

### Continuous Testing

- Run unit tests on every commit (fast feedback)
- Run property tests on every pull request (comprehensive validation)
- Run integration tests nightly (external service dependencies)
- Run E2E tests before releases (full system validation)

### Test Data Generation

For property-based tests, define arbitraries for core domain objects:

```typescript
// Concept arbitrary
const conceptArbitrary = fc.record({
  id: fc.uuid(),
  name: fc.string({ minLength: 3, maxLength: 50 }),
  description: fc.string({ minLength: 10, maxLength: 200 }),
  subjectId: fc.uuid()
})

// Understanding score arbitrary
const understandingScoreArbitrary = fc.float({ min: 0, max: 1 })

// Dependency edge arbitrary
const edgeArbitrary = fc.record({
  prerequisiteId: fc.uuid(),
  dependentId: fc.uuid(),
  strength: fc.float({ min: 0, max: 1 })
})

// Ensure no self-loops
const validEdgeArbitrary = edgeArbitrary.filter(
  edge => edge.prerequisiteId !== edge.dependentId
)
```

### Mocking Strategy

- Mock OpenAI API calls in unit tests (use recorded responses)
- Mock database in unit tests (use in-memory SQLite or test containers)
- Use real services in integration tests (test environment)
- Minimize mocking in property tests (test real logic)

### Performance Testing

While not part of correctness properties, performance should be monitored:

- Graph construction: < 5 seconds for 100 concepts
- Understanding score calculation: < 2 seconds per session
- Learning path generation: < 3 seconds for 100 concepts
- Heatmap data generation: < 1 second for 100 concepts

Performance tests should run separately from correctness tests.
