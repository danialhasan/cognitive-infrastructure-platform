# Autonomous TDD Setup Workflow
## Complete Configuration for Attention-Decoupled Development

**🎯 Purpose**: Configure Codex for autonomous test-driven development with business requirement alignment and systematic log-based feedback.

---

## 🔧 **Infrastructure Setup Commands**

### **Initial Environment Configuration**
```bash
#!/bin/bash
# autonomous-tdd-setup.sh

echo "🔧 Setting up autonomous TDD environment..."

# Create log directory structure
mkdir -p logs
mkdir -p tickets
mkdir -p autonomous-state

# Initialize test runner with log piping
echo "📋 Starting test runner with log output..."
npm run test:watch > logs/test-output.log 2>&1 &
TEST_RUNNER_PID=$!
echo $TEST_RUNNER_PID > autonomous-state/test-runner.pid

# Initialize dev server with log piping
echo "🚀 Starting dev server with log output..."
npm run dev > logs/dev-server.log 2>&1 &
DEV_SERVER_PID=$!
echo $DEV_SERVER_PID > autonomous-state/dev-server.pid

# Set up combined log monitoring
echo "📊 Setting up unified log monitoring..."
tail -f logs/test-output.log logs/dev-server.log > logs/combined.log &
LOG_MONITOR_PID=$!
echo $LOG_MONITOR_PID > autonomous-state/log-monitor.pid

echo "✅ Autonomous TDD environment ready!"
echo "📁 Logs available at: logs/combined.log"
echo "🎯 Ready for business requirement processing"
```

### **Verification Commands**
```bash
# Verify all systems are running
ps -p $(cat autonomous-state/*.pid) # Should show 3 running processes

# Test log output is working
echo "Test log output:" && head -5 logs/test-output.log
echo "Dev server log:" && head -5 logs/dev-server.log
echo "Combined log:" && head -10 logs/combined.log
```

---

## 📋 **Business Requirement Processing Protocol**

### **Ticket Analysis Framework**
```typescript
// Systematic business requirement extraction
interface BusinessRequirement {
  ticketId: string;
  title: string;
  businessValue: string;
  acceptanceCriteria: string[];
  technicalConstraints: string[];
  successMetrics: SuccessMetric[];
}

// Convert to executable test specifications
function generateTestSpecification(requirement: BusinessRequirement): TestSpec {
  return {
    redTests: generateFailingTests(requirement.acceptanceCriteria),
    verificationSteps: mapToLogVerification(requirement.technicalConstraints),
    businessValidation: createBusinessValueTests(requirement.businessValue),
    successCriteria: convertToAssertions(requirement.successMetrics)
  };
}
```

### **Red Test Generation Pattern**
```typescript
// Generate failing tests that capture business requirements exactly
export function generateBusinessAlignedTests(ticket: BusinessRequirement) {
  // Example for approval decision ticket
  const redTests = `
    describe('${ticket.title}', () => {
      it('should persist approval decision in database', async () => {
        const approvalData = {
          candidateId: 'test-candidate-123',
          decision: 'approved',
          decidedBy: 'hiring-manager-456',
          reason: 'Strong technical skills',
          slackContext: { channelId: 'C123', messageId: 'M456' }
        };

        const response = await request(app)
          .post('/decisions/approval')
          .send(approvalData)
          .expect(200);

        // Business requirement: Response must include trace ID
        expect(response.body).toHaveProperty('traceId');
        expect(response.body.traceId).toMatch(/^trace-[a-f0-9-]{36}$/);

        // Business requirement: Decision must be persisted
        const persistedDecision = await db.decisions.findById(response.body.decisionId);
        expect(persistedDecision).toBeDefined();
        expect(persistedDecision.candidateId).toBe('test-candidate-123');
        expect(persistedDecision.decision).toBe('approved');
      });

      it('should update notification history', async () => {
        // This test will fail initially - that's the RED state
        const notifications = await db.notifications.findByCandidateId('test-candidate-123');
        expect(notifications).toHaveLength(1);
        expect(notifications[0].type).toBe('approval_decision');
      });
    });
  `;

  return redTests;
}
```

---

## 🔄 **Autonomous Development Cycle**

### **Phase 1: RED (Business Requirements → Failing Tests)**
```typescript
async function redPhase(ticket: BusinessRequirement) {
  console.log(`📝 RED Phase: Writing failing tests for ${ticket.title}`);

  // Generate tests that encode business requirements
  const redTests = generateBusinessAlignedTests(ticket);

  // Write tests to appropriate test files
  await writeTestFile(`tests/${ticket.module}.test.ts`, redTests);

  // Verify tests fail for correct reasons
  const testResults = await readLogFile('logs/test-output.log');
  const failureReasons = parseTestFailures(testResults);

  console.log(`❌ Tests failing as expected: ${failureReasons.length} requirements unmet`);

  return {
    phase: 'RED',
    testsWritten: true,
    failuresExpected: failureReasons,
    readyForGreen: validateRedState(failureReasons)
  };
}
```

### **Phase 2: GREEN (Minimal Implementation)**
```typescript
async function greenPhase(redResults: RedPhaseResult) {
  console.log(`✅ GREEN Phase: Implementing to satisfy business requirements`);

  let testsPassing = false;
  let attempts = 0;
  const maxAttempts = 10;

  while (!testsPassing && attempts < maxAttempts) {
    // Write minimal implementation
    const implementation = generateMinimalImplementation(redResults.failuresExpected);
    await writeImplementationFile(implementation);

    // Check logs for test results
    const testResults = await readLogFile('logs/test-output.log');
    testsPassing = parseTestResults(testResults).allPassing;

    if (!testsPassing) {
      console.log(`🔄 Attempt ${attempts + 1}: Tests still failing, iterating...`);
      const stillFailing = parseTestFailures(testResults);
      // Use failure details to refine implementation
    }

    attempts++;
  }

  return {
    phase: 'GREEN',
    testsPass: testsPassing,
    implementationComplete: true,
    readyForRefactor: testsPassing
  };
}
```

### **Phase 3: REFACTOR (Quality + Security Review)**
```typescript
async function refactorPhase(greenResults: GreenPhaseResult) {
  console.log(`🔧 REFACTOR Phase: Improving code quality while maintaining GREEN`);

  const currentImplementation = await readImplementationFiles();

  // Apply quality improvements
  const refactoredCode = await improveCodeQuality(currentImplementation, {
    maintainTestPassing: true,
    functionalProgrammingPatterns: true,
    businessLogicSeparation: true,
    errorHandlingImprovement: true
  });

  // Verify tests still pass after refactoring
  const testResults = await readLogFile('logs/test-output.log');
  const stillPassing = parseTestResults(testResults).allPassing;

  if (!stillPassing) {
    console.log(`⚠️ Refactoring broke tests, reverting...`);
    await revertToGreenState();
    return { phase: 'REFACTOR', success: false, reverted: true };
  }

  // Run security and bug review before ticket completion
  console.log(`🔍 Running /review command for security and bug analysis...`);
  const reviewResults = await runCodeReview(); // Codex /review command

  if (reviewResults.issues.length > 0) {
    console.log(`⚠️ Review found ${reviewResults.issues.length} issues, addressing...`);
    await addressReviewIssues(reviewResults.issues);

    // Re-verify tests still pass after addressing review issues
    const finalTestResults = await readLogFile('logs/test-output.log');
    if (!parseTestResults(finalTestResults).allPassing) {
      await revertToGreenState();
      return { phase: 'REFACTOR', success: false, reviewIssues: reviewResults.issues };
    }
  }

  return {
    phase: 'REFACTOR',
    qualityImproved: true,
    testsStillPassing: true,
    ticketComplete: true
  };
}
```

---

## 📊 **Log Analysis and Feedback**

### **Test Output Parsing**
```typescript
// Systematic log analysis for autonomous decision making
export function parseTestResults(logContent: string) {
  const lines = logContent.split('\n');

  // Extract test results
  const testSummary = lines.find(line => line.includes('Test Suites:'));
  const testResults = lines.filter(line => line.includes('✓') || line.includes('✗'));

  // Parse failure details
  const failures = lines.filter(line => line.includes('FAIL') || line.includes('Error:'));
  const failureDetails = failures.map(parseFailureReason);

  return {
    allPassing: testSummary?.includes('0 failed'),
    totalTests: extractTestCount(testSummary),
    failures: failureDetails,
    needsImplementation: failureDetails.filter(f => f.type === 'not_implemented'),
    needsFixing: failureDetails.filter(f => f.type === 'logic_error')
  };
}
```

### **Development Server Monitoring**
```typescript
// Monitor compilation and runtime status
export function parseDevServerLogs(logContent: string) {
  const lines = logContent.split('\n');

  return {
    compilationStatus: extractCompilationStatus(lines),
    serverRunning: lines.some(line => line.includes('Server listening')),
    runtimeErrors: extractRuntimeErrors(lines),
    performanceMetrics: extractResponseTimes(lines),
    lastSuccessfulBuild: extractLastBuildTime(lines)
  };
}
```

---

## 🛡️ **Safety and Validation Mechanisms**

### **Business Alignment Validation**
```typescript
// Ensure autonomous work meets business requirements
const businessAlignmentCheck = {
  beforeImplementation: "Verify tests encode actual business requirements",
  duringImplementation: "Monitor for scope creep beyond ticket requirements",
  afterImplementation: "Validate output matches stakeholder expectations",
  handoffValidation: "Human approval required for business logic changes"
}
```

### **Quality Gates**
```bash
# Mandatory validation before ticket completion
1. All tests pass (GREEN state confirmed)
2. Business requirements satisfied (acceptance criteria met)
3. Code quality standards maintained (functional patterns)
4. No regressions introduced (existing tests still pass)
5. Documentation updated (if business logic changed)
```

---

## 🎯 **Integration with Human Workflows**

### **Handoff Protocol**
```bash
# When autonomous cycle completes
1. Generate completion summary: tickets/autonomous-completion-report.md
2. Highlight any business decisions required
3. Document patterns learned for future cycles
4. Request human validation before deployment
```

### **Escalation Triggers**
```typescript
// When to request human attention
const escalationTriggers = {
  businessRequirementAmbiguity: "Acceptance criteria unclear or contradictory",
  technicalArchitectureChange: "Implementation requires architectural decisions",
  externalServiceFailure: "Required integrations unavailable",
  qualityGateFailure: "Cannot achieve GREEN state within reasonable attempts",
  timeBoxExpiration: "Autonomous cycle duration limit reached"
}
```

---

**🎯 Success Indicator**: Codex operates autonomously for 4-12 hour cycles, completing business requirements without human attention, maintaining quality through systematic TDD methodology.

*This workflow enables true attention decoupling: agents work while humans focus on strategy, resulting in multiplied productivity and systematic business alignment.*