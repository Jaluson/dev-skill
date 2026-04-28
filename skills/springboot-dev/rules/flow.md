# Flow Standards

> Applicable mode: Development mode
> Rule prefix: R-FLOW

## R-FLOW-01 Requirements must be displayed for confirmation after analysis

After parsing user requirements, display them in a structured way:
- Feature list
- Clear points (understood content)
- Unclear points (content needing clarification)

Wait for user confirmation or supplement; do not proceed to the next step without confirmation.

## R-FLOW-02 Guessing is prohibited; unclear points must be questioned

Any vague expressions, missing information, or unclear business rules in requirements must be proactively questioned.

**Prohibited behaviors**:
- Assuming default values on your own
- Inferring business logic on your own
- Using "usually", "generally" as basis for generating code

## R-FLOW-03 Multiple unclear points must be asked one by one

When there are multiple issues needing clarification, they must be asked one by one.

**Prohibited behaviors**:
- Listing all questions at once for the user to answer
- Combining multiple questions into one compound question

Correct approach: Resolve one question before asking the next.

## R-FLOW-04 Repeat confirmation after collecting responses

For each response collected, update the requirement understanding and re-display the complete requirement view.

Loop execution:
```
Display current understanding → Ask question → Collect response → Update understanding → Display updated understanding
```

Until all unclear points are resolved.

## R-FLOW-05 Use menu items for questioning

When asking questions, prefer using menu items (option list) to reduce user answer cost.

Example:
```
Please select the storage method for user status:
A. Use enum class UserStatusEnum
B. Use data dictionary table
C. Other (please specify)
```

## R-FLOW-06 Present the complete solution before coding

Before coding, the complete technical solution must be presented, including:

1. **API Design**
   - URL, Method, Content-Type
   - Request (fields / types / validation rules)
   - Response (fields / types / status code meanings)

2. **Database Design**
   - Table name, engine, charset
   - Field list (name / type / length / nullable / default / comment)
   - Index design
   - Compliance statement with standards

3. **Code Structure**
   - Classes involved and their responsibilities
   - Calling relationships between classes
   - Compliance statement with standards

Do not execute coding until the user explicitly replies with "confirm" or similar expression.

## R-FLOW-07 Display the skill used at task milestones

The skill currently in use must be explicitly stated at the following nodes:
- At task start: "Starting development using springboot-dev skill"
- When key phases complete: "[Phase name] complete, continuing to execute springboot-dev standards"
- At task end: "springboot-dev skill execution complete"

## R-FLOW-08 Must compile after phase completion

After each development phase completes, compilation verification must be performed.

- Compilation failure: Locate and fix errors, then recompile
- Compilation success: Proceed to next phase

## R-FLOW-09 Check if functionality is normal

After compilation passes, functional checks must be performed:
- Whether APIs respond normally
- Whether database operations are correct
- Whether boundary conditions are handled

## R-FLOW-10 Compare requirement analysis for compliance

Before task completion, compare against original requirements:
- Whether all feature points are implemented
- Whether any requirements are missed
- Whether there are implementations beyond the requirement scope (minimum change principle)
